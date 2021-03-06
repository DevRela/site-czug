---
created: 2007-01-19 12:14:43
creator: zhangbingkai
description: 'Plone中有这样三个概念：'
title: Plone中的系统角色和本地角色
---
<h1>Plone中的系统角色和本地角色</h1>
<p>Plone中有这样三个概念：</p>
<dl class="docutils">
<dt>系统角色</dt>
<dd>系统存在的角色。比如 管理员，审批人等，就是指在Plone控制面板中发配的角色。</dd>
<dt>本地角色</dt>
<dd>当前内容本身所拥有的角色，不是从用户和组管理中分配的角色。比如，某文件/文件夹的创建者，他拥有此文件/文件夹的所有者角色，而这个所有者就是此文件/文件夹的本地角色。</dd>
<dt>继承的角色</dt>
<dd>从上一级目录中所继承过来的角色。比如 user1，它是文件夹A设置的共享管理员，在文件a中用户user1 默认继承了管理员角色</dd>
</dl>
<p>用Plone的朋友应该都清楚，一般设置站点管理员，有两种方式，在网站设置中将某一用户添加到Administrators组，和在根目录中设置共享内容，将某一用户设置为当前共享的管理员权限。其实这两种方式有很大的区别。你现在可能已经知道了，设置到Adminstrators组的用户成为了系统角色，而设置到根目录中的共享管理员权限的用户是站点根目录的本地角色。</p>
<p>那这系统角色和本地角色的两种用户还有什么其它的区别吗？</p>
<p>有的，其实道理很简单，系统角色是控制不了的，他的权力太大了。而本地角色是可以控制的。下面举一个非常明显的例子。</p>
<p>我们知道Plone中可以对内容设置保密，而同时又知道设置保密对管理员是不起作用的，管理员依然可以看到保密的内容。其实也并非这样，你还可以再设置一层。在内容共享标签中可以设置
<em>去掉上级文件夹继承的角色</em> ，那我们就去掉它的上级继承过来的角色。设置后，会看到共享标签页中的内容的当前共享的权限中，从上级继承权限的角色用户/组都变成灰色的了（这里没有贴图，可以自行操作一下）。这一步其实是去掉了本地角色相关的权限。好了，在这里我们就来证明一下系统角色和本地角色的区别。分别用一个系统角色的管理员和一个本地角色的管理员登录站点，看看能不能显示出刚设置保密后同时又设置去掉继承角色的内容
。是的，实事已经证明出来了，本地角色的管理员没有权限访问设置后的内容，而系统角色的管理员是可以访问的，同时对此内容有管理员权限。</p>
<p>好了，上面的例子应该可以足够说明系统角色和本地角色的不同。那么，看到这个例子后，你平时设置站点的管理员是否要考虑一下用哪种方式啰。</p>
