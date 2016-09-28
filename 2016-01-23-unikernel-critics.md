date: 2016-01-23 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: "关于 Unikernel，你注意到或没注意到的一些东西"
preview: "Docker 收购了 Unikernel System，引起讨论纷纷"
categories: ["container"]
tags: ["docker","unikernel","hypervisor","hyper","container"]
---

> **利益相关**（强行植入广告）：我们（Hyper）创业团队（也是开源项目）同样做了一种基于 Hypervisor 技术的容器解决方案，然而与 unikernel 不同的是，我们是面向通用的容器镜像的，可以直接运行已有镜像。在我看来，大部分人在热捧 unikernel 的时候，实际上他需要的是 Hyper 这样的技术。

> **免责声明**：我所做的预测只对我自己负责，我们在用创造未来的方式来预测未来，并且有其他人与我们在一同来创造未来，我们坚信我们的预测是正确的。

![作为一个概念的 unikernel](/assets/unikernel-in-article.png)

## 新闻背景

前两天，[Docker 宣布收购了 Unikernel Systems](https://blog.docker.com/2016/01/unikernel/)，引起了朋友圈刷屏（比如[这个](https://mp.weixin.qq.com/s?__biz=MzA5OTAyNzQ2OA==&mid=401469511&idx=1&sn=d4628798a25b2c28d6d0acc2edf85068)和[这个](https://mp.weixin.qq.com/s?__biz=MjM5MDE2MTczMQ==&mid=401874323&idx=1&sn=9e78416191cb7b4b2cf04684b97d4750)）。不过 Twitter 上还是有一些批评声音的，而且 Joyent 的 Bryan Cantrill 大神亲自向信众解释了[为啥 unikernel 不适合于生产系统](https://www.joyent.com/blog/unikernels-are-unfit-for-production)，当然，在存在利益相关这方面，他和我是一样的。

在和一些技术圈的朋友讨论后，我发现有的朋友并不是很了解 unikernel 技术，只是听起来觉得很好。于是，昨天我在朋友圈里写了这么一段话

> 事实上，unikernel 和 docker image 都是对应用的一个静态封装，从很多种意义上说，unikernel 都是一个更精巧的封装，而 docker image 则是一种更简单、松散、或者说更通用的封装。对于每个人是不是有意义，这因人而异，历史上精巧专用的方案被简单而更具兼容性的方案打败的例子比比皆是。这里，很多人都忽视的一点就是，从 unikernel 的定义出发，就不存在一个 unikernel 可以适应多样的已有二进制程序。需要好好审视自己的应用场景，才知道这张旧船票能否登上你的客船

现在我在 Blog 里继续解释一下。

插一句题外话， unikernel 是一类技术，并不只有 Unikernel Systems 一家拥有，有兴趣的同学可以看看 OSV，这个团队来自 Red Hat，头领是 KVM 的作者 Avi，是一个非常优秀而有执行力的团队。

## 什么是 unikernel

根据 [Wiki 定义](https://en.wikipedia.org/wiki/Unikernel)，unikernel 是专用的、单一地址空间，将操作系统功能作为库调用的程序镜像。也就是说，做一个 unikernel 应用，可以不需要复杂、完整的操作系统，可以只包含需要的功能。想想这简直是小而美的微服务架构的终极追求。

Docker 镜像之所以为人们所称道的原因就在于它完整封装了应用环境，使得应用可以在不同的环境里无需修改直接运行，不用考虑 distro、依赖库之类的问题，而这里，unikernel 连 kernel 都一并打包了，只要放在 hypervisor 上即可运行，这种 portability 似乎是无与伦比的，另一方面，hypervisor 带来的隔离效果要比 linux container 更好，而且，去掉不必要功能后，unikernel 包含的可能含有漏洞的代码也会少，从安全的角度讲，你做得越少，你做错的也就越少，人们不禁赞叹，这真是终极的解决之道啊。

不过，等等，这里似乎有点问题啊。单一地址空间、操作系统作为库，这样的话和传统的程序有点不一样啊，程序不是跑在 Linux kernel 上了，那么多进程调度谁来做？已有的程序和库怎么用呢？是的，这就是我所说的不存在可以直接适配多样的已有二进制程序的 unikernel，你需要为每个程序一切从头做起，做自己的 unikernel app。换一个角度说，实际上 unikernel 不是让你把你的已有程序和它依赖的 kernel 功能打包，相反，是提供一套工具，帮你把你的程序嵌入到内核里去，得到一个你自己的专有的内核，这要从源码出发，甚至进行一定的移植工作。

所以，当我在看到这个新闻的时候，甚至在想，docker 这是要回到 PaaS 上去，放弃统一镜像，转而做源码级的服务么？

## 通用的 unikernel？

看到我前面的论述，有人会立刻说，把 init 程序，比如 systemd 甚至 Hyper 的 hyperstart 和 unikernel 打包，然后不就可以直接接入已有二进制程序直接，做成通用的 unikernel 了么？

可是，仔细看看啊各位，首先，从 API/ABI 的角度看，通用的 Unix 或者 Linux kernel 是建立在完整的系统调用集的基础上的，unikernel 通过将内核代码与 App 代码的耦合，放弃了对系统调用的提供。而一旦转而提供完整的 ABI 给应用，那么，和 Linux kernel 也就没有区别了。所以说，unikernel 在提供通用性和进行应用边界划分的时候，选择的不是一个更加普适的接口。或者说，当它转而通用地支持已有的二进制程序的时候，那么它重新变成 hyper 的解决方案，而不是 unikernel 了。

## Unikernel 化一个应用，你面对的问题

除了丧失二进制级的应用通用性之外，Unikernel 其实还面临着很多其他的问题：

- 首先，你需要移植程序，相当于支持一个新的平台。各种 unikernel 技术多少都提供了和 libc 或 java 平台或其他语言环境一致性非常高的开发环境，方便用户的移植，然而，移植工作多少还是需要的。
- 其次，如果你的应用本身是多线程或多进程的，需要考虑单一地址空间的限制，这不仅是程序语言方面的限制，有的时候也是架构或语义层面的影响。
- 最后，如 Bryan Cantrill 所说，unikernel 程序实际是完全内核态程序，而且只有单一地址空间，这根本没法 debug 嘛，传统或现代的通用程序调试设施基本上都无法使用，连进程的概念都没有，这对开发和调试带来了极大的难度，显然并不是所有的应用开发者都能驾驭这样的应用的。

## Unikernel 适用的场景有哪些

那么，什么应用需要这个场景呢？我个人认为，只有极端追求性能和小体积的场所，可以不惜代价地去做这种 unikernel，就此，很多人都觉得 IoT （物联网）似乎是最佳应用场景之一。我只想问，我们早年间在 4KB ROM，128B RAM 的限制下写的 51 单片机程序在算不算 unikernel 程序？

## 你真的想要的是什么？

感谢各位看到这里，作为一篇到处植入广告的文章，能看到最后的都是真爱，所以，我就再插入一段广告吧。如果你的容器服务跑在可信任的环境里，性能是首要要求，那么基于 Linux Container 的 Docker 是你的首选；而如果要提供多租户的容器服务，或者容器里的应用不被完全信任（比如用于测试等用途），那么，基于 Hypervisor 的 Hyper 完全胜任这个环境。并且，两者使用完全相同的镜像，对大部分镜像来说（除了那些刻意放弃主机和容器之间隔离性的应用），Container 能跑的，Hypervisor 也同样能跑。