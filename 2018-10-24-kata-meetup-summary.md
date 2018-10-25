title: "Kata Beijing Meetup 小总结"
date: 2018-10-24 22:08:08 +0800
update:
author: me
cover: "/assets/kata-meetup-1.jpg"
categories:
    - kata containers
tags:
    - meetup
    - hyper
preview: "上周三（10月17日），我们在北京环球贸易中心组织了 Kata Containers 在国内的第一次独立的 Meetup，反响热烈，交流深入，在此按照组委会的要求，来写一份家庭作业总结一下。"
draft: false

---

![合影](/assets/kata-meetup-1.jpg)

上周三（10月17日），我们在北京环球贸易中心组织了 Kata Containers 在国内的第一次独立的 Meetup，反响热烈，交流深入，在此按照组委会的要求，来写一份家庭作业总结一下。

## 感谢

本次 Meetup 是 Intel, Huawei, Hyper.sh (H2I) 三方共同发起的，在此首先感谢场地提供方 Intel 和午餐赞助商华为的大力支持，没有赞助商们的大力支持，当然不会有成功的大会了。更重要的是，赞助商们不光出钱，还花费了大量的组委会时间进行组织、沟通、联络、材料准备等工作。特别感谢 Intel 的冰姐和运通二位，进行了大量的联络和筹备工作，还有负责后勤工作的齐乐，做了非常完备和周密的会场相关各项准备。

当然也要感谢 OpenStack Foundation，尤其是昊阳同学，在他加入基金会一事还处于地下状态的时候，就为我们提前泄漏了基金会管理层在10月的行程，并最终促成了 Jonathan 一行参加了 Meetup 并给出了开场致辞，让这个 Meetup 有点 Mini Summit 的感觉了。

## 缘起

致谢之后，我们回到时间线上来，这次 Meetup 大概要从6月的 OpenInfra Days China 活动说起。在那次活动上，我们介绍了 Kata 1.0 和之后的方向，同时也和 Intel 国内的同事们有了一次很美好的合作经历，那之后不久，冰姐就首先提起了搞个专门的 Meetup 的想法和我以及华为的张伟、梁辰晔几位大佬交流，考虑到社区开发进展和 Berlin Summit、云栖、华为 Connect 等国内外会议的时间，我们把时间定在了金秋十月，并如上所说，综合昊阳同学泄漏的基金会管理层行踪，确定了时间，提前开始了组织工作。

## 筹备

筹备工作事实上不应该由我来总结了，因为我毕竟是干得最少的人之一，冰姐在 Meetup 当天早上还表示，在梦里她都在操心 Meetup 组织活动，而我，是她梦里最掉链子的两个家伙之一…… 至于另一个…… 我就不透露了，总之，组委会的朋友们付出了巨大的努力让整个 Meetup 活动看起来从容不迫、有条不紊，远非一日之功的。

组委会提前了两个月左右就确定了主要的话题，并积极邀请了很多公司，从报名情况看，百度、阿里、腾讯、金山、蚂蚁金服、网易、青云等一系列国内互联网、云服务企业，还有银联、联通、苏宁、中兴、华为、联想、IBM、斯博伦等国内外电信、设备、金融公司/机构也都参加了活动，并进行了积极交流，直到最后两三天还不断有新报名参加的人，可以说，热烈的反响是对组委会筹备工作最大的肯定。

筹备工作唯一黑点在我这，本来我答应运通和冰姐准备一个 Demo 的，然后……最后我改成只介绍社区进展了。

## 召开

10月17日，Meetup 如期召开，当天的北京一扫前几日的阴霾（小学生作文风啊……），我赶到会场楼下的时候刚好碰上运通、昊阳和基金会一行，之后上楼就看到了其他组委会的成员都已经就位了。会议首先是组委会致辞，之后还邀请了 Jonathon 说了句 "Hi"。然后就是运通和我的开场介绍，说好他10分钟给我20分钟的，但实际上他和冰姐联手之后，我只剩下了5分钟来介绍社区进展，我大概用近几年来最快的演讲语速和翻页速度一口气翻完了几页幻灯片，准备的梗完全没有施展空间，还是超时了3分钟，不过无碍干货部分，再加上社区开发活动完全是在网上开放的，我的介绍其实也没啥讲太多的必要。

![开场](/assets/kata-meetup-2.jpg)

上午的正式 session 来自百度、华为和中兴：

