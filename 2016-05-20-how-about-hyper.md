title: "[知乎回答] 如何评价 Hyper_"
date: 2016-05-20 10:08:08 +0800
update:
author: me
cover:
categories:
    - zhihu
tags:
    - hyper
    - zhihu
preview: 原题描述：根据官网所说，hyper_ 貌似是一个基于 VM 的分层系统，比 Docker 共享 Kernel 更加安全，但是比一般的 VM 系统快得多，能做到秒级启动，而且本身非常精简。不知道有没有大牛已经用过了或者感兴趣，对它的评价是怎样的？
draft: false

---

答案链接：https://www.zhihu.com/question/35412725/answer/101715150

为啥没人邀请我？虽然我在知乎上一直专注于无线通信相关问题，不过，我确实是 Hyper_ 的联合创始人和主要作者之一。

前两天， Docker 的 @Honglin Feng 在某群里跟我说知乎的广告效果不错，我还说
“我都是回答的那些神马在学校的时候学的，没来得及忘掉，也不会用到的东西，废物利用”，
没想到今天就有人给我这个亲手打广告的机会，不由得还真是有点激动呢。而且这个题居然不是我们自己人提的，颇感意外啊。

先从整体上说一下。刚才 @Gnep 提到，可以关注 [HYPER_](https://www.hyper.sh) 这个我们自己做的基于 HyperContainer 的容器云。
是的，提到我们 Hyper_ ，大致可以提两件事情 ——

- 一是，我们的容器技术 HyperContainer，就是题主写到的“Make VM run like Container”，
  或者叫“Secure as VM, Fast as Container”，这个我们是开源的，位于 [Hyper_ · GitHub](https://github.com/hyperhq/hyperd/) ，
  简单地说是结合了 hypervisor 和 Docker Image 的一种更强调隔离的 App Container，和我们很相似的技术是 Intel 的 Clear Container，
  我们是同一个星期 Release 的（2015年5月），并没有互相参考，思路暗合；
- 二是基于我们的容器技术做的容器云 Hyper_ ( https://www.hyper.sh )，因为我们的底层隔离了，上层可以做得更简洁，
  完全去掉了一般公有服务里的基于虚机的资源池这一层，让用户可以像在一台无限大、永远在线的个人电脑上跑 Docker 一样，
  直接创建并运行 Container。因为没有资源池这一层，需要初始化的东西很少，所以可以在三秒钟左右的时间里（
  这个和 container 包含不包含额外的卷有关系，有额外的数据卷需要的时间要长一点）跑起一个虚拟机，
  并且每个用户的网络和容器都是完全隔离、互不干扰的。而且，因为真的很快，所以秒级计费是有实际意义的哦。

```
---->8------进入细节的分割线---->8------
```

以上和以下提到的 Docker 均为 Docker Inc 的商标，属于 Docker 公司的知识产权。

首先从开源项目的由来说起，在最近的两三年中，容器成为云计算领域了最热门的话题。回顾历史，从 FreeBSD Jail
算起容器技术出现在 *nix 领域中已经发展了接近二十年了，致力于在操作系统中进行更强的运行时隔离。

然而，Docker容器与以往不同，不仅在于运行时隔离，更在于对应用及其依赖环境的标准化封装，从而做到跨部署环境的不变性（immutable）
和一致性（consistency）。可以说，这是一种新型的应用容器。
这些应用容器对应用运维的方式带来了颠覆性的效果，然而，运行时容器技术并不足以面对多租户的环境。
Linus 在2015年的 LinuxCon 上也坦陈，安全性问题常常是个Bug的问题，而犯错不可避免，所以，要想达到好的安全性，
只靠安全加强是不够的，多一层的封装才是解决之道。在这种背景下，自然地期待保持应用容器的用法，引入虚拟化的强隔离。

HyperContainer就是这样一种技术，结合了虚拟化的强隔离和容器的轻量级——Secure as VM, Fast as Container。
HyperContainer 并非是简单在虚机里放一个 Docker Daemon，而是用 hypervisor 替换掉了基于 namespace 的 runtime，
在虚机里面并没有 daemon，也没有完整的 guest distro，而是直接将 Docker Image 放在定制的 Guest Kernel
上执行的，所以可以做到很快。并且如其他的回答提到的，从用法上说，我们和 Docker 等容器技术是一致的。时间线大致如下——

- 我们从2015年初开始做这个项目
- 2015年5月以开源项目的形式发布
- 7月，随着 LinuxFoundation 宣布了开放容器促进组织，Hyper_ 将 HyperContainer 的运行时部分分离为 runV 项目，列为 OCI 的参考实现之一（runtime-spec/implementations.md at master · opencontainers/runtime-spec · GitHub）
- 2015年10月，又发布了Hypernetes项目，结合了 HyperContainer 容器技术，Kubernetes 调度引擎以及OpenStack 的 Neutron 和 Cinder 等，成为一个多租户的 Kubernetes 发布。

就社区而言，Hyper 作为一组中国人发起（但运行得略国际化）的开源项目，目前在国际上的影响还是略有一些的，除了上面提到的 OCI 之外，Hyper团队正在和 Kubernetes 社区积极合作，Kubernetes 已经计划在未来的版本中集成对 HyperContainer runtime 的支持；另一项由兄弟创业公司数人科技的 @肖德时 肖总推进，并有前阿里员工参与的工作（[MESOS-3435] Add containerizer support for hyper）正在将 HyperContainer runtime 集成到 Mesos 项目中去。

```
---->8------关于 https://www.hyper.sh 云服务的分割线---->8------
```

目前的容器云服务有很多，大到 Google 的 GKE，新到国内的一系列云，但 Hyper_ 有所不同，大部分 Docker 用户都满意 Docker
的命令行用户体验，然而，部署成服务的时候，却需要先拥有一个虚机的资源池，并为这个资源池付费，而非实际跑的 Container。

而 Hyper_ 是直接基于 HyperContainer 和 Hypernetes 项目的，Container 的运行时隔离由 HyperContainer
保障，网络隔离由 SDN 完成，我们还做了一个新的 Image/Container 存储引擎，让集群的分布式对用户完全透明，
所以，用户用起来和用自己的笔记本没什么不同，包括像 Docker Link 这样的功能我们也是提供了的，而且在命令行和 API
上都和 Docker 一致。如果说和 本机有什么不同的话，Hyper_ 上，用户可以直接给容器关联公网 IP，这是 Hypernetes
项目给我们带来的额外好处。

从用户体验的角度看：

- 一是简单快速，没有多余的东西，虚机是什么鬼，cluster 是什么鬼，都不需要关心，更不用管调度器是土鳖还是高大上；
- 二是随时起停，当起停的时间很短的时候，起停都不是负担，比如你在上面跑 s***socks 的时候，可以需要的时候跑，不需要的时候停，简单绿色、低碳生活；
- 三是可以按需使用，用更少的资源更少的钱，完成恰如所需的任务，不用跑不需要的服务。

当然，虽然我自己不太年轻了，但我们还是个年轻的开源项目、年轻的创业公司，项目还在快速发展，每天都有新的 PR 被 merge，
有新的功能改进和 bug fix；云上面，Compose 这些高级功能也很快就会提供。如果有问题，可以随时在 github 上 file issue 或联系我们。
