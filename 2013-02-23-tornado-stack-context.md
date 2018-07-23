date: 2013-02-23 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: Tornado 的 stack context
date: 2013-02-23 00:39:27 +0800
categories:
- scripts
tags:
- async
- memcached
- python
- scripts
- stack_context
- timeout
- tornado
status: publish
type: post
published: true
---
<blockquote>按：本人 python 菜鸟，对 tornado 更没什么研究，这两天小摆弄了一下，记一下，有不对的还请指正</p></blockquote>

<p>这两天在用 <a title="Tornado" href="http://www.tornadoweb.org/" target="_blank">tornado</a> 做一个 memcached 的 proxy，作为一个 Python 的高性能异步框架，tornado （实际是 epoll/kqueue... ）的思想是——单线程+异步化，线程的运行时间不等待任何东西，这样就要求 memcached 的访问也必须异步化。如果线程在等待中消耗了，就无法达到高并发的目的，这个问题是无法通过简单地交给线程池或什么其他东西来达到的。</p>

<p>于是，这里就不能用常用的 python-memcache 来做了，实际上有几个基于 tornado 的 memcache 客户端，<a href="https://github.com/dpnova/tornado-memcache" target="_blank">这个</a>是维护得相对好的一个，也是一年前的了，而且，有两个问题：</p>

<ul>

<li>连接建立是同步的，不是异步的</li>

<li>没有超时机制</li>

</ul>

<p>这样，在 server 或网络出现问题的时候，就可能遇到大麻烦，所以，我的目的就是绞尽脑汁加入超时机制，这个初步做出来了，等把 get 之外的方法也都异步化之后就反馈出来。这里主要依赖的机制就是 tornado 的 stack context——再次声明，我是这个方面的菜鸟，有什么不对的地方大家嘘之余给指出来呗。</p>

<p>Stack context 的意图就是为执行程序保存一个上下文，在需要的时候，可以回到这个上下文执行，包括异常，都可以更好地、统一地处理。这个功能的代码不是很多，也比较清晰，但是文档……嗯，至少我是没看明白，结合 httpclient 的源码作为例子，加上看 stack_context 的代码，大概明白了是怎么用了。</p>

<p>首先，在希望抓住问题的入口的地方要留住上下文：</p>

<pre class="brush: python; gutter: true">        #......
        context = partial(self._cleanup, fail_callback = fail_callback)
        with stack_context.StackContext(context):
            getattr(c, cmd)(*args, **kwargs)</pre>

<p>这里，后面的执行内容，包括回调、触发事件，都可以通过抛出异常退到这里，而管理异常的就是 context，这里，用 functools.partial 包装了一下 _cleanup，_cleanup 的写法大致是这样的:</p>

<pre class="brush: python; gutter: true">    @contextlib.contextmanager
    def _cleanup(self, fail_callback = None):
        try:
            yield
        except _Error as e:
            print &quot;gotcha&quot;, e
            if fail_callback:
                fail_callback(e.args)</pre>

<p>这里，异常会被捕获，并调用用户指定的出错回调函数进行处理。后面的代码里，遇到故障，抛出异常就可以了，比如，可以用这个异常来返回超时：</p>

<pre class="brush: python; gutter: true">    def _on_timeout(self, server):
        self._timeout = None
        server.mark_dead(&#039;Time out&#039;)
        raise _Error(&#039;memcache call timeout&#039;)</pre>

<p>这个异常是通过 io_loop 的 timeout 方法来触发的:</p>

<pre class="brush: python; gutter: true">            self._timeout = self.io_loop.add_timeout(
                    time.time() + self.request_timeout,
                    stack_context.wrap(partial(self._on_timeout, server)))</pre>

<p>这样，就可以在异步程序里比较干净地处理掉超时问题了。</p>

<p>这个代码对我这个水平的初学者还是比较晦涩的，大家可以参考下 HTTPClient 的源码，等我把这个 memcached client 的代码改完之后，也会放出来供参考指正的。</p>

<p>----</p>

<p>update2: 放这里了<a href=" https://github.com/gnawux/tornado-memcache"> https://github.com/gnawux/tornado-memcache</a> , get 测试过，其他还没有，另外，我不是多个 server sharding 的应用场景，相关的还没测试。</p>

<p>update :对于 timeout，设上了表忘了清除，如果是其他方式抛异常退出的话，也在抛异常的地方或者是最后处理异常的时候，把超时去掉</p>

<pre class="brush: python; gutter: true">        if self._timeout is not None:
            self.io_loop.remove_timeout(self._timeout)
            self._timeout = None</pre>

<p>&nbsp;</p>