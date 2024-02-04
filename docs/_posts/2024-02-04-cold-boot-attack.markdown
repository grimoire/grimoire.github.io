---
layout: post
title:  "【技术分享】cold boot attack——你的RAM不错哦，让我康康！"
date:   2024-02-04 14:00:000 +0800
categories: 技术分享
lang: zh-CN
---

如果大家看过电影《孤注一掷》的话，应该对剧中主角保护证物硬盘的桥段印象深刻吧。由于 ROM 的`非遗失性`的特征，即使电脑由于损坏无法使用，也可以用技术手段恢复其中的数据，不小心保护自己的硬盘可是会[吃大亏](https://zh.wikipedia.org/zh-cn/%E9%99%B3%E5%86%A0%E5%B8%8C)的哦。

鉴于 ROM 这种非遗失性的特性，对于硬盘数据的保护与解析的研究层出不穷，相对的，需要`持续供电`才能保存数据的 RAM 通常会被认为是更安全的。我一度也这么认为，直到看到 [这个项目](https://github.com/anfractuosity/ramrecovery)，使用 [cold boot attack](https://en.wikipedia.org/wiki/Cold_boot_attack) 来读取内存中的数据。那么到底 cold boot attack 是什么呢？下面就：

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/letmeseesee.png?raw=true" alt="让我康康！" style="width:512px;"/>

{% include note.html content="这是一篇面向外行的科普。鉴于我并没有相关领域的经验，难免在用词甚至内容上存在错误，如果有哪位观众老爷发现问题，欢迎在下面评论中指出。" %}

## RAM 的金鱼脑

RAM 大致可以分为两类，SRAM 和 DRAM，前者的使用[触发器](https://en.wikipedia.org/wiki/Flip-flop_(electronics)) 来实现，而后者则靠一个定期刷新的 MOS 电容来实现。

<img src="https://upload.wikimedia.org/wikipedia/commons/3/31/SRAM_Cell_%286_Transistors%29.svg" alt="sram" style="width:256px;"/>
<img src="https://upload.wikimedia.org/wikipedia/commons/b/bd/DRAM_Cell_Structure_%28Model_of_Single_Circuit_Cell%29.PNG" alt="dram" style="width:256px;"/>

不管哪一种，本质都是根据`电容`是否存储电荷来存储数据的。电容中的电荷在失去能量供给后会迅速释放，也就失去了记录数据的能力。

但是由于放电本身也需要时间，因此失去能量的 RAM 还是能在很短的时间内维持数据。而在特定条件下，这种维持数据的能力甚至能持续数小时甚至一周！这种现象被称为 [Data remanence](https://en.wikipedia.org/wiki/Data_remanence#Data_in_RAM)。

那么这种条件是什么呢？又如何利用这种最后的波纹呢？

科幻作品中经常会能看到低温休眠仓或人体冷冻技术，通过降低体温并使人进入休眠状态，最低程度维持人体的生命体征，以期能够在未来重新唤醒。

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/cryonics.jpg?raw=true" alt="cryonics" style="width:512px;"/>

这一现在对 RAM 也有效！在 ["Cold-Boot Resistant Implementation of AES in the Linux Kernel"](https://faui1-files.cs.fau.de/filepool/thesis/diplomarbeit-2010-mueller.pdf) 中，作者通过将内存冷冻到特性-50摄氏度，内存中的数据在数分钟内几乎没有损失

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/freeze_DIMM.png?raw=true" alt="freeze" style="width:512px;"/>

## Cold Boot Attack

一些保密程度较高的项目通常不允许对硬盘中的数据进行拷贝，硬盘中的数据也会做加密这样就算拿到硬盘也未必可以得到想要的数据。但是读入内存的数据通常是真实的，一些加密相关的信息也可能会在内存中残留，这种时候就轮到 Cold Boot Attack 大展身手了。

这种攻击有一定的时效性，攻击者要对内存进行：

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/touxi.png?raw=true" alt="偷袭！" style="width:512px;"/>

首先攻击者要有备而来：

1. 准备一个可以用闪存启动的迷你操作系统
2. 对系统进行冷重启（code reboot，也就是断电重启）来启动这个迷你操作系统
3. 这时内存中的数据可能还没来得及清空，使用闪存中的系统将内存中的数据拷贝出来

这样就可以随便访问内存中残留的数据了。

当然如果因为某些原因无法冷重启的话，还可以直接——`拔内存`...不过这种时间通常会更长，为了防止由于断电导致的数据丢失，可以使用冷冻喷雾或者液氮对内存进行降温，延长数据残留时间。

<img src="https://upload.wikimedia.org/wikipedia/commons/7/7c/K%C3%A4ltespray_2009_PD_IMG_5196.JPG" alt="freeze spray" style="width:256px;"/>

{% include note.html content="cold boot attack 中的 cold 指的是指 “冷启动”（断电重启），而不是靠低温降低易失性。" %}

[@hanfractuosity](https://github.com/anfractuosity) 在 github 上发布了一个[特别有意思的项目](https://github.com/anfractuosity/ramrecovery)以展示这种攻击。

<img src="https://github.com/anfractuosity/ramrecovery/blob/main/images/setup.jpg?raw=true" alt="ramrecovery" style="width:512px;"/>

整个项目由两个树梅派，一个修改过的 USB hub，以及一个裸金属 kernel 组成。

1. 树梅派A会把一张蒙娜丽莎写入内存
2. 断电重启后，USB hub 会在一个指定的延迟后，切换树梅派A的系统
3. 切换系统后，裸金属会读取树梅派A中残留的内存数据
4. 残留的数据被写入树梅派B

由于有定制的 USB hub 来快速冷重启切换系统，因此不需要用冷冻法来延缓数据丢失。如果把冷启动的延迟设置为 0.75 秒，那么恢复的蒙娜丽莎就像下面这样

<img src="https://github.com/anfractuosity/ramrecovery/blob/main/images/image_0.75.png?raw=true" alt="image_0.75" style="width:512px;"/>

尽管出现了很多噪声，但是大致还是能看出原图的。作者说如果不进行冷却的话，大约 1s 以后数据基本就全部丢失了。

## 对策

根据 [wiki](https://en.wikipedia.org/wiki/Cold_boot_attack#Effective_countermeasures)，这种攻击方案有一些常见的防御手段：

1. `物理隔离`：机房锁了，内存焊死了，那么冷启动这个前提就不存在了。
2. `内存加密`：既然访问的内存中的数据，那么就把这些数据也加密了，cpu 只访问一个被解密的内存页。加密的密钥可以写在寄存器中。
3. `清理数据`：在重启的时候根据上次关机的状态清除数据，或者异常关机时随机写入脏数据，都可以一定程度上降低被攻击的可能。
4. `外部密钥`：不向内存里写入敏感信息，使用外部的硬件密钥（可能类似银行的 U 盾哪种）。

[这里](https://www.youtube.com/watch?v=uxjpmc8ZIxM&ab_channel=GoogleTechTalks)有 XBOX 对其安全方案的设计，其中就有涉及到的硬件内存加密，感兴趣的观众可以去看看，我看不懂😎

<img src="https://github.com/grimoire/grimoire.github.io/blob/code-boot-attack-resources/resources/xbox_mem_hash.png?raw=true" alt="xbox_mem_hashing" style="width:512px;"/>

## 结语

第一篇技术分享博客完成啦！尽管和我从事的工作没啥关系，不过查资料的时候还是蛮开心的。希望能让大家有所收获。

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
