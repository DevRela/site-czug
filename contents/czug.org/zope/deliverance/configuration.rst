---
created: ''
creator: ''
description: Deliverance 的所有配置都存储在一个 xml 文件中，在这个文件中可以配置页面匹配(page matches)，置换规则(transformation
  rules)，代理(proxying)和服务器(server)配置。
title: Deliverance 的配置
---
Deliverance 配置 Deliverance Configuration
=====================================================

.. contents::

Deliverance的所有配置都集中在一个xml文件里，这个文件可以配置：页面匹配、转换规则、代理、还有服务器配置。文件分为若干个主要部分，概览如下：

.. sectnum::

All the configuration for Deliverance goes in an XML file.  This file can configure page matches, transformation rules, proxying, and server settings.  Though it is all in one file, there are several major sections.  A quick overview:

`规则与主题 rule and theme`_:
    定义了主题并把具体的转换应用到页面。
    Determine the theme and apply the actual transformations to the page.
`匹配、页面类 match and page classes`_:
    在更多的复杂的网站里，在页面类中，你可以根据不同的标准来应用规则。
    In more complicated sites, page classes allow you to apply rules based on different criteria.
`代理和服务器设置 proxy and server-settings`_: 
    控制代理结点、服务器、及安全设置。
    Controls the proxy destinations, server, and security settings.
`请求与响应的匹配 request/response matching`_:
    一些元素只能在请求或响应符合一定条件下才能使用，这些是指它们有公共的行为和属性。
    Several elements can be conditionally used only when the request or response matches conditions.  All these elements share common behavior and attributes.
`参考 pyref Python references`_:
    一些元素能够唤起python钩子，把钩子细节显示在元素旁边，用来描述一般的语法。
    Several elements can call Python hooks.  This describes the general syntax, while the details of the hook are described alongside the element.
`开发人员调错平台 developer debugging console`_:
    这提供了关于deliverance正在做的事情的信息。
    This gives you information about what Deliverance is doing, on a page-by-page basis.

所有的东西放在<ruleset>标签里。
Everything goes in a ``<ruleset>`` tag.

.. comment: FIXME: should this be <deliverance>?

规则和主题 rule and theme
-----------------------------------

这是deliverance所做的事情的核心：主题与内容并行，并应用各种转换。
This is the core of what Deliverance does: take a theme along with your content, and apply transformations.

主题 theme
~~~~~~~~~~~~~~~~~~~

The ``<theme>`` element defines the theme you'll be using.  The theme is given as a URL.  The basic form is:

.. code-block:: xml

    <theme href="/theme.html" />

This defines the theme as being at ``/theme.html``.  If possible it will be fetched with an internal request, though external requests are also possible (if you host the theme outside of Deliverance).

If you have ``<theme>`` at the global level (in the ``<ruleset>`` element) then that is the default theme.  You can also put it inside ``<rule>`` elements, and then it will apply just to that rule set.  This is useful if you are using `match and page classes`_.

The theme element also supports `pyref Python references`_.  If you use:

.. code-block:: xml

    <theme pyref="mymodule:get_theme" />

Then define a function like:

.. code-block:: python

    def get_theme(request, response, log):
        return "/%s/theme.html" % request.host

The ``request`` and ``response`` objects are `WebOb <http://pythonpaste.org/webob/>`_ objects.

The default function name is ``get_theme``.

规则 rule
~~~~~~~~~~~~~~~

<rule>元素定义了一组转换动作，它也支持页面类和请求、响应匹配，在那些分类中可以看到。同时，你能够内嵌一个<theme>元素。但最重要的是那些转换动作。
The ``<rule>`` element defines a set of transformations.  It also supports `page classes`_ and `request/response matching`_, which you can read about in those sections.  Also, as mentioned above, you can include a ``<theme>`` element.  But the most important thing is the transformation actions.

转换动作是按顺序执行的，它的起点是theme文件，这些动作把内容里面的元素复制到theme主题里面来。
The transformation actions are applied in order.  The starting point for the transformation is the *theme* document, and the actions copy elements from the content into the theme.

