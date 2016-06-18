date: 2015-11-15 08:08:08 +0800
update: ""
author: me
topic: ""
draft: true
top: false
title: "Kubernetes 中的 Service Account"
preview: "Service Account 是 Kubernetes 用于集群内运行的程序，进行服务发现时调用 API 的帐号"
categories: ["cloud"]
tags: ["cloud","hyper","kubernetes","hypernetes","service discovery"]
---

> 照例广告放最前：Hyper 作为一家新的、有梦想的、不做“Copy-to-China”的创业公司，我们欢迎想创新的、有想法的、有技术的小伙伴加盟，并且，我们允许远程办公。

口号版：

> Hyper是一家创业公司，致力于开发更加高效的云计算的基础设施。

百字版：

> Hyper是借助虚拟化来加强隔离性的 Docker Image 的执行引擎 -- Hyper 将 Docker Image 直接运行在虚拟机中，而无需 Guest OS，更无需在 Guest 中运行 Docker Daemon；Hypernetes 是使用 Hyper 进行容器隔离的 Kubernetes distro，同时利用 Neutron 隔离网络、利用 A

## 从前云计算时代到今天的云计算

我在[之前的文章](http://wangxu.me/cloud/2012/07/05/改变互联网的IaaS服务/)中，提到过 Zynga 的故事，这里再引用一次《Zynga价值观和创业文化》

> 因为Mafia wars太火了，据说NetOPS整个部门都在加班加点的在机房上架服务器，安装服务器，那时候还没有使用Amazon的AWS，还是靠数据中心。他们的压力非常大, 要上架的服务器数量太多了。以至于NetOPS当时的老板，夜里在上架服务器的时候实在找不到人手来帮忙，**他只好把他怀孕的老婆叫来帮他插网线**
> [...省略部分内容...]
> FarmVille团队也没几个人，他们也不想等着服务器错过了机会，他们当时据说都没和NetOPS部门商量一下就马上抱着试一试的态度使用了Amazon的AWS, 虽然遇到不少问题，但服务器可以马上就有还是不错的，随后FarmVille由于及时的推广和持续的更新，没有错过最好的时机，一下变成公司了最火的游戏。

Zynga 已经不像当年那么耀眼了，可是 AWS 上却在重复一个又一个创业故事，因为资源无限可用、随时可用、随用随付——所有采购、谈IDC、上架、布设网络等等这些资源相关的东西就和用户不再相关了，用户只需要一些用来管理基础设施的代码而已。

## PaaS的梦想与现实

> 游走在易运维和可迁移之间

毫无疑问，AWS引领的 IaaS 至今仍是最为用户所接受的云服务，然而，这并不意味着 IaaS 已经完美了。简单地说，IaaS帮用户减少了机房相关的维护工作，但用户仍然需要管理虚拟机集群，运维的工作同样是繁复的。如果用户使用了

## 黄金分隔点

> Docker的颠覆性在于以应用为中心

## 虚机与容器的边界

> 虚拟机的问题不在“虚拟”，而在于“机”。

## “真正”的CaaS

> “计算机科学领域的所有问题都可以通过增加一个间接层解决，除了间接层过多的问题” —— David Wheeler


