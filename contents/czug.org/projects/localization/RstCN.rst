---
created: 2006-09-15 08:46:53
creator: panjy
description: reStructuredText 是一个功能强大的“所想即所得”文本编写格式，但是目前对中文支持不佳。
title: 新结构化文本的中文支持
---
reStructuredText_ 是一个功能强大的“所想即所得”文本编写格式，但是目前对中文支持不佳。

.. _reStructuredText: /docs/plone/plonebook/X_e6_96_b0_e7_bb_93_e6_9e_84_e5_8c_96_e6_96_87_e6_9c_ac

主要问题：

- 内嵌文本的修饰，需要多余的空格。

  需要调整断字规则
- 表格支持非常差

  表格可以采用全角的方式处理
- 提示信息的中文输出
- 文本元数据中文支持

  参考docutils/languages/
- 中文的PDF输出支持

  参考：<a href="http://www.zope.org.tw/Members/joeysj/docs/BackTalk_patch_for_big5_pdf/view">http://www.zope.org.tw/Members/joeysj/docs/BackTalk_patch_for_big5_pdf/view</a>


参考：日文的补丁：
http://city.plala.jp/download/rst