There are four actions:

``<replace>``:
    用内容里面中的element元素替代theme主题中的一些东西。
    Replaces something in the theme with elements from the content.
``<append>``:
    把指定内容附加到theme主题中的一个element元素中尾部。
    Appends content to an element in the theme.
``<prepend>``:
    把指定内容附加到theme主题中的一个element元素中首部。
    Prepends content to an element in the theme.
``<drop>``:
    删除theme主题或内容中的一些元素。
    Removes elements from the theme or the content.

Selectiong选择器及其类型 Selection and selection types
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Each rule depends on selecting elements from the theme and content.  The most basic selection is done with CSS 3 selectors.  For instance, this places the element in the content with the id ``content`` *after* the element in the theme with the id ``header``:

.. code-block:: xml

    <append content="#content" theme="#header" />

You can also use `XPath <http://www.w3.org/TR/xpath>`_ selectors.  Any selector starting with ``/`` is treated as an XPath expression, while everything else is treated as CSS.  CSS can only select elements, and while XPath can select text or attributes Deliverance is only interested in elements.  Moving elements around has some limitations, so there are different explicit types of selection:

.. comment: FIXME: a better XPath link would be nice, like to a tutorial.

``elements:``
    The default, this applies the rules to the elements selected.
``children:``
    A common type, this applies rules to the *children* of the elements selected (including text content of the elements).
``attributes:``
    This applies the rules to just the attributes.  Also you can apply it to just specifically named attributes, for instance just to the ``class`` and ``style`` attributes with ``attributes(class,style):``.
``tag:``
    This applies the rule to the tag, but not the children of the element.  For instance, dropping a tag keeps the children in the document, but removes their enclosing tag.

You can apply any of these like ``content="children:#content"``.  Not all combinations make sense, and some are not allowed.  For instance, ``<replace content="attributes:#content" theme="elements:#header" />`` does not make sense, as you can't replace elements with attributes.  Generally ``elements:`` and ``children:`` work together, ``attributes:`` only works with ``attributes:``, and ``tag:`` only works with ``tag:``.

When selecting elements you can use the ``||`` operator.  This applies to both CSS and XPath selectors, and with the operator you can mix the two.  The ``||`` operator takes the results of the first selector that matches anything.  So ``content="#content || children:body"`` will take the element ``#content`` if there is one, and if there is not one it will take all the children of ``<body>``.  You can mix ``elements:`` and ``children:`` using ``||``, though no other types can be mixed like this.

``<replace>``
+++++++++++++

The ``<replace>`` action replaces something in the theme with something in the content.  Exactly what is replaced depends on the selection type.  Some examples:

.. code-block:: xml

    <replace content="children:#content-wrapper" theme="children:#content" />

this replaces the elements *inside* the theme element ``#content`` with the elements inside the content element ``#content-wrapper``.  The resulting document won't have any element with the id ``#content-wrapper`` (unless the theme already had an element with that id).

.. code-block:: xml

    <replace content="elements:#content-wrapper" theme="elements:#content" />
    <replace content="#content-wrapper" theme="#content" />

both of these are the same (``elements:`` is the default selection type).  This replaces the theme element ``#content`` with the content element ``#content-wrapper``.  The resulting document has no element ``#content``.

.. code-block:: xml

    <replace content="attributes:body" theme="attributes:body" />

this removes all the attributes (e.g., ``class``, ``onload``) from the ``<body>`` element in the theme, and moves over the attributes from the content body element.

.. code-block:: xml

    <replace content="tag:#content" theme="tag:#content" />

this replaces the tag ``#content`` in the theme with its corresponding tag from the theme.  They might not be the same tag name (e.g., the theme might be a ``<p>`` and the content ``<div>``), and all attributes will be taken from the content.

``<append>`` and ``<prepend>``
++++++++++++++++++++++++++++++

These actions obvious are very similar; ``<append>`` puts things from te content after things in the theme, and ``<prepend>`` puts things from the content before things in the theme.  Some examples:

