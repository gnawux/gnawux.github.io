title: "个人业余项目: 基于 Kata 和 PVM 的云上沙箱服务 Xolis"
date: 2026-08-09 23:56:00 +0800
update: 
author: me
categories:
    - open-source
tags:
    - kata-containers
    - sandbox
    - virtualization
    - container
    - kubernetes
    - xolis
    - kubecon
    - demo
cover: /assets/kubecon-jp-2026-demo-on-stage.jpg
preview: "正是在这些开源工作的基础上，我们才可以用一个星期就实现一个 Agent Sandbox 的服务的开发，展示这些社区开源项目里已经成熟且高效的部分，正是我们的这一演示的目的。"
draft: false

---

![横滨 KubeCon Japan 2026 演示现场](/assets/kubecon-jp-2026-demo-on-stage.jpg)

在前不久的 KubeCon Japan 2026 上，我和 Japan AI 的 Huyu 演示了一个基于开源技术栈的沙箱服务的构建——在 AWS EKS 上运行 Kubernetes 的 agent-sandbox CRD，在节点侧使用 Kata runtime-rs 构建 sandbox，这个沙箱使用 runtime-rs 内建的 dragonball rust-vmm，shim/runtime/vmm 三合一，无需任何额外进程，不论是管理开销还是内存开销都压缩到了最低；另一方面，得益于 dragonfly 和 nydus，可以极大地压缩沙箱所需的镜像拉取时间。

