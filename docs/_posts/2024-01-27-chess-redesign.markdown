---
layout: post
title:  "【HN趣闻】 一看就懂的国际象棋棋子设计"
date:   2024-01-27 14:30:000 +0800
categories: hacker-news HN趣闻
lang: zh-CN
---

做为一个有原则的开源项目贡献者，我始终秉持着“<span style="color:red">亲爱的用户，我是你爹</span>”的准则，因此在用户体验的设计方面没少被我的领导怼。痛定思痛，我开始探索一些用户友善的 API 设计，也因此发现 hacker news 上一篇有趣的 [post](https://news.ycombinator.com/item?id=39128902)，下面就对 post 的评论做一些整理和分享，希望大家喜欢。

{% include note.html content="下面的分享内容主要时关于国际象棋，鉴于中文读者可能更习惯于中国象棋，因此这里会使用“车，马，相”等中国象棋中的称谓来取代“城堡，骑士，主教”等称呼。" %}

## 面向新手的棋子设计

话题的开始源于一篇 [twitter](https://twitter.com/graycrawford/status/1750225562281632048)

> 我意识到可以用攻击方向来设计棋子

<img src="https://pbs.twimg.com/media/GEoKM4JbQAAxQtJ?format=jpg&name=4096x4096" alt="redesign1" style="width:256px;"/>
<img src="https://pbs.twimg.com/media/GEoKRSKaUAACyHN?format=jpg&name=4096x4096" alt="redesign2" style="width:256px;"/>

这种想法不算新鲜。事实上，早在 1923 年，包豪斯的雕塑家 [Josef Hartwig](https://en.wikipedia.org/wiki/Josef_Hartwig#Chess_sets) 就设计过一套 [Bauhaus-Schachspiel](https://de.wikipedia.org/wiki/Bauhaus-Schachspiel)。这套棋子有典型的包豪斯风格，采用简单的几何图形来表示各个棋子，比如象是 X 型，马是 L 型，对于刚接触国际象棋的人来说更容易理解。

不过55～155马克也非常有包豪斯风格。对于当时刚经历一战战败，穷的荡气回肠的德国人而言，多少有点高攀不起了。

<img src="https://upload.wikimedia.org/wikipedia/commons/5/55/Bauhaus_chess_set.jpg" alt="Bauhaus-Schachspiel" style="width:512px;"/>

之后也一直有人探索这种把功能融合进棋子的设计，比如下面这套，将“功能”和“美观”分别投影在两个方向上。

<img src="https://i.ebayimg.com/images/g/Yx4AAOSwTY9i7XeC/s-l960.jpg" alt="redesign3" style="width:256px;"/>
<img src="https://i.ebayimg.com/images/g/AZUAAOSwuABi7XeE/s-l960.jpg" alt="redesign2" style="width:256px;"/>

或者索性提供一个带教程的底座。

<img src="https://i.ebayimg.com/images/g/gQQAAOSwOBRlYOTy/s-l1600.jpg" alt="redesign3" style="width:512px;"/>

这种设计的初衷是为了让新手能够更快的上手，熟悉棋子走法过则。不过除了走法以外，国际象棋还有很多额外的规则，比如：

1. 卒可以走直线和斜线，初始时可以选择走两格
2. 卒可以进行 [en passant](https://en.wikipedia.org/wiki/En_passant)
3. 卒可以升级成其他棋子
4. 马的移动不会受到周围棋子的影响（国际象棋中并不存在“卡马脚”的操作）
5. 王车易位

这些规则也使得单纯使用几何图形的棋子设计存在一些缺陷，比如不慎将象旋转了45度的话，象可能被误认成车等等...这方面讨论比较多的是马的设计。

由于马的走法比较特殊，因此出现了两种主张:

- 一种和作者的设计相同，觉得棋子应该指出马的<span style="color:red">前进方向</span>

<img src="https://pbs.twimg.com/media/GEobORAbkAACIxg?format=jpg&name=small" alt="kight1" style="width:256px;"/>

这种设计的主要问题是，它没有描述“马只能按照切比雪夫距离前进2格”的限制，会使得用户理解错误，比如 [Grant Acedrex](https://en.wikipedia.org/wiki/Grant_Acedrex) (一种国际象棋的变体) 中 Unicornio 的走法：

<img src="https://github.com/grimoire/grimoire.github.io/blob/chess-resources/resources/unicornio.png?raw=true" alt="Unicornio" style="width:256px;"/>

- 另一种设计是主张“马的移动方式是<span style="color:red">进2平1</span>”，基于这一考量，主要会产生 L 型或条顿骑士团徽章形状的设计。

<img src="https://pbs.twimg.com/media/GEqoWAFXQAAPRXm?format=jpg&name=small" alt="kight2" style="width:256px;"/>

这样能正确描述“有限的移动距离”。但是会让用户误以为马的可能会被紧贴的棋子阻碍移动（再次强调，国际象棋中不存在卡马脚），因此也许更适合中国象棋而不是国际象棋。

哪种设计更合理是没有定论的，说到底这种功能化的设计主要是帮助用户熟悉规则，快速上手，体会游戏的乐趣。之后不管用户使用这种功能性棋子还是漂亮的艺术棋子都没差，相当于在电子游戏里换皮肤了。

## 非正规国际象棋

另一个比较有趣的话题是关于历史上出现过的一些国际象棋的变体。
