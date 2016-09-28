date: 2009-03-22 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: '[译文] Xen: 完成工作'
date: 2009-03-22 20:49:00 +0800
categories:
- translation
tags:
- kernel
- linux
- lwn
- virtualization
- xen
status: publish
type: post
published: true
---
<p><a href="http://lwn.net/Articles/321696/">http://lwn.net/Articles/321696/</a><br />
By <strong>Jonathan Corbet</strong><br />
March 4, 2009<br />
王旭 2009年3月22日译</p>

<p>曾几何时，Xen是炙手可热的虚拟化技术。Xen的开发者们拥有一个自由软件中最为领先的虚拟化解决方案，Xen看起来正是Linux虚拟化的未来。很多风投在追逐这个神话，各个发行版也竞相提供基于Xen的虚拟化平台。然而，在这条路上，Xen似乎已经走失了。XenSource的开发者们看起来似乎没有兴趣让他们的代码进入主线内核，而其他人的努力也遇到了没完没了的障碍。于是，Xen在多年以来就一直停留在主线之外了。Xen首次对外发布是2003年的事了，可Xen的核心代码仅仅在2007年10月才进入了2.6.23。</p>

<p>就在同时，KVM出现了，而且一下子抢走了众多的注意力。它以难以置信的速度一下子就进入了主线内核，而且很多内核开发者都毫不掩饰的表达了他们对于KVM锁采用的方法的青睐。就在最近，Red Hat<a href="http://www.redhat.com/about/news/prarchive/2009/agenda.html">发布</a>了他们的KVM虚拟化时间表，让这种偏爱更加官方了。同时，<a href="http://lwn.net/Articles/218766/">lguest</a> 成为了那些想要尝试虚拟化的人的简单选择。</p>

<p>Xen的故事是“上游优先”策略产生的原因的一个经典实例，该策略要求代码在被提供给客户之前应该首先进入主线。发布者们忙不迭地首先发布Xen，然后就发现他们自己正在支持一段out-of-tree的代码，这些代码常常根本不被他们的开发者所支持。特别地，对外发布的Xen通常只支持相当老的内核，希望提供一些更新的内容的发布者需要做很多工作来让它们一起工作。现在，至少部分的发布者（发行版）已经开始走向其他的选择了，而高层的内核开发者则在问一个问题——事到如今，是否还值得来把剩下的Xen代码merge进来。</p>

<p>所有人都在说，Xen已经岌岌可危了。不过，或许更严重的所谓Xen已经死掉了的流言有些言过其实了。</p>

<p>主线中的代码实现了 Xen 的 DomU 概念——一没有访问硬件的权限的非特权域。而一个完整的Xen则需要更多功能；需要一个用户空间（原文如此）的hypervisor（是以GPL许可发布的）和内核中的Dom0代码。Dom0是hypervisor启动的第一个域；定性情况下，它比其他Xen guest拥有更多的特权。Dom0用于在管理策略的管理下小心的向其他域提供那些诸如访问硬件、网络等的特权。一个实用的Xen系统必须包含Dom0代码——目前是一大块out-of-tree的kernel代码。</p>

<p>Jeremy Fitzhardinge 希望改变这一局面。他发出了<a href="http://lwn.net/Articles/321298/">一组Xen Dom0 patch</a> ，希望能够进入 2.6.30。在审阅者中，Andrew Morton问了<a href="http://lwn.net/Articles/321699/">这个问题</a> ：</p>

<blockquote><p>我讨厌成为说这句话的人，不过，我们确实应该坐下来弄明白把它merge到Linux当中来是否真的合适。我觉得Xen是实现虚拟化功能的老方法了，世界已经专项新方向KVM了。</p></blockquote>

<p>三年之内，我们是不是会后悔我们merge了Xen？</p>

<p>Andrew的问题本质上是：（1）完成这个工作需要什么代码（这次提交的代码之外），和（2）这么做的原因是什么?第一个问题的<a href="http://lwn.net/Articles/321701/">答案</a> 是“再有2-3个尺寸差不多的补丁集就将提供引导dom0所需的所有代码”。之后有一些其他的不同内容可能不会进入主线。不过，Jeremy说，让核心代码进入主线将会减少发行版需要打的out-of-tree的补丁，让所有人的生活变轻松一些。对于第二个问题，Jeremy回应到：</p>

<blockquote><p>尽管kernel周期中有来自kvm的影响，Xen还是拥有庞大并且仍在增长的用户群的。此时，他们都工作在大量的out-of-tree补丁之上，这对大家来说都十分不快。让他们成为主线内核的一部分再好不过了。你知道，这和我们争论的所有其他事情都一样。</p></blockquote>

<p>此外，Jeremy表示，Xen仍然有其存在的价值。它的设计在很多方面和KVM有本质的不同；<a href="http://lwn.net/Articles/321702/">这封邮件</a> 精辟的解释了这些差异。所以，Xen作为另一个解决方案已然有意义。</p>

<p>Jeremy指出的一些Xen的优势包括：</p>

<ul>

<li>Xen的实现页表的方法消除了shadow页表或guest中嵌入页表的需要；这样，可以让许多负载获得更好的性能。</li>

<li>Xen的hypervisor是非常轻的，并且可以独立运行；而KVM的hypervisor则就是Linux内核本身。似乎有些厂商（HP 和Dell被点到名了）在他们的很多系统的firmware中会提供一个Xen的hypervisor。这些代码于是和可以和其他的东西一样具有可以立即使用的特性了。</li>

<li>Xen的半虚拟化方式可以使用不支持虚拟化的硬件一起工作。而KVM则需要硬件支持。</li>

<li>分离hypervisor, Dom0和 DomeU的方式让安全性检查更简单了。各个域之间的隔离还允许让每个设备都在一个独立的域中提供服务，这可以看做是一种重量级的微内核架构。</li>

</ul>

<p>与此不同，KVM相对更简单、更易用，并且可以访问所有的内核特征。在Jeremy看来，两个系统在Linux中各有不同的位置。</p>

<p>这个讨论在最后重归沉默，这表明Jeremy的解释相当不错。Xen历史上犯过错误，但这个项目仍然活着，而且仍然有它的原因继续存在下去。你的编辑（译 著：编辑就是本文原文作者）预测，Dom0的代码在尝试进入2.6.30的merge window的时候还会遇到一定的反对意见。</p>