这个演示项目就位于 GitHub 上：[gnawux/xolis](https://github.com/gnawux/xolis)， `kubecon-japan-2026` 这个 tag 对应了演示的完整状态，随后一个星期里，我继续把 pvm 的支持也加入到了项目之中，有了 pvm，Kata VMM 可以在任何云上主机中运行，比如 AWS EC2 的任意 x86 instance，而无需 nested virtualization 支持，对于部署的灵活性和成本管理都更加友好，这个版本就指向 `demo-with-pvm`这个 tag。未来我会在有空的时候继续做下一步实用化的开发，至少作为 Kata 社区的一个可用的 sandbox 演示项目。

## 缘起：社区需要被看见

这次演示的缘起是在今年3月的阿姆斯特丹，我和 huyu 在 Kata 的展台上相遇了，他来问一些 Kata 相关的信息，探讨 Sandbox 的方案，而彼时我作为 Kata 的社区的退休老登，正在那里闲聊，一起讨论了 Kata 之后，考虑到 KubeCon Japan 的投稿关门时间很接近了，我便提起可以一起做一个基于 Kata 的 Sandbox 的 Topic，就这样一拍即合了。刚好社区一向喜欢这种有社区又有本土，有上游又有应用，并且串起来多个开源项目、听众也可以实操的合作话题，很自然地就中稿了。

其实我脑子里想做这样一个项目的想法已经有一段了，起因一方面当然是 e2b 有一点火，而其实这和我们当年的容器云服务是有一点点类似的；另一方面是众多的公司和团队都推出了他们的 Sandbox 项目，而这些项目都在比较的时候把 Kata 作为了一个对比项，认为自己的项目比 Kata 快很多倍或开销低很多，但是他们没有详细介绍过他们的 Kata 环境的配置，根据他们的测试情况，我认为他们从来都没有（也没想去）认真调优过用于对比的 Kata 环境。

实际上，集成了 dragonball rust-vmm 已经在上游很久了（去年年底就进入稳定状态），这个版本也几乎就是蚂蚁和阿里在大规模生产环境中使用了很多年的配置，作为一个用 Rust 语言写成的单进程沙箱，它从一开始的目标就是低开销和易运维。刚好，这个版本在历经很多繁复的测试和验证之后，在7月21日正式发布了，我在7月29日的社交媒体上是介绍这一里程碑的：

> 全 Rust 化是我和 Samuel 在 2019年 Denver Summit 正式开始推动的，从波哥的 `rust agent` 开始，到攀哥主导的 `runtime-rs`，在劝说 clh 出 kata-profile 无果的情况下，把阿里开启的 dragonball 最终集成为 runtime 内建的 `rust-vmm`，其实蚂蚁和阿里内部版本这么跑很久了，今天下午我会秀一个用 kata-4.0 的开源 stack 的 `agent sandbox`，all-in-one rust binary 可是干净太多了。

## 关键因素：开源基础，特别是 PVM

当然，Runtime 是构成一个沙箱系统不可或缺的一环，但并不是唯一的一环，在我的 Xolis 演示项目里，用到了众多的开源项目，并且只用到了开源项目，没有任何大公司的内部项目，简单地说，主要分为两个链路：

- 运行时链路：Kubernetes ~[Agent Sandbox](https://agent-sandbox.sigs.k8s.io/)~ 定义了沙箱行为和 Warm Pool，通过标准的 Kubernetes 和 Containerd，创建 Kata Sandbox，这个 Sandbox VMM 跑在 kernel kvm 之上，通过 [PVM \(Pagetable-based Virtual Machine\)](https://lpc.events/event/18/contributions/1766/) ，我们的 kvm 可以不依赖硬件虚拟化的辅助；
- 镜像链路：通过镜像加速项目 ~[Nydus](https://nydus.dev/)~ ，可以使用遵循 OCI Image artifacts 规范的 Nydus 加速镜像格式，从标准的镜像仓库中，通过 Lazy Load，以很低的时延拉取镜像，并可以在鸡群中通过 ~[Dragonfly](https://d7y.io/)~ 进行 P2P 传输加速。

这其中，我特别想介绍一下 PVM，它使得基于 KVM 的 VMM 无需硬件虚拟化辅助即可运行，不需要 CPU 具有 `vmx` 标志，这也就让基于 Kata 可以直接跑在任何虚拟机里，不论它是8代以前的 EC2 instance，还是其他公有云的服务器，都可以运行。这对于基于虚拟化的强隔离容器来说，意味着首先我们可以在任意虚拟机里调试 Kata 的环境了，极大地便利了开发；然后就是可以扩大部署环境的范围，让它真的像普通容器一样，没有运行环境限制。正因如此，我愿意称 PVM 加持的 Kata 为 Kata 的完全形态。

PVM 的 PatchSat 已经投递到了 LKML，相关论文也已经在系统顶会 SOSP 和内核开发者大会 LPC 上宣讲过了，感兴趣具体内容的朋友可以去查阅代码和论文，这里我简单提一下原理——从代码框架上来说，它利用了 2007 年开始，逐步进入内核的 Xen 引入的 `pvops` 框架，允许 Host 和 Guest 内核进行协作；从封装上来说，利用了页表带来的地址空间隔离性来隔离 Host 和 Guest，PageTable 也是 PVM 名字中的 “P”。利用这些已有基础，PVM 得以用很小的改动，来支持了 KVM 的一个半虚拟化驱动，从而让我们略略 patch 一下内核来实现无需硬件支持的 KVM 虚拟化。

PVM 是由我们团队从 hyper.sh 到蚂蚁时代的内核开发者赖江山，最近几年的发起的一个力作，但这已经不是他在轻量级虚拟化这个领域里的第一个突破性成果了，之前，他还曾经利用虚拟机 Live Migration 的特性，进行一次不拷贝内存的本机到本机的 Migration，从而实现了虚拟机的快速启动的 “Template 技术”，这一技术后来被我们团队的另一位骨干同学彭涛 Port 到了 Kata 上，成为加速 Kata 冷启动的标准方案。

正是在这些开源工作的基础上（除了尚未合入内核的 PVM 和我的演示项目，其他均为 OIF 或 CNCF 的社区项目），我们才可以用一个星期就实现一个 Agent Sandbox 的服务的开发，其实，即使你不用 Kata，从 PVM 到 Nydus 这些开源贡献，仍然可能被用在了一些其他团队的“创新方案”中，展示这些社区开源项目里已经成熟且高效的部分，正是我们的这一演示的目的。

## 当前状态：已实现的和下一步

目前，这个项目里包含了几个主要部分：

- 总体架构设计、说明文档、演示相关文档和架构图；
- 在 AWS 上创建集群的 OpenTofu 代码、加工 AMI 和容器镜像的代码等；
- 一个简单的用于演示的服务代码，主要由 Rust 和 Python 写成；
- 相关测试代码，其他相关的辅助代码和必要补丁。

![演示示意图](/assets/xolis-demo.png)

所有这些代码和相关测试都是通过 Agent 辅助编程来开发测试的，目前的服务代码还比较简单，未来如果提升服务规模，需要额外使用数据库、进行多 AZ 部署、高可用的代码等；另外，网络流量相关功能还比较简单，下一步可以考虑丰富，比如 Kata 的 TSI 功能，可以将 Sandbox 內的网络劫持到容器之外，从而进行更多的逻辑控制等；以及，目前只有 AWS 一个云的支持，也只用了 EKS 的 K8s，如果做成更完善的应用，多云多服务商的支持也是一定会考虑的。

## 未来考虑

目前，我首先希望完善这个 Demo 的内容，并且确保各种社区的功能都完整地被集成进来；在演示的开发过程中，Kata 上游的老朋友们也给予了很多的支持，这个项目我也希望可以进一步推动成为社区的一个基准用例或者演示项目。Anyway，Demo 完成就是成功，开始下一步之前，我们先喝杯咖啡，休息一下吧。

