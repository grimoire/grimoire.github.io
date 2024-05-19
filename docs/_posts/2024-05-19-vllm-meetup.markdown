---
layout: post
title:  "【FML】好消息：我上了 vllm meetup..."
date:   2024-05-19 17:00:000 +0800
categories: FML MoE
lang: zh-CN
---

好消息：我上了 vllm meetup...

## 坏消息：是公开处刑

[vLLM](https://github.com/vllm-project/vllm) 在四月组织了一次 meetup，分享了他们的项目进展，roadmap 以及 benchmark，其中也包括与同类项目的比较，[LMDeploy](https://github.com/InternLM/lmdeploy) 也被选作是比较的对象之一。

得益于我的大佬同事们的努力，LMDeploy（TurboMind） 在 Llama2 的性能评估中遥遥领先。就在我以为对手已经风中残烛的时候，vLLM 默默翻开了陷阱卡：

<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/mixtral8x7b-suck.png?raw=true" alt="mixtral-8x7b" style="width:512px;"/>

<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/model-support-important.png?raw=true" alt="support-model-important" style="width:512px;"/>

😡这么烂的 TP 还有 MoE 的支持谁做的，一定要出重拳！让我看看提交记录...

**<span style="color:red">好像是我<span>**
<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/crown.png?raw=true" alt="crown" style="width:128px;"/>

咳咳，还是先看看问题出在哪里吧

## Mix of Expert

上面提到的出现性能问题的模型是 Mixtral，它是一个 MoE 模型。MoE 是 Mix of Expert 的缩写，它通过增加更多的权重以及计算来提升模型的效果，那么这些权重和计算到底是什么？

### 集成学习

想象你是一位投资者，为了确定是否要购入某支股票请教了数位分析师。假设每一位分析师分析正确的概率大约是 60%，那么无条件信任其中的任意一位，投资成功的概率也就是 60%。

<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/single-exp.png?raw=true" alt="single-expert" style="width:256px;"/>

但假如这些分析师预测的概率独立，并且其中有 2/3 的人支持购买该股票，1/3 的人不支持呢？那么支持的分析师预测错误概率显然是远低于 40% 的。

<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/multi-exp.png?raw=true" alt="multi-expert" style="width:256px;"/>

显然，当你拥有更多分析师（Expert）时，作出正确选择的可能性会更高。这就是[集成学习](https://en.wikipedia.org/wiki/Ensemble_learning) 的核心，通过一定的策略组合复数个专家模型，以期待可以得到更好的结果。

{% include note.html content="上面的例子省略了很多细节，比如每个专家模型能力的下限；模型间分布的差异以及独立性等等，这里就不做展开了。" %}

MoE 就是集成学习的一种，它会将 transformer 中 decode layer 里的一个 MLP 替换成复数个 MLP 的组合。MoE 的 Mixtral-8x7b 模型拥有70亿参数，在许多 benchmark 中可以超过拥有700亿参数的 Llama 2。

### MoE 模块结构

目前支持 MoE 的模型包括 Mixtral，DeepSeek，Qwen，DBRX 等，尽管在实现细节以及参数配置上存在一些差异，但是模型结构基本是一样的。

<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/architecture-of-GLaM.gif?raw=true" alt="MoE" style="width:512px;"/>

MoE 是 token-wise 的，每一个 token 对应的特征，会经过一个 Gate（在有的论文或代码中也被叫做 Router），以选择数个最适合它的专家模型。

还是以前面投资者为例子，Gate 相当于理财公司的前台，根据客户（Token）类型（股票，债权，期货...）选择最适合的分析师（Expert）。Gate 还会给推荐的 Expert 打分（weight），让 Token 知道哪个 Expert 的结果更值得信任。

<img src="https://github.com/grimoire/grimoire.github.io/blob/vllm-failed-resources/resources/MoE1.png?raw=true" alt="MoE1" style="width:512px;"/>

之后输入会被送入对应的这些 Expert model（通常是一个比较小的 MLP），然后用 Gate 提供的 weight 进行线性加权，以得到最终的结果。

## 性能瓶颈

{% include note.html content="本章节的内容可能需要对模型和代码由一定的了解，我的写作能力也稀烂，没兴趣的读者可以在这里止步。如果有兴趣的话欢迎来 LMDeploy 或 vLLM 的 repo 中的讨论以及 PR。" %}

MoE 中的 Gate 保证每个 token 都可以选择最适合自己的 Expert model 来进行计算以提高精度，实现通常遵循下面的方式：

```python
# gate 层提供每个 expert 的权重，topk 选择最适合的 expert
expert_weight = gate(hidden_states)
weight, topk_id = torch.topk(expert_weight, k)

# 遍历每个 expert
output = torch.empty(...)
for expert_id in range(num_expert):
    # 选择使用当前 expert 的 token
    # 这一步通常会涉及到 stream synchronize
    state, s_id = select_token(hidden_state, topk_id, expert_id)

    # 使用 expert 进行计算
    exp_out = experts[expert_id](state)

    # 将输出结果加权放到 output 中
    scatter_output(output, exp_out, s_id, weight)

return output
```

这种实现带来了两个问题：

1. 要想充分利用 GPU 的计算能力，我们通常会在推理开始时就将所需的计算 kernel 放在队列中，这样在 GPU 完成一个 kernel 计算时，可以马上开始下一个。但是由于 Gate 运算之后我们才能知道 token 适用哪个 Expert，GPU 只能等待 expert id 的划分完成（这一步通常涉及到 cuda stream synchronize），降低硬件吞吐。
2. 向 GPU 发布计算请求这件事本身也存在一些开销，对于哪些运算量巨大的 Kernel （比如 MHA）这种开销通常可以忽略不计，但是 MoE 中的每个 Expert 通常都是很小的 MLP，参与运算的 token 数量也不多，这也就导致了大量低运算量的 kernel launch。

上面的两个问题都是由 kernel launch 引起的（无法提前启动；大量小 kernel 启动），vLLM 中选择的方法是：

模型初始化时，所有 Expert 的权重会被连续存储在一个 Tensor 里，`gate` 和 `up` 存储在 `w13_weight` 中， `down` 被存储在 `w2_weight` 中。

```python
        self.w13_weight = nn.Parameter(
            torch.empty(self.num_total_experts,
                        2 * self.intermediate_size,
                        self.hidden_size,
                        dtype=params_dtype))
        self.w2_weight = nn.Parameter(
            torch.empty(self.num_total_experts,
                        self.hidden_size,
                        self.intermediate_size,
                        dtype=params_dtype))
```

Gate 输出的 Expert id 不再直接拿来划分 Token，而是用自定义的 kernel 对齐到 block size，方便后续计算

```python
    sorted_token_ids, expert_ids, num_tokens_post_padded = moe_align_block_size(
        topk_ids, config['BLOCK_SIZE_M'], E)
```

再然后就是 fused_moe kernel。可以看作是带 expert id 的 MLP 实现

```python
    invoke_fused_moe_kernel(hidden_states,
                            w1,
                            intermediate_cache1,
                            a1_scale,
                            w1_scale,
                            topk_weights,
                            topk_ids,
                            sorted_token_ids,
                            expert_ids,
                            num_tokens_post_padded,
                            False,
                            topk_ids.shape[1],
                            config,
                            compute_type=compute_type,
                            use_fp8=use_fp8)

    ops.silu_and_mul(intermediate_cache2, intermediate_cache1.view(-1, N))

    invoke_fused_moe_kernel(intermediate_cache2,
                            w2,
                            intermediate_cache3,
                            a2_scale,
                            w2_scale,
                            topk_weights,
                            topk_ids,
                            sorted_token_ids,
                            expert_ids,
                            num_tokens_post_padded,
                            True,
                            1,
                            config,
                            compute_type=compute_type,
                            use_fp8=use_fp8)
```

token expert 的选择被放到了 align block size，大量的 expert model 被融合成一个 fused MLP，是非常合理的做法，也难怪我被吊打了。

## 结尾

以前想在这个 blog 里做一些和工作内容相关的内容，倒是没想到契机是这个...

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
