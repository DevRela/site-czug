.. Contents::
.. sectnum::

为什么pagelet?
=====================
网站的页面，有很多一致的地方，只有少数内容是不同的。

使用pagelet可以统一管理相同的地方，让程序员把焦点集中在不同的内容区域，简化和分离开发。

主模板
====================
使用主模板来定义网站相同的地方。

为了方便定制，主模板同时提供用于各个页面定制的插槽:

- 正文插槽，这就是所谓pagelet
- css/javascript/kss的插槽
- 左边插槽，右边插槽

主模板的设计比较精细，一般有经验的人负责设计，普通程序主要在于使用即可。这里不详细介绍。

一个基础的pagelet页面
========================
写法和标准的browser:page browser:view类似，先写一个view类。

然后注册到主模板上::


添加js
==================

全局的js
----------------------

仅仅对当前的页面
-----------------------------

添加css
======================
全局的css
------------------

仅对当前的页面
-----------------------

实例：隐藏左侧列
---------------------------

添加kss
==================

全局的css
------------------

仅对当前的页面
-----------------------

添加左/右侧面板
==========================

编写zpt
---------------

编写viewlet
---------------

注册
--------

