---
created: 2005-12-19 16:54:13
creator: taoyh
description: 主要作用是控制各个文件的浏览权限，对增加某个内容没有做详细解释。
title: 建立一个私有的Plone站点
---
<h2 class="Heading"><br />这儿是建立一个私有plone站点的基本步骤</h2><ul><li><div class="Heading">从ZMI（zope managemnet interface）进入你的plone站点，也就是一个plone目录。</div></li><li>在plone站点的根目录找到security 标签页，找到 Add portal member项，仅仅许可manage和owner，这个操作将去掉Join 按钮（也就是不允许网站访问者自己进行注册）。</li><li>在plone根目录中找到portal_workflow </li><li>一般会看到有两个工作流在plone的站点中使用 <code>folder_workflow</code> and <code>plone_workflow</code> </li><li>点击Contents标签</li><li>点击folder_workflow </li><li>点击<code>States</code> 标签</li><li>每个 States 标签中的列表，做以下操作：</li><ul><ul><li>点击States 的名称</li><li>点击Permissions 标签</li><li>改变任何Anonymous 的打勾的选项给Authenticated ，也就是把Anonymous 的选项转移给Authenticated </li><li>保存</li><li>重复以上步骤对每个state</li></ul></ul><li>对 <code>plone_workflow</code> 也做同样的操作</li></ul><p></p><p></p><p>回到portal_workflow 目录，点击Update Security Settings 按钮，确认所有的改变对存在的内容起作用。</p><p>用一个有效的帐户登陆进入网站内容才可见。</p>