- 来自百度的白宇的话题是关于他们的边缘 Serverless 场景的，在这个场景中，Kata 被用作他们的自研的调度系统中的沙箱容器，来保障对非信任代码时的隔离性。
- 和笔者同为 Kata Arch Committee 成员的张伟，这次带来的是华为容器云的介绍，在介绍华为方面的一些底层开发之后，他也慷慨展示了一套基于 Kata 的多租户 Kubernetes 服务，让很多还没有得到试用邀请的在场观众首次亲眼见到了华为云容器服务的交互界面。
- 上午最后一个技术话题是来自中兴的苗艳强介绍的 NFV 相关的应用，和张伟类似，这也是一个基于 kubernetes 的应用案例，但因为 NFV 这个背景，ZTE 的案例更关注网络方面的功能和性能。

让人颇为兴奋的一点是，三位演讲者的话题不仅有场景介绍、应用案例，也包含了对上游的一些问题反馈，以及在 Kata 社区提出的 Issue 和 PR。

午饭后回到会场，下午开场，来自 OTC 虚拟化团队的钟阳为大家介绍了颇受关注的 Qemu 轻量化改造项目——NEMU，这些工作和 Kata 的集成已经在进行中了，并且也在提交回 Qemu 社区。在 Nemu 介绍之后，大家进行了大概两个小时的分组讨论，对各自关心的议题进行了深入探讨。

## 成果

![讨论](/assets/kata-meetup-3.jpg)

一个 Meetup，最主要的成果当然是网友见面了，有些是新网友见面，有些网友几个月前还见过，也有些网友几年没见了，却因为 Kata 再次相逢。Meetup 在分组讨论之后总结了各自的问题，我、张伟和其他相关的参会者也一一做了解释、建议或记录，会后，会议总结也反馈给了社区的邮件列表，大家相约把自己的 Issue/Fix 回馈给社区，共同进步。

## 附: 发往社区邮件列表的总结

On Oct 17th Wed, we held a Kata deep dive meetup in Beijing. Most of the major cloud providers, and ICT equipment vendors in China attended the meetup and exchange their thoughts, practice, and use cases. [The participants](https://etherpad.openstack.org/p/kata-meetup-beijing-2018) included Baidu, Alibaba, Kingsoft, Tencent, QingCloud, Suning, IBM, UnionPay, Netease, China Unicom, ZTE, Unicloud, AntFin, Lenovo, Spirent, etc. The one-day meetup was supported by Intel (Sponsor), Huawei (Sponsor), Hyper.sh, and OpenStack Foundation. I'd like to thank Maggie Liang from Intel especially, who effectively led the organizing work of the meetup.

The executive director of OpenStack Foundation, Jonathan Bryce, who was visiting China in the week, gave a short opening speech for the meetup. And before the sessions, Yuntong Jin (Intel) and I (Xu Wang from Hyper.sh) gave an introduction of the meetup and status update of the development in Kata Containers community.

In the morning, developers from Baidu, Huawei, and ZTE introduced their practice on Edge, Public Cloud, and NFV cases. And Intel Developers gave an introduction of NEMU VMM in the early afternoon. After all the sessions, the attendees were grouped by topics and had a two-hours open discussion. The following topics are addressed:

- In-sandbox networking policies: 
  - Case:
    - comes from Baidu;
    - not a kubernetes scenario;
    - configure a network interface for the sandbox and connect to the management network;
    - group the processes in the sandbox to two groups based on PID
    - the processes from the provider could access local (mgmt) network, while the other processes that running guest binaries don't have the permission.
  - Comments from Xu: we could generalize the requirement to apply different networking rules for different processes in the sandbox;
- VLAN networking mechanism support and hotplug
  - Case: 
    - An NFV scenario;
    - Configure different VLAN for different tenants instead of overlays
  - Comments from Xu:
    - The bridge + veth pair method might work for VLAN;
    - The user could write their own CNI plugin for accelerating;
    - There is ongoing work in the upstream community to help general CNI plugins to support Kata, however, it might need to modify the current CNI interface and existing plug-ins;
- Networking performance metering and tuning:
  - Many participants care about networking performance;
  - It's appreciated that all the users could provide their real-life cases, which could help the community to find and fix the issues better.

And there are many other topics are discussed as well:
- 9PFS performance and alternatives;
- Block device based rootfs support;
- Memory footprint optimization;
- Streaming media offload support;
- Vsock support;
- Container image management issues for big container images (such as some tensorflow images are 8GB+).

Some of the feature requests or issue will be raised in Github.

