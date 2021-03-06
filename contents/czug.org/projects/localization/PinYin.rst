---
created: 2006-09-15 08:46:53
creator: panjy
description: 让plone支持支持汉字和拼音的自动转换
title: 拼音支持
---
.. Contents:: 目录

**在Plone2.1中，安装ZopeChinaPak后，可支持自动以拼音作为id**

背景
=====

在英文主宰的信息世界，汉字受到不公正的待遇:

- 在w3c的标准中，URL中是不支持中文字符的
- Zope的ID，必须为中文
- webdav的文件夹必须为中文
- 中文自身可能有多种编码，直接使用中文会带来很多的不一致问题

要解决这个问题，目前：

- 必须手工输入英文id，这个实在太难
- zwiki等提供了自动转换为内码的方式，但是导致链接十分的长，而且不直观。

解决方法
========

使用ascii字符来显示中文，拼音是一个不错的解决方案。

提交内容
========

1. 一个独立的python包
2. 对zwiki进行补丁
3. 修改Zope/CMF/Plone对上载的中文文件，自动把id转换为中文的拼音

问题
====

- 使用已经有相关的python包？

  xyb提供了一个 `pinyin_zopeext.py`__

  __ pinyin_zopeext.py
- 是否有现有的拼音—汉字库供使用？
- 是否支持繁、简等多种编码？



From Zoomq Tue Apr 13 17:27:07 +0800 2004
From: Zoomq
Date: Tue, 13 Apr 2004 17:27:07 +0800
Subject: 有必要嘛？
Message-ID: <20040414092707+0800@www.czug.org>

拼音本身也不过是外国人强制解析中文的成果！
并不能代表什么哪？

其实Python 本身可以非常好的支持全中文的程序，比如说中蟒 ， 只是因为互联网的各种网关，网桥等等硬件只能够解析ASII 的URL,才使Web应用不得不屈于英文……
考虑到外国朋友的理解，还不如使用

中英文对换！

这可是有大量的字典可以运用？！

From ccube Mon Apr 19 11:55:52 +0800 2004
From: ccube
Date: Mon, 19 Apr 2004 11:55:52 +0800
Subject: 同字
Message-ID: <20040420035552+0800@www.czug.org>

拼音也會有多字同碼的情況, 我在想的是和倉頡輸入法一樣的問題，多個字同一個碼怎處理。加數字？

還有一問，速度估計如何？

From panjy Mon Apr 19 12:49:11 +0800 2004
From: panjy
Date: Mon, 19 Apr 2004 12:49:11 +0800
Subject: re: 同字
Message-ID: <20040420044911+0800@www.czug.org>

在wiki种的确需要避免同字的问题。否则不同汉字生成的Wiki ID相同了。不知道是否有完全的拼音码表。

另外可能的一个问题是，一个汉字可能会多音，不知道用哪个。但也可以给一个粗略的方法：选择第一个。

性能问题，我觉得问题不大。因为仅仅是在新添加的时候，需要进行转换。


From xyb Mon Apr 19 14:22:58 +0800 2004
From: xyb
Date: Mon, 19 Apr 2004 14:22:58 +0800
Subject: 我以前写的一个 python module，也可以做为 zope external 来使用。
Message-ID: <20040420062258+0800@www.czug.org>

<a href="http://xie.freezope.org/upload/Files/pinyin_zopeext.py">http://xie.freezope.org/upload/Files/pinyin_zopeext.py</a>

From panjy Mon Apr 19 14:57:16 +0800 2004
From: panjy
Date: Mon, 19 Apr 2004 14:57:16 +0800
Subject: good, thanks xyb!
Message-ID: <20040420065716+0800@www.czug.org>

正是需要这个！ 中文的汉字看起来也不是那么多，一个map就搞定了 ;-)可暂时用这个。

前面我说“完全的拼音码表”，意思是，使用"拼音+数字"和汉字进行一一映射，来唯一标识一个汉字。




From panjy Mon Apr 19 15:02:46 +0800 2004
From: panjy
Date: Mon, 19 Apr 2004 15:02:46 +0800
Subject: 写个程序，可处理一下
Message-ID: <20040420070246+0800@www.czug.org>

基于上面已经有的拼音表，可写个程序，把重复的拼音附加一个数字编号，生成一个新的“拼音-汉字库”，就可以实现唯一标识汉字了，这个实现应该不难。

From hewei Mon Jun 28 22:39:58 +0800 2004
From: hewei
Date: Mon, 28 Jun 2004 22:39:58 +0800
Subject: 对于英文ID的个人见解
Message-ID: <20040629143958+0800@www.czug.org>

可能由于我的开发多少都和国外相关，我一直使用有意义的英文ID。而且我的观点是，英文ID或URL是网页内容的一个重要概括。

你们是否在检索网站时，得到过除中文、日文外的其它非英文网站结果？有时候页面的信息可能对你有用，但页面上你内容一点都不懂，是什么感觉？

同样的，我认为，使用16进制编码或拼音做ID，不仅不利于与国际接轨，对自己的意义也不见得很大。倒不如顺序编号好写好记。

From panjy Mon Jun 28 23:45:19 +0800 2004
From: panjy
Date: Mon, 28 Jun 2004 23:45:19 +0800
Subject: to hewei: 这个项目主要针对自动生成ID的场合
Message-ID: <20040629154519+0800@www.czug.org>

zwiki会自动生成id, 上载文件, 也会自动生成id. 这些id都会出现一大串乱码.

这个项目本意是想解决这个问题.