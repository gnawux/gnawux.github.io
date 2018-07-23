date: 2011-06-06 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: ZooKeeper Watches
date: 2011-06-06 10:30:12 +0800
categories:
- translation
tags:
- hadoop
- programming
- translation
- zookeeper
- 分布式系统
status: publish
type: post
published: true
---
<p><em>按：王旭（http://wangxu.me/blog, @gnawux）于2011年6月6日译自 ZooKeeper程序员指南 （<a href="http://zookeeper.apache.org/doc/r3.3.3/zookeeperProgrammers.html">http://zookeeper.apache.org/doc/r3.3.3/zookeeperProgrammers.html</a>）的同名章节。似乎很少有文档提这个啊，我其实在看这个之前一直不明白这东东是怎么用的。</em></p>

<p><span style="font-family: Verdana, Helvetica, sans-serif; font-size: 13px; line-height: 15px; background-color: #ffffff;">所有的Zookeeper读操作，包括getData()、getChildren()和exists()，都有一个开关，可以在操作的同时再设置一个watch。在ZooKeeper中，Watch是一个一次性触发器，会在被设置watch的数据发生变化的时候，发送给设置watch的客户端。watch的定义中有三个关键点：</span><span style="font-family: Verdana, Helvetica, sans-serif; font-size: 13px; background-color: #ffffff;">&nbsp;</span></p>

<div class="section">

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;"><strong>一次性触发器</strong></p>

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">一个watch事件将会在数据发生变更时发送给客户端。例如，如果客户端执行操作getData("/znode1", true)，而后&nbsp;/znode1 发生变更或是删除了，客户端都会得到一个&nbsp;&nbsp;/znode1 的watch事件。如果&nbsp;&nbsp;/znode1 再次发生变更，则在客户端没有设置新的watch的情况下，是不会再给这个客户端发送watch事件的。</p>

</li>

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;"><strong>发送给客户端</strong></p>

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">这就是说，一个事件会发送向客户端，但可能在在操作成功的返回值到达发起变动的客户端之前，这个事件还没有送达watch的客户端。Watch是异步发送的。但ZooKeeper保证了一个顺序：一个客户端在收到watch事件之前，一定不会看到它设置过watch的值的变动。网络时延和其他因素可能会导致不同的客户端看到watch和更新返回值的时间不同。但关键点是，每个客户端所看到的每件事都是有顺序的。</p>

</li>

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;"><strong>被设置了watch的数据</strong></p>

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">这是指节点发生变动的不同方式。你可以认为ZooKeeper维护了两个watch列表：data watch和child watch。getData()和exists()设置data watch，而getChildren()设置child watch。或者，可以认为watch是根据返回值设置的。getData()和exists()返回节点本身的信息，而getChildren()返回子节点的列表。因此，setData()会触发znode上设置的data watch（如果set成功的话）。一个成功的&nbsp;create() 操作会触发被创建的znode上的数据watch，以及其父节点上的child watch。而一个成功的&nbsp;delete()操作将会同时触发一个znode的data watch和child watch（因为这样就没有子节点了），同时也会触发其父节点的child watch。</p>

</li>

</ul>

</div>

<p><span style="font-family: Verdana, Helvetica, sans-serif; font-size: 13px; line-height: 15px; background-color: #ffffff;">Watch由client连接上的ZooKeeper服务器在本地维护。这样可以减小设置、维护和分发watch的开销。当一个客户端连接到一个新的服务器上时，watch将会被以任意会话事件触发。当与一个服务器失去连接的时候，是无法接收到watch的。而当client重新连接时，如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。只有在一个特殊情况下，watch可能会丢失：对于一个未创建的znode的exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个watch事件可能会被丢失。</span><span style="font-family: Verdana, Helvetica, sans-serif; font-size: 13px; background-color: #ffffff;">&nbsp;</span></p>

<div class="section"><a name="N10239"></a><a name="sc_WatchGuarantees"></a><br />

<h3 class="h4" style="font-family: 'Trebuchet MS', verdana, arial, helvetica, sans-serif; font-weight: bold; margin-top: 18px; margin-bottom: 0px; font-size: 17px; margin-right: 0px; margin-left: 0px; padding: 0px;">ZooKeeper对Watch提供了什么保障</h3>

<p style="line-height: 15px; text-align: left; margin-top: 0.5em; margin-bottom: 1em;">对于watch，ZooKeeper提供了这些保障：</p>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">Watch与其他事件、其他watch以及异步回复都是有序的。&nbsp;ZooKeeper客户端库保证所有事件都会按顺序分发。</p>

</li>

</ul>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">客户端会保障它在看到相应的znode的新数据之前接收到watch事件。</p>

</li>

</ul>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">从ZooKeeper接收到的watch事件顺序一定和ZooKeeper服务所看到的事件顺序是一致的。</p>

</li>

</ul>

<p><a name="N1025E"></a><a name="sc_WatchRememberThese"></a><br />

<h3 class="h4" style="font-family: 'Trebuchet MS', verdana, arial, helvetica, sans-serif; font-weight: bold; margin-top: 18px; margin-bottom: 0px; font-size: 17px; margin-right: 0px; margin-left: 0px; padding: 0px;">关于Watch的一些值得注意的事情</h3>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">Watch是一次性触发器，如果你得到了一个watch事件，而你希望在以后发生变更时继续得到通知，你应该再设置一个watch。</p>

</li>

</ul>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">因为watch是一次性触发器，而获得事件再发送一个新的设置watch的请求这一过程会有延时，所以你无法确保你看到了所有发生在ZooKeeper上的一个节点上的事件。所以请处理好在这个时间窗口中可能会发生多次znode变更的这种情况。（你可以不处理，但至少请认识到这一点）。</p>

</li>

</ul>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">一个watch对象或一个函数/上下文对，为一个事件只会被通知一次。比如，如果同一个watch对象在同一个文件上分别通过exists和getData注册了两次，而这个文件之后被删除了，这时这个watch对象将只会收到一次该文件的deletion通知。</p>

</li>

</ul>

<ul style="padding-top: 0px; padding-right: 25px; padding-bottom: 0px; padding-left: 25px; margin: 0px;">

<li style="margin-top: 0.5em; margin-bottom: 0.5em; padding-top: 0px; padding-right: 5px; padding-bottom: 0px; padding-left: 5px;">

<p style="line-height: 15px; text-align: left; padding: 0px; margin: 0px;">当你从一个服务器上断开时（比如服务器出故障了），在再次连接上之前，你将无法获得任何watch。请使用这些会话事件来进入安全模式：在disconnected状态下你将不会收到事件，所以你的程序在此期间应该谨慎行事。</p>

</li>

</ul>

</div>

<p>&nbsp;</p>

<p>&nbsp;</p>