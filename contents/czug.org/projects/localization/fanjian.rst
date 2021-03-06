---
created: 2007-10-24 11:21:12
creator: panjy
description: 让Plone支持中文繁体和简体的自动转换
title: 中文繁简转换
---
项目起因
==============
1. 整理czug的过程中发现一些资料是台湾兄弟们整理的繁体文档，对于我们简体的读者，阅读还是存在一些不方便之处。

2. 网络上的确已经有一些免费转换工具了，但是我还是害怕病毒，不感下载。而且我现在也习惯了不使用盗版软件的习惯（已经有更好的开放源代码软件了，呵呵）

是否可以为czug写一个zope版的繁简转换程序呢？

要求
=============
1. 完成一个基础的中文UTF8编码的繁简转换包，采用标准的编码接口，比如::

    >>> '简体'.encode('zh-jianti') 
    簡體
    >>> '簡體'.encode('zh-fanti')
    简体

2. 在上面的基础上，完成一个zope/plone上的产品

  1. 即时转换，支持拷贝粘贴的方式继续翻译
  2. 文件方式翻译：支持转换整个文本文件
  3. 选择输出方式：可直接在界面上显示，或者选择下载为文件

用户使用案例 Use Case
======================
1. 用户输入需要转换的内容，可选择如下方式之一：

   - 用户粘贴所拷贝的文字到界面上的输入框
   - 通过上载框进行整个文件的上载. (第一期实现时，上载文件必须选择输入输出的编码类型，gb/utf8/big5等，以后可考虑自动识别）

2. 用户选择转换模式：

   - 繁转简
   - 简转繁

3. 用户设置输出方式：

   - 在线编辑
   - 文件下载

#. 用户设置输出编码：是否采用utf-8编码？

#. 用户执行转换

   点击转换按钮，提交表单进行转换。转换的结果为：

   - 如果用户选择在线编辑方式，则会将转换后的文本直接在线显示

   - 如果用户选择文件下载方式，则会出现用户浏览器上会下载对话框，用户下载保存转换后的内容。

实现方法
============
1. 考虑简化界面，考虑使用archetypes在plone中实现，可作为一个portal_gbbig5的工具的形式存在。用户在plone界面中访问portal_gbbig5就可以访问了

2. 这个工具包括如下字段:

   - 输入内容

   - 输入的编码

   - 转换方式

   - 输出编码

   - 输出方法

   - 输出的内容

3. 转换算法：参考limodou的实现

其他参考链接
==============
1. `青藤书屋 <http://artvine.com.tw/images/uu2.htm>`__ 提供了在线繁简转换工具，中西日历也很可爱。这是一个不错的样板。

2. `忆资繁简转换系统 <http://home.netvigator.com/~en0ch/MemGBBig5.htm>`__ 这是一个商业系统

3. `繁简转换网关服务器 <http://download.enet.com.cn/html/020192003012001.html>`__ 很好的想法，我们也可以实现:-)

4. `来网智能简繁转换系统 <http://www.laisoft.com/products/big5.htm>`__ 测试了一下在线转换，我用mozilla好像不行

5. `office上的繁简转换宏 <http://download.pchome.net/utility/lan/gbbig5/11265.html>`__ 这个想法也不错

6. `FrontPage 上也能够进行繁简转换 <http://www.pconline.com.cn/pcedu/soft/wl/brower/10111/821.html>`__

7. `google上更多 <http://www.google.com/search?q=+%E7%B9%81%E4%BD%93+%E7%AE%80%E4%BD%93+%E8%BD%AC%E6%8D%A2&sourceid=mozilla-search&start=0&start=0&ie=utf-8&oe=utf-8>`__

From hewei Mon Jun 28 22:24:18 +0800 2004
Subject: 词汇转换问题

没有词汇转换功能的产品效果将大打折扣。Office支持繁简词汇转换但我个人感觉不太理想，不值得挖出来。自己做一个吧。