.. code-block:: xml

    <append content="children:#sidenav" theme="children:#sidebar" />

this moves the children of the content element ``#sidenav`` to the end of the theme element ``#sidebar``, combining the navigation of the theme and content.  If you wanted the content navigation to go first you'd use:

.. code-block:: xml

    <prepend content="children:#sidenav" theme="children:#sidebar" />

另一个例子 Another example:

.. code-block:: xml

    <append content="children:#sidenav" theme="ol.menulinks" />

this moves the children of ``#sidenav`` *after the element* with the element in the theme ``<ol class="menulinks">``.

.. code-block:: xml

    <append content="li.reference" theme="children:ol.menulinks" />

this moves any element in the content like ``<li class="reference">`` *into* the element in the theme ``<ol class="menulinks">``.

You can also use these with attributes:

.. code-block:: xml

    <append content="attributes:div#content" theme="attributes:div#content-wrapper" />

This adds any attributes from ``div#content`` into the theme element ``div#content-wrapper`` -- but when the attribute already exists in the theme, the theme attribute is kept.  If you use ``<prepend>`` then when there are overlapping attributes the content attribute value is kept.

.. comment: FIXME: should attributes(class) know about how classes can be combined?

You can't use ``tag:`` with these actions.

``<drop>``
++++++++++

The ``<drop>`` action is used to remove problematic elements from a theme or content.  You can also use it with `if-content`_ for other kinds of conditions.  Since this action doesn't move elements around you don't need to provide both ``content`` and ``theme`` attributes; either will do.

A common example is removing a stylesheet which introduces conflicts:

.. code-block:: xml

    <drop content="link[href $= '/sitestyle.css']" />

This is one of the more advanced selectors that CSS 3 allows.  You can't (confidently) use it in browsers yet, but you can in Deliverance!  The ``$=`` operator means *ends-with*, so this selector could be described as: *all link elements with href attributes that end with '/sitestyle.css'*.  Other operators are the obvious ``=``, ``^=`` which means *starts-with*, ``*=`` which means *contains*.  Note that all comparisons are case-sensitive.

.. comment: FIXME: should those realy be case-sensitive?

另一个例子 Another example:

.. code-block:: xml

    <drop content="attributes(class):a.external-link" />

which removes the class from any ``<a class="external-link">`` elements.

.. code-block:: xml

    <drop content="tag:font" />

which removes all ``<font>`` tags, but doesn't remove any actual text content.

``if-content``
++++++++++++++

All the actions can take an attribute ``if-content="selector"``.  This selector is attempted on the content, and if it doesn't match anything then the action is skipped.

零匹配、多重匹配、错误处理 Zero or multiple matches and error handling
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Except for ``<drop>``, in actions it is necessary that the ``theme`` selector match exactly one element.  If it doesn't match *any* element, then the action can't be performed.  If it matches many elements then the action is ambiguous: which element should be the target?  Also, when no content element is matched it is a sign something is wrong, and in some cases if multiple content elements are matched it is a problem.

The default handling for all these is to log the problem at the level "warn".  If there are multiple matches when that is not expected, the default handling is to log and use the first element.  There are four attributes to override this:

``notheme``:
    The handling when no theme element is found.
``manytheme``:
    The handling when more than one theme element is found.
``nocontent``:
    The handling when no content elements are found.
