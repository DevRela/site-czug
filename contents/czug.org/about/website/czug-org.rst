---
created: 2006-08-24 23:32:11
creator: panjy
description: 介绍本网站上安装的产品、系统的配置，以及补丁等信息
title: CZUG.org内幕
---
 <p>主要安装的产品包括：</p>

 <p>系统软件</p>

 <ul>
  <li>操作系统：Linux 2.6内核 （Ubuntu）</li>

  <li>web服务器：Apache2 , 采用rewrite和zope配合</li>

  <li>web应用服务器：Zope2的最新版本</li>

  <li>内容管理平台：Plone 2.x</li>

  <li>web cache服务器：采用Squid，对系统进行加速</li>
 </ul>

 <p>系统配置结构为：</p>
 <pre>
                    Internet
                       |
                     Squid               --&gt; Squid进行web加速，对外的唯一入口
            ___________|__________
           |                      |
   ________|_____________       Apache   --&gt; Apache提供其他的动/静态服务
  |                      |      
ZopeAppServer1     ZopeAppServer2        --&gt; 2个zope应用服务器群集
  |______________________|
        |           |
ZEOStorageSever1 ZEOStorageServer2       --&gt; 数据库分布存储（DBTab）
</pre>

 <p>Zope产品</p>

 <ul>
  <li>CMFContentPanels</li>
  <li>Ploneboard</li>

  <li>MPoll</li>

  <li>SimpleBlog</li>

   <p>在zope配置文件中添加LC_ALL zh_CN的环境变量</p>
  </li>

  <li>CJKSplitter</li>

 </ul>

 <p>硬件</p>

 <ul>
  <li>服务器: 宝德PT600R（Intel s875部门级服务器主板，SATA高速硬盘，512M DDR
  ECC内存，P4 2.6G）</li>

  <li>IDC网络：位于上海环球网络机房</li>
 </ul>
