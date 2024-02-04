---
layout: post
title:  "【技术分享】cold boot attack——你的RAM不错哦，让我康康！"
date:   2024-02-04 14:00:000 +0800
categories: 技术分享
lang: zh-CN
---

<!-- 引言，顺便加 buff -->
如果大家看过电影《孤注一掷》的话，应该对剧中主角保护证物硬盘的桥段印象深刻吧。由于硬盘`非遗失性`的特征，即使电脑由于损坏无法使用，也可以用技术手段恢复其中的数据，不小心保护自己的硬盘可是会[吃大亏](https://zh.wikipedia.org/zh-cn/%E9%99%B3%E5%86%A0%E5%B8%8C)的哦。

鉴于这种非遗失性的特性，对于硬盘数据的保护与解析的研究层出不穷，相对的，需要`持续供电`才能保存数据的 RAM 通常会被认为是更安全的。我一度也这么认为，直到看到 [这个项目](https://github.com/anfractuosity/ramrecovery)，使用 [cold boot attack](https://en.wikipedia.org/wiki/Cold_boot_attack) 来读取内存中的数据。那么到底 cold boot attack 是什么呢？下面就：

<img src="" alt="让我康康！" style="width:512px;"/>

<!-- 介绍 cold boot attack 是什么-->

<!-- 介绍树梅派 exploit 项目-->

<!-- 结语 -->

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
