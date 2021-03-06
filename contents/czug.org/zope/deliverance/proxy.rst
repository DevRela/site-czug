---
created: ''
creator: ''
description: 在规则中设置代理
title: 关于代理
---
关于代理About the Proxy
===========================

Proxying can be setup inside the rules as well:

.. code-block:: xml

    <proxy *match-attrs>
      <dest href="dest" | pyref="pyref" *pyargs />
      <transform strip-script-name="1" keep-host="1" />
      <request header="Some-Header" content="some content" />
      <request pyref="pyref" *pyargs />
      <response header="Some-Header" content="some content" />
      <response pyref="pyref" *pyargs />
      <response rewrite-links="1">
    </proxy>

The ``<proxy>`` element takes all the same attributes that ``<match>`` does for matching (not including ``class``, ``abort``, ``last``).

The request is proxied to a location given with the ``<dest>`` element, either a location to proxy to, or a Python callback.  Any Python callback can raise ``AbortProxy``, and the proxy will be skipped (looking for later matching proxies).  It can proxy to http/https and to file locations.  You can use URI templates for destinations as well, substituting headers and environmental variables as well as ``{here}`` which is the directory location of the rule document (e.g., ``<dest href="{href}/theme-files" />``).

The <transform> element controls how the request is transformed when it is forwarded.  By default all the standard headers -- X-Forwarded-For, X-Forwarded-Host, X-Forwarded-Scheme -- are added.  The Host header is not preserved by default, but if you use ``keep-host="1"`` it will be.  As a minor matter, ``environ['SCRIPT_NAME']`` is typically just ignored.  You can have it stripped off, and then X-Forwarded-Path will also be set.  (FIXME: check that header name)

You can modify both the request and the response with multiple ``<request>`` and ``<response>`` tags.  The request can set headers to literal strings, and you can modify the request arbitrarily with ``pyref``.  The response can also have headers added, and arbitrary modification with ``pyref``.  You can also rewrite all links with ``rewrite-links="1"``; this is typically necessary if the X-Forwarded-\* headers aren't used to construct links in the application.  You can also use this to try theming on an existing live site.

FIXME: should there be a way to avoid theming on a section?

FIXME: it would be nice to be able to put in a hard restriction on ``file:`` URLs (both to disallow, or simply to give a base directory that you can't possibly go above).

FIXME: there should be a way of indicating what portion of the path is stripped.  If you do ``<proxy path="/foo">`` then ``/foo`` is stripped (moved to SCRIPT_NAME) before proxying, but for any other kind of matching it doesn't work.  Note you can still do it with ``pyref``.