``manycontent``:
    The handling when more than on content element is found (and that doesn't make sense in the context of the action).

A value ``notheme="ignore"`` means that the action is ignored and only a "debug" level logging message is produced.  A value like ``nocontent="abort"`` means that if there is no content then all theming will be aborted, and the unthemed content page will be displayed.  For ``manytheme`` and ``manycontent`` you can indicate ``first`` or ``last`` to select the first (default) or last element.  You can combine this like ``manytheme="ignore:first"``.

一个例子 An example:

.. code-block:: xml

    <replace content="children:#content" theme="children:#content"
             nocontent="abort" />

Almost all rules should have at least one action with ``nocontent="abort"``.  This is the action that moves the primary body of the content into the theme.  If that primary body can't be found then when people browse the page they'll see the theme and lose all the useful content.

``manycontent`` is used when you are using selection types ``attributes:`` or ``tag:`` -- in both these cases the theme and content element match up one-to-one.

When using ``<drop>`` multiple theme and content elements are expected, so no error handling is necessary in that case.

用href导入外部内容 Including external content with ``href``
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

In addition to manipulating the theme and content documents, you can bring in content from a third source using the ``href`` attribute on a rule.

If you include ``href`` the document at that location is used instead of normal content document.  For example:

.. code-block:: xml

    <append href="/sidebar" content="children:body" 
            theme="children:#sidebar" />

This appends all the elements in the body of ``/sidebar`` into the theme.  This can be used to make the theme more dynamic.  Deliverance doesn't produce content on its own, or modify content in complex ways, but you can use includes like these to introduce several sources of dynamic content into a single page: ``/sidebar`` can be a dynamically generated page itself.

转移、复制内容 Moving and copying
++++++++++++++++++++++++++++++++++++++++++

By default actions *move* elements from the content to the theme.  That is, if you select content with an action, later actions won't be able to access those content elements.  This is particularly useful in cases like this:

.. code-block:: html

    <body>
      <div class="navigation">links...</div>
      Some content
    </body>

with the rules:

.. code-block:: xml

    <rule>
      <replace content="children:#navigation" theme="children:#sidebar" />
      <replace content="children:body" theme="children:#content"
               nocontent="abort" />
    </rule>

If elements were copied the navigation would show up twice in the resulting page.  If you *do* want content copied instead of moved then add ``move="0"`` to the action.

标准动作 Standard actions and ``suppress-standard``
++++++++++++++++++++++++++++++++++++++++++

There are several actions that are "standard".  These are common for HTML generally.  The actions are:

.. code-block:: xml

  <rule>
    <replace content="children:/html/head/title"
             theme="children:/html/head/title" nocontent="ignore" />
    <append content="elements:/html/head/link"
            theme="children:/html/head" nocontent="ignore" />
    <append content="elements:/html/head/script"
            theme="children:/html/head" nocontent="ignore" />
    <append content="elements:/html/head/style"
            theme="children:/html/head" nocontent="ignore" />
  </rule>

These copy over the title and any links, scripts, or stylesheets from the content into the theme.  These are always applied unless you use ``<rule suppress-standard="1">``.


.. _`page classes`:
.. _`match`:

匹配、页面类 match and page classes
-------------------------------------

Note: for a simple site you can mostly ignore this, and just define a single ``<rule>`` that is applied to all requests.  But in many cases you will need to apply specific rules only to parts of the website.  For example, different applications may have navigation in specific parts of the page, and you want rules to move that navigation around the page that is specific to the application.

Often different sets of rules will apply to different parts of the site.  To define what rules go with what requests/responses there is the concept of "page classes".  These are classes that apply to the page, and any rules with those classes are used.  For example, imagine a page has the classes ``trac`` and ``default``.  Then these rules would be used:

.. code-block:: xml

    <rule class="trac">...</rule>
    <rule class="default">...</rule>

Like CSS any rules that match any of the classes on a page will be run.  Also like CSS you can have multiple classes anywhere, separated by spaces.  Unlike CSS the classes are ordered, and define the order the rules are run in.

.. comment: FIXME: should they be run according to the order of the rules in the ruleset?

There are several ways to define classes on a page.  If you have control of the server you can add the response header ``X-Deliverance-Page-Class`` with the page classes.  You can also add a ``class`` attribute to `proxy`_ elements.

A third way is with ``<match>``.  These elements match requests and add classes.  You can use them like:

.. code-block:: python

    <match domain="lists.*" class="lists" />

This will add the "lists" class to any requests going to a domain ``lists.*``.  The request matching is described in `request/response matching`_.

代理及服务器配置 proxy and server-settings
------------------------------------------------------

Deliverance comes with a proxy, ``deliverance-proxy``.  This starts a server and proxies HTTP requests to other backend servers (Zope, Apache/PHP, paster, etc).

Settings for the proxying also go in the rule file.  The ``<server-settings>`` element defines aspects of the server, while ``<proxy>`` defines specific servers to proxy to.

.. _`server-settings`:

``<server-settings>``
~~~~~~~~~~~~~~~~~~~~~

This element looks like:

.. code-block:: xml

    <server-settings>
      <server>localhost:8080</server>
      <execute-pyref>false</execute-pyref>
      <display-local-files>false</display-local-files>
      <dev-allow>
        127.0.0.1
        192.168.0.1/24
      </dev-allow>
      <dev-deny>
        192.168.0.121
      </dev-deny>
      <dev-htpasswd>/etc/deliverance-dev-users.htpasswd</dev-htpasswd>
      <dev-user username="bob" password="uncle" />
      <dev-expiration>60</dev-expiration>
    </server-settings>

You can't actually use quite all of these together.  Going over the individual settings:

``<server>``:
    This gives the host and port to use.  If you use ``localhost`` or ``127.0.0.1`` for the host then only local connections are allowed.  If you use ``0.0.0.0`` then the server is started up on all your interfaces.  The default is ``localhost:8080``.

``<execute-pyref>``:
    This defaults to true.  If this is true then ``pyref`` attributes are allowed (see `pyref Python References`_).  This gives anyone who can write to your rules the ability to execute arbitrary code for each request, so you should turn it off if untrusted people have that access.

``<display-local-files>``:
    The `developer debugging console`_ will, by default, display any local files.  If the person accessing that console generally has ssh or other access to the server this isn't a problem.  But if not you should turn this off.

``<edit-local-files>``:
    You can edit files through the `developer debugging console`_ unless you include ``<edit-local-files>false</edit-local-files>``.

The ``<dev-*>`` tags define access to the `developer console`_.

``<dev-allow>``:
    This is a list of IP addresses (or IP+mask) that are allowed access.

``<dev-deny>``:
    These addresses are specifically disallowed.

``<dev-htpasswd>``:
    This is a list of usernames and passwords created with the Apache ``htpasswd`` program.  You log in using these.

``<dev-user>``:
    Instead of ``<dev-htpasswd>`` you can put username/passwords directly in.  This isn't a very good idea, but combined with a restrictive ``<dev-allow>`` it's not so bad.  You can't use both this and ``<dev-htpasswd>``.  You must use one of these to get access to the console.

``<dev-expiration>``:
    The time, in minutes, that a session can last.  You have to re-login after this amount of time.  This defaults to 0, meaning no expiration.

``<dev-secret-file>``:
    The location where the server-side secret should be kept.  This file will be created on its own, as well as the directory that contains it, but Deliverance needs permission to write here.

.. comment: FIXME: what's the default IP restriction?
.. comment: FIXME: say something about variable substitution.

.. _proxy:

``<proxy>``
~~~~~~~~~~~

The ``<proxy>`` element is what defines what gets routed where.  The basic pattern is:

.. code-block:: xml

    <proxy path="/trac">
      <dest href="http://localhost:10002" />
    </proxy>

This proxies any requests to ``/trac`` to the dest address.  A request to something like ``/trac/view/1`` will go to ``http://localhost:10002/view/1``.  If you wanted to keep ``/trac`` you can use ``<proxy strip-script-name="0">``.

If you want to pass the ``Host`` header through without changing it, use ``<proxy keep-host="1">``.  This generally represents the request accurately, but has not been the norm for systems like this, and often the original host name is only available in ``X-Forwarded-Host``.  "Real" proxies like Squid usually preserve the Host header.  Apache does not unless you configure it to do so.  So if you have setup your system for Apache proxying you probably weren't preserving the Host header.

You can also match against things besides the path; anything in `request/response matching`_ is available.  But without a simple ``path`` match the path stripping won't work.  (A path match like ``path="regex:.*/manage$"`` wouldn't work, for instance.)