要进行词汇转换，首先要将句子进行断词处理。这个我正在开发，已经差不多了。现在应该考虑建立和维护一个繁简词汇对照（带例句的更好）数据库了。以便供将来的繁简转换产品使用。

From panjy Mon Jun 28 23:47:07 +0800 2004
Subject: 是需要一个繁简对照词汇表

是需要一个繁简对照词汇表. 同时, 如果能够有语义上的分析, 才能解决可能出现的词汇误换的问题.

From xyb Tue Jun 29 14:59:49 +0800 2004
Subject: “同文”的对照表整理的不错，而且算法也很好

值得我们借鉴： http://tongwen.mozdev.org/

From panjy Tue Jun 29 15:47:12 +0800 2004
Subject: 基于浏览器的繁简转换, 的确是高招:)

很不错的, 很通用, 很方便.

不过我们可能要求一些个性化的转换, 如把繁体的"程式", 转换为"程序"之类的东西.


From xyb Tue Jun 29 15:54:44 +0800 2004
Subject: 可以在同文的算法基础上稍加改进

就可以做成通用的算法。比如，把我们的映射表建立成一个树状结构，在这个树里对翻译文章的汉字进行遍历，找到最长匹配路径，就是翻译出的内容。

From hewei Wed Jun 30 00:42:29 +0800 2004
Subject: 最长匹配路径算法不是最好的

在我的尚未发布的中文索引系统中采用了 2 words lookahead + maxinum path + frequency 的综合算法，初步测试效果不错。所有发现的错误断词都可以通过扩充词库得到更正。

我的产品现在缺少一套词典维护系统，包括分科的词典，词目增删以及特殊断词修正算法等。我想全部做成Zope/Plone产品。

还有就是目前这个词典通过MySQL保存。有谁知道转换成ZODB是能够通过降低查询的overhead而获得更高的效率？

From xyb Tue Aug 3 10:47:55 +0800 2004
Subject: 项目名称可以叫 zhconv

另外，hewei 的系统不知道做的怎么样了？能提供专用于转换繁简的接口吗？会用开源的协议放出来吗？

From hewei Wed Aug 4 12:27:39 +0800 2004
Subject: 繁简依靠词典

会是开源的。但开发工作现在被其它事情耽搁了。

From Zoomq Thu Aug 5 14:59:00 +0800 2004
Subject: 用 NLP 來解決目前簡轉繁那麼多錯別字


http://ljm.idv.tw/mywiki/NaturalLanguageProcessingFightingSpam
嗯嗯,台湾朋友也开始有新的想法了!大家一起参照一下子!???!

From Zoomq Thu Aug 5 14:59:27 +0800 2004
Subject: 編輯簡繁訂正詞庫

http://ljm.idv.tw/mywiki/SimplifiedToTraditionalLexicon

From panjy Fri Dec 31 08:31:02 +0800 2004

一个类似的zope产品

这个是spellcheck：功能类似，可做开发参考：

http://zope.org/Members/EIONET/SpellChecker/0.7/News_Item.2004-12-10.4950


`wikipedia的繁简考虑 <http://zh.wikipedia.org/wiki/Wikipedia:1.4%E7%89%88%E7%9A%84%E7%B9%81%E7%AE%80%E5%A4%84%E7%90%86#.E7.B3.BB.E7.BB.9F.E9.BB.98.E8.AE.A4.E8.BD.AC.E6.8D.A2.E8.A1.A8>`__


From panjy Sun Jan 9 01:49:44 +0800 2005
Subject: 一个繁简转换的帖子

http://www.longwin.com.tw/~jon/blog/archives/000407.html

From panjy Wed Jun 8 09:08:14 +0800 2005
Subject: perl上的繁简转换模块

http://search.cpan.org/~autrijus/Encode-HanConvert/

http://blog.autrijus.org/archives/000024.html
