---
created: ''
creator: ''
description: ''
title: Deliverance 代码布局
---
Deliverance代码布局 Map of the Deliverance Code
===================================================

This document will help you navigate your way around the Deliverance code base.  You don't need to know this level of detail if you are just *using* Deliverance -- you'll only need this information if you want to contribute to Deliverance development, or integrate Deliverance in another product.

布局代码 The Code
---------------------------

Here's the code:

.. toctree::

   modules/exceptions
   modules/log
   modules/middleware
   modules/pagematch
   modules/proxycommand
   modules/proxy
   modules/pyref
   modules/ruleset
   modules/rules
   modules/security
   modules/selector
   modules/stringmatch
   modules/themeref
   modules/util

deliverance代理启动路径 Startup Path with deliverance-proxy
--------------------------------------------------------------

This describes what happens when you run ``deliverance-proxy``.

1. ``deliverance-proxy`` calls :func:`deliverance.proxycommand.main()`

2. ``main()`` parses the options and does some minor validation, then calls :func:`deliverance.proxycommand.run_command()`

3. The configuration file is parsed for the server and proxy settings by :class:`deliverance.proxy.ProxySettings`

4. The rule configuration is loaded by a wrapper class :class:`deliverance.proxycommand.ReloadingApp`.  This is a class that wrap :class:`deliverance.proxy.ProxySet` and applies the middleware from `ProxySettings`.  It checks the modification time of the configuration file on each request in case you edit the file.

5. :class:`deliverance.proxy.ProxySet` represents all the ``<proxy>`` elements, which define how requests are mapped to remote hosts.

6. :class:`deliverance.proxy.ProxySettings` represents ``<server-settings>``, which defines things like the port to serve on, and developer console login methods.

7. The actual WSGI application is  :meth:`deliverance.proxy.ProxySet.application` and is wrapped by the authentication built by :meth:`deliverance.proxy.ProxySettings.middleware`.  The authentication is implemented with :class:`devauth.DevAuth`.

8. `run_command` applies an interactive debugger if you gave the option ``--interactive-debugger``.  This is to debug problems with Deliverance itself.  Alternately if you just gave ``--debug`` it applies a non-interactive debugger.  (The interactive debugger allows arbitrary code execution.)

9. `run_command` also applies :class:`wsgifilter.proxyapp.DebugHeaders` if you give ``--debug-headers``.  This prints out incoming and outgoing headers, and if you provide ``--debug-headers --debug-headers`` it will show request and response bodies as well.

10. Finally the server is started with the constructed application.  It uses :mod:`paste.httpserver`, a threaded server.

请求路径 Request Path
-------------------------------

This describes what happens when a request comes in.

1. We'll ignoring some of the debugging middleware applied by `run_command`, though it does get entered first.

2. The request goes to :class:`devauth.DevAuth`, which checks IP addresses and cookies to see if you've logged in via ``/.deliverance/login``.  If you are logged in it sets ``environ['x-wsgiorg.developer_user']``

3. The request then goes through :meth:`deliverance.security.SecurityContext.middleware` which instantiates an instance of :class:`deliverance.security.SecurityContext` and puts it in ``environ['deliverance.security_context']``.  This security context is used later to allow or disallow ``pyref``, viewing files, and whether ``?deliv_log`` will show the developer console.

4. Now we enter :meth:`deliverance.proxy.ProxySet.application`.  This sets up :class:`deliverance.log.SavingLogger` which accepts log messages so we can display them later in the request.  Log messages are all per-request.  We then pass the request on to an instance of :class:`deliverance.middleware.DeliveranceMiddleware` -- it is instantiated with a callback into :meth:`deliverance.proxy.ProxySet.proxy_app` and a callback to get the rules.

5. Deliverance really starts doing something in :meth:`deliverance.middleware.DeliveranceMiddleware.__call__`.  Here it checks if the request should be unthemed (because of ``?deliv_notheme``).  It also dispatches ``/.deliverance`` to :meth:`deliverance.middleware.DeliveranceMiddleware.internal_app` (which itself is fairly simple, and just implements some stuff like viewing files, logging in, etc).

6. The request is passed on to ``self.app``, which :meth:`deliverance.proxy.ProxySet.proxy_app`.  This goes through all the instances of :class:`deliverance.proxy.Proxy` (each of which represent one ``<proxy>``) and tries to match them against the request.  If none matches it returns a 404.

7. Assuming something matches, the request goes to :meth:`deliverance.proxy.Proxy.forward_request`.  This fixes up the path based on the rules (possibly stripping some of the leading text).  It applies any ``<request>`` modifications, forwards the request, and then applies any ``<response>`` modifications, including link rewriting.

