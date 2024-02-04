---
layout: post
title:  "【HN趣闻】 俺寻思国际象棋应该这么搞..."
date:   2024-01-28 01:27:000 +0800
categories: hacker-news HN趣闻
lang: zh-CN
---

做为一个有原则的开源项目贡献者，我始终秉持着“<span style="color:red">亲爱的用户，我是你爹</span>”的准则，因此在用户体验的设计方面没少被我的领导怼。痛定思痛，我开始探索一些用户友善的 API 设计，也因此发现 hacker news 上一篇有趣的 [post](https://news.ycombinator.com/item?id=39128902)，下面就对 post 的评论做一些整理和分享，希望大家喜欢。

{% include note.html content="下面的分享内容主要关于国际象棋，鉴于中文读者可能更习惯于中国象棋，因此这里会使用“车，马，象”等中国象棋中的称谓来取代“城堡，骑士，主教”等称呼。" %}

## 面向新手的棋子设计

话题的开始源于一篇 [twitter](https://twitter.com/graycrawford/status/1750225562281632048)

> 我意识到可以用攻击方向来设计棋子

<img src="https://pbs.twimg.com/media/GEoKM4JbQAAxQtJ?format=jpg&name=4096x4096" alt="redesign1" style="width:256px;"/>
<img src="https://pbs.twimg.com/media/GEoKRSKaUAACyHN?format=jpg&name=4096x4096" alt="redesign2" style="width:256px;"/>

这种想法不算新鲜。事实上，早在 1923 年，包豪斯的雕塑家 [Josef Hartwig](https://en.wikipedia.org/wiki/Josef_Hartwig#Chess_sets) 就设计过一套 [Bauhaus-Schachspiel](https://de.wikipedia.org/wiki/Bauhaus-Schachspiel)。这套棋子有典型的包豪斯风格，采用简单的几何图形来表示各个棋子，比如象是 X 型，马是 L 型，对于刚接触国际象棋的人来说更容易理解。

不过55～155马克的价格也非常有包豪斯风格。对于当时刚经历一战战败，穷的荡气回肠，大负大跪的德国人而言，多少有点高攀不起了。

<img src="https://upload.wikimedia.org/wikipedia/commons/5/55/Bauhaus_chess_set.jpg" alt="Bauhaus-Schachspiel" style="width:512px;"/>

之后也一直有人探索这种把功能融合进棋子的设计，比如下面这套，将“功能”和“美观”分别投影在“侧面”与“上面”两个方向上。

<img src="https://i.ebayimg.com/images/g/Yx4AAOSwTY9i7XeC/s-l960.jpg" alt="redesign3" style="width:256px;"/>
<img src="https://i.ebayimg.com/images/g/AZUAAOSwuABi7XeE/s-l960.jpg" alt="redesign2" style="width:256px;"/>

或者索性提供一个带教程的底座。

<img src="https://i.ebayimg.com/images/g/gQQAAOSwOBRlYOTy/s-l1600.jpg" alt="redesign3" style="width:512px;"/>

这种设计的初衷是为了让新手能够更快的上手，熟悉棋子走法规则。不过除了走法以外，国际象棋还有很多额外的规则，比如：

1. 卒可以走直线和斜线，初始时可以选择走两格
2. 卒可以进行 [en passant](https://en.wikipedia.org/wiki/En_passant) （吃过路兵）
3. 卒可以升级成其他棋子
4. 马的移动不会受到周围棋子的影响（国际象棋中并不存在“卡马脚”的操作）
5. 王车易位

这些规则很难只用几何设计来描述。另外上面的设计还有别的问题，比如不慎将象旋转了45度的话，象可能被误认成车等等...

其中被讨论最多的问题就是马的设计，由于马的走法比较特殊，关于它的设计有两派意见:

- 一种和作者的设计相同，觉得棋子应该指出马的<span style="color:red">前进方向</span>

<img src="https://pbs.twimg.com/media/GEobORAbkAACIxg?format=jpg&name=small" alt="kight1" style="width:256px;"/>

这种设计的主要问题是，它没有描述`马只能按照切比雪夫距离前进2格`的限制，可能会使得用户理解错误，比如 ：

{% include posts/chess-redesign/unicornio.html %}
[西班牙大象棋](https://en.wikipedia.org/wiki/Grant_Acedrex) (一种国际象棋的变体) 中`犀牛`的走法

- 另一种设计是主张“马的移动方式是`进2平1`”，基于这一考量，会产生 L 型或条顿骑士团徽章形状的设计。

<img src="https://pbs.twimg.com/media/GEqoWAFXQAAPRXm?format=jpg&name=small" alt="kight2" style="width:256px;"/>

这样能正确描述`有限的移动距离`。但是会让用户误以为马的可能会被紧贴的棋子阻碍移动（再次强调，国际象棋中**不存在**卡马脚）。这样的设计也许更适合中国象棋而不是国际象棋。

哪种设计更合理是没有定论的，说到底这种功能化的棋子设计主要是用来帮助用户熟悉规则，快速上手，体会游戏的乐趣。熟练以后，不管用户使用这种功能性棋子还是漂亮的艺术棋子都没差，相当于在电子游戏里换皮肤了。

## 非正规国际象棋

另一个比较有趣的话题是关于历史上出现过的一些国际象棋的变体。

国际象棋通常被认为是从印度7世纪发明的游戏 [查图兰加](https://en.wikipedia.org/wiki/Chaturanga) 演变而来，在这个过程中经过各种国家/地区，产生了许多变体。尽管由于种种原因它们没有成为主流，但是他们的设计还是很有趣的。

### 皇后的诞生

大家可能都会有一个疑问，为什么国际象棋中最强大的单位是“皇后”？事实上在前面提到的象棋起源，原本占据皇后位置的单位是 [Ferz](https://en.wikipedia.org/wiki/Ferz)，名字来源于波斯语的 “farzin”，代表“首相”、“大臣”或“贤者”。

当时这个单位的功能更像是中国象棋中的`仕`，只能`斜向移动一格`。日本的大将棋中也有名为“猫刃”的变体。

<img src="https://fondodx.com/053d2e7a57ec/neko.jpg" alt="kight2" style="width:256px;"/>

传到欧洲后随时代演变，慢慢出现了 “Queen” 或 “lady” 之类的称呼，阿拉伯世界则会称其为“维齐尔”（帝国时代或十字军之王的玩家应该不陌生吧）。之后随着伊比利亚半岛上的[卡斯蒂利亚的布兰卡](https://en.wikipedia.org/wiki/Blanche_of_Castile)以及[伊莎贝尔一世](https://en.wikipedia.org/wiki/Isabella_I_of_Castile)等著名的女性君王出现，以及基督教对圣母的崇拜等原因，“Queen”这个称呼才算是正式固定下来。

15世纪左右，Queen 在西班牙得到了史诗强化（也就是现在的 车+象 的移动模式），然后藉由印刷术的发展传向欧洲各地。鉴于当时欧洲基督教世界极端的厌女倾向，这种玩法也被蔑称为“疯女人棋”。

### 未被启动的神话棋子

并非每个棋子都有皇后这么好运，被正式版收录还得到强化。还有大量未被正式采用的棋子，这类棋子被称为神话棋子（[Fairy chess piece](https://en.wikipedia.org/wiki/Fairy_chess_piece)）

皇后可以被看作是 `车+象` 的结合，那么当然，历史上也出现过 `车+马` 结合和 `象+马` 结合的棋子。他们就是 [女皇](https://en.wikipedia.org/wiki/Empress_(chess)) 和 [公主](https://en.wikipedia.org/wiki/Princess_(chess))，可以说是“妇女能顶半边天”了。（不过实际上它们的名字就和皇后 Queen 一样，是随着历史不断演变的）

<img src="https://upload.wikimedia.org/wikipedia/commons/d/d9/Chess_clt45.svg" alt="empress" style="width:128px;"/>
<img src="https://upload.wikimedia.org/wikipedia/commons/a/a5/Chess_alt45.svg" alt="princess" style="width:128px;"/>

当然，最 IMBA 的还要数[亚马逊](https://en.wikipedia.org/wiki/Amazon_(chess))了，彻底发扬了`我都要`的精神，就算发给[烈焰红唇](https://www.youtube.com/@Harstem)他应该也说不出 `YOU SUCK` 了。

{% include posts/chess-redesign/amazon.html %}

还有像 [西班牙大象棋](https://en.wikipedia.org/wiki/Grant_Acedrex) 这种加了一大堆 mod，什么长颈鹿、犀牛、鳄鱼、狮子和...呃...狮鹫。让我想起了——

<img src="https://images.saymedia-content.com/.image/c_limit%2Ccs_srgb%2Cq_auto:eco%2Cw_700/MTc2NzIzNjA4OTY5Njg0OTcx/heroes-of-might-and-magic-iii-the-restoration-of-erathia-game-review.webp" alt="heroes3" style="width:512px;"/>

### 变种棋盘

除了在棋子上下文章以外，棋盘也可以做很多创新，最常见的自然是改变形状，比如六边形的 [hexagonal chess](https://en.wikipedia.org/wiki/Hexagonal_chess#) (这下更像英雄无敌了)，或者是环形的 [circular chess](https://en.wikipedia.org/wiki/Circular_chess)

<img src="https://upload.wikimedia.org/wikipedia/commons/8/84/Hexagonal_chess.svg" alt="hexagonal" style="width:256px;"/>
<img src="https://upload.wikimedia.org/wikipedia/commons/4/45/Rundskak_foto.jpg" alt="circular" style="width:256px;"/>

除了二维棋盘以外，还有更高纬度的

<img src="https://upload.wikimedia.org/wikipedia/commons/8/8e/Rhombic_Chess_gameboard_and_starting_position.png" alt="Rhombic" style="width:256px;"/>
<img src="https://upload.wikimedia.org/wikipedia/commons/f/f8/Thrones_Chess_initial_setup.png" alt="Thrones" style="width:256px;"/>

这么多变体中比较著名的是 [三维国际象棋](https://en.wikipedia.org/wiki/Three-dimensional_chess#Raumschach)。最早在德国由 Ferdinand Maack 发明，可供两个人在 5x5x5 的棋盘上对战。他在1919年创建了一个三维国际象棋俱乐部，受二战影响终止了。棋盘大致长这个样子：

<img src="https://upload.wikimedia.org/wikipedia/commons/c/cb/Star_trek_chessboard.JPG" alt="3d chess" style="width:256px;"/>

尽管这款棋盘的寿命不长，但是它对后世依然有不小的影响力，《星际迷航》和《生活大爆炸》中都曾出现过：

<img src="https://upload.wikimedia.org/wikipedia/en/4/4b/StarTrekChess.jpg" alt="3d chess" style="width:256px;"/>
<img src="https://i.stack.imgur.com/sYonR.jpg" alt="3d chess" style="width:256px;"/>

另外 `Three-dimensional chess` （或者 4d/5d ...）也成为了一句俗语，用来描述那些复杂抽象的事物。比如 2016 年的某国大选，由于结果太过抽象，被说成是在玩 [4d chess](https://knowyourmeme.com/memes/trump-is-playing-4d-chess) ...

如果大家对自己的大脑足够有自信，可以用 [这个软件](http://www.parmen.com/) 玩玩看。

## 结语

今天分享的就这些了，也不知道合不合各位观众老爷的胃口。这里先给大家拜个早年吧，祝大家新年快乐，龙年吉祥～

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