You can also add `page classes`_ for the proxied request, by using ``<proxy class="page-class">``.

You can use ``<proxy editable="1">`` to allow the files references by the proxy to be editable through the `developer debugging console`_.  If you do this you have to have a ``<dest>`` that references a ``file:///...`` URL.

The proxy can contain several elements...

proxy: ``<dest>``
+++++++++++++++++

The ``<dest>`` element defines the destination.  You must have a dest element.

You can give an ``href`` value of both ``http://...`` URLs and ``file:///...`` URLs.  Files are served directly without proxying, though this is seemless to the rest of the process.

The value in ``href`` can be a URI template (though only the simplest form of template).  You can use headers like ``{Host}``, environmental variables like ``{REMOTE_USER}``, or the variable ``{here}`` which points to the directory containing the rule file.  I.e., if the rule file is in ``/etc/deliverance/rules/ruleset.xml`` then ``{here}`` would be ``file:///etc/deliverance/rules``.  This template is substituted for every request, so it can be fully dynamic.

In addition to this you can use `pyref Python references`_, like:

.. code-block:: xml

    <proxy path="/trac">
      <dest pyref="mymodule:get_proxy_dest" />
    </proxy>

with the function:

.. code-block:: python

    def get_proxy_dest(request, log):
        if not request.remote_addr.startswith('192'):
            raise AbortProxy('Bad remote_addr: %r' % request.remote_addr)
        return 'http://localhost:10002'