8. The request is forwarded to :meth:`deliverance.proxy.Proxy.proxy_to_dest`.  If it sees the destination is a ``file:///`` URL then it passes the request to :meth:`deliverance.proxy.Proxy.proxy_to_file` (which itself is not very interesting).  The method sets up some headers: ``X-Forwarded-For``, ``X-Forwarded-Scheme``, ``X-Forwarded-Server``, and ``X-Forwarded-Path``.  It then actually forwards the request via :func:`wsgiproxy.exactproxy.proxy_exact_request`.

9. The response is now complete, and we are back in :meth:`deliverance.middleware.DeliveranceMiddleware.__call__`.  If the response wasn't of Content-Type text/html, then the request is passed back without any modification.

10. The rule set is retrieved from `Proxy` -- an instance of :class:`deliverance.ruleset.RuleSet` (that was parsed directly from the configuration file).  The response is mostly modified in-place by :meth:`deliverance.ruleset.RuleSet.apply_rules`.  Finally :meth:`deliverance.log.SavingLogger.finish_request` is called to display the developer console if appropriate.

11. :meth:`deliverance.ruleset.RuleSet.apply_rules` determines the response headers, which also includes headers declared in the body with ``<meta http-equiv>``.

12. The classes are determined by calling :func:`deliverance.pagematch.run_matches`, which calls all the ``<match>`` elements that can add classes to the request.  Also classes from the response header ``X-Deliverance-Page-Class`` are added, and any classes in the request in ``environ['deliverance.page_classes']`` (these last classes are set when you use ``<proxy class="...">``, since ``<proxy>`` was handled earlier).  If no classes are declared, then the single class ``default`` is used.

13. The applicable rules are determined -- that is, all the classes from the previous step are used to select the rules.  Also, a theme is determined from the ``<theme>`` element in a rule, or defaulting to the ``<theme>`` element inside ``<ruleset>``.  The theme is also resolved as it can be a URI Template.

14. The document and theme are parsed.  All of the rules are run against the document and theme in order.  Because rules can contain match attributes, some rules may be skipped.

15. If none of the rules has ``suppress-standard="1"`` then the "standard" rules are also applied.  These are located in :data:`deliverance.ruleset.standard_rule`.

16. Any of the rules can raise :exc:`deliverance.exceptions.AbortTheme`, which will cause the entire request to be unthemed.

17. The request is serialized and returned.  But we didn't describe how the rules work internally yet...

18. Each ``<rule>`` tag is an instance of :class:`deliverance.rules.Rule`.  This is just a container for actions, which are applied in-order.

18. Each action tag is a class: ``<replace>`` is :class:`deliverance.rules.Replace`, ``<append>`` is :class:`deliverance.rules.Append`, ``<prepend>`` is :class:`deliverance.rules.Prepend``, and ``<drop>`` is :class:`deliverance.rules.Drop`.

19. The main method is :meth:`deliverance.rules.TransformAction.apply` (or for ``<drop>`` it is :meth:`deliverance.rules.Drop.apply` -- ``<drop>`` is a little different from the other actions).  This method does some selection tasks that are common across the selections, as well as some error checking.  It then calls the method ``self.apply_transformation`` which is implemented per-action.  The implementation performs the actions (which isn't that hard) and tries to make good context-sensitive log messages.

That summarizes pretty much everything involved.  Hopefully this helps you poke into the code more easily.  There's still lots of ancilliary methods and modules, but this is the core of the request path.

解析配置文件 Parsing the Configuration
--------------------------------------------

The Deliverance configuration is XML, and in most cases an element in the configuration maps to a class, and that class has a method ``.parse_xml(element, source_location)``

Here's a run through the elements and their classes:

``<server-settings>``:
    :class:`deliverance.proxy.ProxySettings` -- all the sub-elements are packaged into this one instance.

``<proxy>``:
    :class:`deliverance.proxy.Proxy` (all the `Proxy` instances are packaged in one :class:`deliverance.proxy.ProxySet`).  ``<transform>`` element information is stored in `Proxy`.

``<proxy><dest>``:
    :class:`deliverance.proxy.ProxyDest`

``<proxy><request>``:
    :class:`deliverance.proxy.ProxyRequestModification`

``<proxy><response>``:
    :class:`deliverance.proxy.ProxyResponseModification`

``<ruleset>``:
    :class:`deliverance.ruleset.RuleSet` -- except ``<server-settings>`` and ``<proxy>`` elements, which are contained separately (as noted above).

``<match>``:
    :class:`deliverance.pagematch.Match`

``<theme>`` (and ``<rule><theme>``):
    :class:`deliverance.themeref.Theme`

``<rule>``:
    :class:`deliverance.rules.Rule`

``<rule><replace>``:
    :class:`deliverance.rules.Replace`

``<rule><append>``:
    :class:`deliverance.rules.Append`

``<rule><prepend>``:
    :class:`deliverance.rules.Prepend`

``<rule><drop>``:
    :class:`deliverance.rules.Drop`

Also many classes support request and response matching.  These are defined with a common set of attributes on an arbitrary element.  The parsing for these is done with :class:`deliverance.pagematch.AbstractMatch`.

