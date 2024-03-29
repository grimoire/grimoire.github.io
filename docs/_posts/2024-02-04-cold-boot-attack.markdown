---
layout: post
title:  "【技术分享】cold boot attack——你的RAM不错哦，让我康康！"
date:   2024-02-04 19:00:000 +0800
categories: 技术分享
lang: zh-CN
---

如果大家看过电影《孤注一掷》的话，应该对剧中主角保护证物硬盘的桥段印象深刻吧。由于 ROM 的`非易失性`的特征，即使电脑由于损坏无法使用，也可以用技术手段恢复其中的数据，所以不小心保护自己的硬盘可是会[吃苦头](https://zh.wikipedia.org/zh-cn/%E9%99%B3%E5%86%A0%E5%B8%8C)的哦。

鉴于 ROM 这种非易失性的特性，它具有更大的安全风险，对于硬盘数据攻防的研究层出不穷；相对的，需要`持续供电`才能保存数据的 RAM 通常会被认为是更安全的。但是也有例外，比如 [这个项目](https://github.com/anfractuosity/ramrecovery)，使用 [cold boot attack](https://en.wikipedia.org/wiki/Cold_boot_attack)（以下用冷启动攻击代称） 来读取内存中的数据。那么到底冷启动攻击是什么呢？这就

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/letmeseesee.png?raw=true" alt="让我康康！" style="width:384px;"/>

{% include note.html content="这是一篇面向外行的科普。鉴于我并没有相关领域的经验，难免在用词甚至内容上存在错误，如果有哪位观众老爷发现问题，欢迎在下面评论中指出。" %}

## RAM 和低温休眠仓

RAM 大致可以分为两类，SRAM 和 DRAM，前者的使用[触发器](https://en.wikipedia.org/wiki/Flip-flop_(electronics)) 来实现，而后者则靠一个定期刷新的 MOS 电容来实现。

<img src="https://upload.wikimedia.org/wikipedia/commons/3/31/SRAM_Cell_%286_Transistors%29.svg" alt="sram" style="width:256px;"/>
<img src="https://upload.wikimedia.org/wikipedia/commons/b/bd/DRAM_Cell_Structure_%28Model_of_Single_Circuit_Cell%29.PNG" alt="dram" style="width:256px;"/>

不管哪一种，本质都是根据`电容`是否存储电荷来存储数据的。电容中的电荷在失去能量供给后会被释放，也就失去了记录数据的能力。

但是由于放电本身也需要时间，因此失去能量供给的 RAM 还是能在很短的时间内维持数据。特定条件下，这种维持数据的能力甚至能持续数小时甚至一周！这种现象被称为 [Data remanence](https://en.wikipedia.org/wiki/Data_remanence#Data_in_RAM)。

那么这种条件是什么呢？

科幻作品中经常会能看到低温休眠仓或人体冷冻技术，通过降低体温进入休眠状态，维持人体最低程度的生命体征，以期能够在未来重新唤醒。

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/cryonics.jpg?raw=true" alt="cryonics" style="width:512px;"/>

对于 RAM 而言，这并非科幻而是现实！低温真的会延长数据残留的时间。在 ["Cold-Boot Resistant Implementation of AES in the Linux Kernel"](https://faui1-files.cs.fau.de/filepool/thesis/diplomarbeit-2010-mueller.pdf) 中，作者将内存冷冻到特性-50摄氏度，内存中的数据在数分钟内几乎没有损失，之后数据才开始慢慢丢失。

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/freeze_DIMM.png?raw=true" alt="freeze" style="width:800px;"/>

## Cold Boot Attack

一些保密程度较高的项目通常不允许对硬盘中的数据进行拷贝，硬盘中的数据也会做加密，这样就算拿到硬盘也未必可以得到想要的数据。

相对的`读入内存`的数据通常是`未加密`的，加密相关的`密钥信息`也可能被保存在内存中，但内存的访问又受到操作系统的保护。这种时候就轮到冷启动攻击大展身手了。

这种攻击有一定的时效性，攻击者要有备而来，对内存进行：

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/touxi.png?raw=true" alt="偷袭！" style="width:256px;"/>

1. 准备一个可以用闪存启动的迷你操作系统
2. 对系统进行冷重启（也就是断电重启）来启动这个迷你操作系统
3. 操作及时的话，内存数据尚未丢失，可以使用迷你系统 dump 出数据

这样我们就可以访问那些受原本操作系统保护的内存数据了。

智能手机通常不存在 reset 按钮，可以直接通过`拔电池`来冷重启；实在无法冷重启的话，还可以直接`拔内存`，总之就别讲什么武德了。

这些方案在切换系统时的时间花销会很长，我们前面提到的通过`低温延长数据残留时间`的方法就可以派上用场了。准备好液氮或者冷却喷雾吧

<img src="https://upload.wikimedia.org/wikipedia/commons/7/7c/K%C3%A4ltespray_2009_PD_IMG_5196.JPG" alt="freeze spray" style="width:256px;"/>

{% include note.html content="冷启动攻击中的`冷`指的是指重启方式，低温能增加成功率，但非必须，比如下面的例子。" %}

[@hanfractuosity](https://github.com/anfractuosity) 在 github 上发布了一个[特别有意思的项目](https://github.com/anfractuosity/ramrecovery)以展示这种攻击。

<img src="https://github.com/anfractuosity/ramrecovery/blob/main/images/setup.jpg?raw=true" alt="ramrecovery" style="width:512px;"/>

整个项目由两个树梅派，一个修改过的 USB hub，以及一个裸金属 kernel 组成。

1. 树梅派A会把一张蒙娜丽莎写入内存
2. 断电重启后，USB hub 会在一个指定的延迟后，让树梅派A使用闪存中的系统
3. 切换系统后，裸金属会读取树梅派A中残留的内存数据
4. 残留的数据被写入树梅派B

由于有定制的 USB hub 来快速冷重启切换系统，因此不需要用冷冻法来延缓数据丢失。如果把冷启动的延迟设置为 0.75 秒，那么恢复的蒙娜丽莎就像下面这样

<img src="https://github.com/anfractuosity/ramrecovery/blob/main/images/image_0.75.png?raw=true" alt="image_0.75" style="width:512px;"/>

尽管出现了很多噪声，但是大致还是能看出原图的。根据作者的说法，如果不进行冷却的话，大约 1s 以后数据基本就全部丢失了。

## 对策

根据 [wiki](https://en.wikipedia.org/wiki/Cold_boot_attack#Effective_countermeasures)，这种攻击方案有一些常见的防御手段：

1. `物理隔离`：机房锁死，内存焊死，那么冷启动这个前提就不存在了。
2. `内存加密`：对内存数据做加密以防止攻击者得到真实数据。加密的密钥可以写在寄存器中。
3. `清理数据`：在重启的时候根据上次关机的状态清除数据；或者异常关机时随机写入脏数据。
4. `外部密钥`：不向内存里写入敏感信息，使用外部的硬件密钥来代替（类似银行的 U 盾之类）。

[这里](https://www.youtube.com/watch?v=uxjpmc8ZIxM&ab_channel=GoogleTechTalks)有 XBOX 对其安全方案的设计，其中就有涉及到的`硬件内存加密`，感兴趣的观众可以去看看，我看不懂😎

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/xbox_mem_hash.png?raw=true" alt="xbox_mem_hashing" style="width:512px;"/>

## 结语

第一篇技术分享博客完成啦！尽管和我从事的工作没啥关系，不过查资料的时候还是蛮开心的。希望能让大家有所收获。

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