You can return any URL.  URI template substitution is *not* performed.  If you raise ``AbortProxy`` then the ``<proxy>`` will be skipped, and another proxy will be looked for.  (If nothing matches a 404 error is returned.)

If you want to perform request rewriting (as described in the next section) you can use ``<dest next="1">`` and the request modifications will be performed but otherwise the proxy section will be skipped.

proxy: ``<request>``
++++++++++++++++++++

This modifies the request before it is proxied on.  You can add headers and run Python code against the request.

To add a header:

.. code-block:: xml

    <request header="X-Project-Name" content="My Project" />

You can also use a `pyref Python reference`_, like:

.. code-block:: xml

    <request pyref="mymodule:modify_proxy_request" />

with the code:

.. code-block:: python

    def modify_proxy_request(request, log):
        request.header['X-Project-Name'] = request.host.split('.')[0]
        return request

You can modify the request in-place or return a new request.

proxy: ``<response>``
+++++++++++++++++++++

This modifies the response.  Like the request you can add headers:

.. code-block:: xml

    <response header="Cache-Control" content="max-age=0" />

You can also rewrite links, which means you can proxy from a site that doesn't particularly want to be proxied.  So if you proxy to ``http://othersite.com`` then of course all the links it returns will be to ``http://othersite.com``.  Using link rewriting it will rewrite all those links to go back through the proxy.  This includes the ``Location`` header, any references to ``domain`` in cookies, and all links in the HTML.

.. code-block:: xml

    <response rewrite-links="1" />

And of course there's pyref:

.. code-block:: xml

    <response pyref="mymodule:modify_proxy_response" />

with the code:

.. code-block:: python

    def modify_proxy_response(
        request, response, orig_base, proxied_base, proxied_url, log):
        # orig_base: the original URL base: http://localhost:8080/trac
        # proxied_base: where dest sent it to: http://localhost:10001/
        # proxied_url: the full destination, e.g.,
        #    http://localhost:10001/view/1
        # request.url: the full original URL, e.g.,
        #    http://localhost:8080/trac/view/1
        response.body += 'look at me!'
        return response

请求/响应匹配 request/response matching
---------------------------------------------

Many elements can match requests and responses: the `match`_ element, `rule`_, and `proxy`_.  They all use the same matching.

There are several attributes that match against different parts of the request or response.  If you provide multiple attribute then they must all match.

字符串匹配 String Matching
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These attributes match a string value against a pattern you provide.  There are several kinds of patterns; different attributes default to different types of patterns, whatever is most logical for the attribute value.

First, the attributes:

``path``:
    This matches the request path.  The default pattern type is ``path:``.

``domain``:
    This matches the request host, the domain name of the request URL.  The default pattern type is ``wildcard-insensitive:``.

