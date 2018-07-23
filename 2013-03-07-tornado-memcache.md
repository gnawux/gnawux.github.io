date: 2013-03-07 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: 再次改动了 tornado-memcache
date: 2013-03-07 09:09:07 +0800
categories:
- scripts
tags:
- memcached
- python
- scripts
- tornado
status: publish
type: post
published: true
---
<p>在上次(<a href="http://wangxu.me/blog/p/758">http://wangxu.me/blog/p/758</a>)之后，再次改动了 tornado-memcache 模块，<a href="https://github.com/gnawux/tornado-memcache/commit/10c5236cb7df764e20986db5cca6868c26e845a7" target="_blank">commit 信息</a>如下：</p>

<blockquote><p>Add gets method and simplify connection estabilish</p>

<div>

<pre>- simplify connection estabilish procedure, as tornado.iostream
  permit write before connection estabilished, I removed the
  callback and connection timeout procedure in _get_server()
- add `gets(self, keys, callback, failcallback)` method, receive
  a list of keys as parameter and return a dictionary of results
  result = {key1:value1, key2:value2...}, only return the got
  keys from memcached
- some debug info, might be cleanup later

Signed-off-by: Wang Xu &lt;gnawux……</pre>

</div>

</blockquote>

<p>简单地说就是两件事情：</p>

<ul>

<li><span style="line-height: 13px;">读了<a href="http://www.tornadoweb.org/documentation/iostream.html#tornado.iostream.IOStream.connect" target="_blank">文档</a>，发现 tornado 的 iostream 允许在连接建立未完成的时候就write，既然我之前已经用 iostream 管理 socket 了，这里就省掉上次在这里加上的callback 和 timeout，直接 write command，等 write 的超时或返回就好了；</span></li>

<li>读了<a href="https://github.com/memcached/memcached/blob/master/doc/protocol.txt" target="_blank">memcached 协议</a>，发现取信息(get)，是支持一次取多条信息的，于是加入了这个支持，这样，批处理操作时会提高不少效率。</li>

</ul>

<p>里面还插入了好多调试的时候的 debug 信息，差不多都注掉了，回头都稳定了再清理一下。</p>

<p>嗯，代码都在 github 了：<a href="https://github.com/gnawux/tornado-memcache" target="_blank">https://github.com/gnawux/tornado-memcache</a></p>