Each pattern string can start with ``patterntype:`` which defines what kind of pattern it is.  Here's a list of the available patterns (these also apply to the key/value matching described in the next section):

``wildcard:``
    This is a wildcard match, i.e., you can use ``*``.

``wildcard-insensitive:``
    Wildcards that are case-insensitive.

``regex:``
    Matches with a `regular expression <http://python.org/doc/current/lib/re-syntax.html>`_.  Note you can use things like ``(?i)`` to make the expression case-insensitive.

``path:``
    Matches the string as a path prefix.  It's like starts-with, except it is aware of /'s.  So ``path:/some-path`` will match the path ``/some-path/to/a/place``, ``/some-path/`` and ``/some-path``, but it will not match ``/some-path-to-somewhere``.

``exact:``
    This matches the exact string.

``exact-insensitive:``
    The exact string, except case.

``contains:``
    True if the pattern shows up anywhere in the string.

``contains-insensitive:``
    Contains, case-insensitive.

``boolean:``
    This tests if the string is "true".  Values "1", "true", "yes", and "on" are true.  Empty, or any other value, is false.  The pattern is fairly insignificant, except ``boolean:not`` inverts the match.

键/值匹配 Key/Value Matching
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are other matches that match both a key and a value, like matching headers and environmental keys.  These look like ``environ="REMOTE_USER: bob"``, like ``key: pattern``.  The key can be a wildcard, and if any key that matches that wildcard matches the pattern then it is a match.  Only wildcards are allowed for the key.  All patterns default to ``exact:``.

The attributes available:

``request-header``:
    Matches a request header.  The headers are case-insensitive.

``response-header``:
    Matches the response headers.

``environ``:
    Matches keys in the request environment.

Python参考 Python References
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also use a ``pyref``, with a function like:

.. code-block:: python

    def match_request(request, response, response_headers, log):
        if response.headers.get('x-notheme'):
            raise AbortTheme
        return True

Note that ``response_headers`` is a list of all the headers, including ``<meta http-equiv>`` headers.

.. _`pyref Python reference`:

pyref Python参考 pyref Python references
-----------------------------------------------

``pyref`` attribute references allow you to hook into Python code.  To read about the details see the `pyref <pyref.html>`_ document.

.. _`developer console`:

开发人员调试台 Developer Debugging Console
--------------------------------------------------

You must be logged in to view the developer console.  The login page is available at ``/.deliverance/login`` -- you define the users using the `server-settings`_ configuration.

After you log in you can add ``?deliv_log`` to any page URL to get a log at the bottom of the page.  The log describes in detail how the page was transformed, including information about how the proxy was chosen and any subrequests.  It also lets you browse the source involved, see what the selectors select in the content or theme, or get a list of interesting ids and classes in the content.

客户端主题化 Clientside Theming
-----------------------------------------

There is an experimental feature that allows client-side theming using Javascript.  This serves the blank theme to the user, with content pulled in via XMLHttpRequests.  This means the user will get a response quickly, seeing the contentless theme page to start with (and so you should write the theme so that any slots contain text like "Loading...").  The individual chunks that are pulled in will be cachable on the client side.

To enable this put this in your config::

    <clientside />

You may also put conditionals on it, like ``<clientside path="/slow-part" />``.  You can only use one theme in this case, as page classes are not matched before you send the theme.  Also any rules that use ``href="..."`` must be possible to apply out-of-order, as the XMLHttpRequest's may come in unexpected orders.  Using, for instance, ``<replace>`` and ``<append>`` on the same element is problematic in this case -- a better solution would be to clear out the theme target element and use ``<prepend>`` and ``<append>``.

Deliverance will be careful about using this technique.  Only clients that are known to run Javascript will be sent these pages (this is tested by setting a cookie via Javascript, and then testing on subsequent requests if the cookie is set).  Also, pages won't be themed like this until it's known that the backend request yields HTML.  If these aren't true the normal server-side theming is used, which should be equivalent.

This has only been tested on Firefox.
