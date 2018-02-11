title: "从 OpenStack 到 OpenInfra"
date: 2018-02-07 18:58:08 +0800
update:
author: me
cover:
categories:
    - kata
tags:
    - openinfra
    - kata_containers
    - runv
    - openstack
    - openstack_foundation
preview: "在今后的 OpenStack 基金会相关的场合，大家将看到越来越多的 OpenInfra ，越来越淡的 OpenStack，这并不是一种抛弃，而是一种回归和升华"
draft: false

---

照例谢绪总邀。写这个短文的缘起是，前不久绪总发布了一条消息，今年的 “OpenStack 中国日”改名为 “OpenInfra 中国日”，随后有些老玩家不解地问，这件事基金会会不会有意见等等。我当时在群里提了一句，改名实际是基于基金会的决策做的，于是绪总邀请我写个简单的说明，作为 FAQ，节省他宝贵的口舌，于是我就这里献丑来略说一二，有什么不到位的地方，还请斧正。另外里面夹带了一些 Kata Containers 的私货，看在同样是项目成员的份上，还请绪总帮忙散播。

![本届峰会的口号已经是OpenInfra了](/assets/summit_infra.jpg)

最近如果投稿 OpenStack 温哥华峰会的话，在 [会议话题分类介绍](https://www.openstack.org/summit/vancouver-2018/summit-categories/) 里面就会看到这段话

> The OpenStack Summit is a four-day event covering everything Open Infrastructure. Content includes keynotes, presentations, panels, hands-on workshops, and collaborative working sessions. Expect to hear about the intersection of many open source infrastructure projects, including Ansible, Ceph, Kata Containers, Kubernetes, ONAP, OpenStack and more.
>
> (我简单翻译一下，看个意思哈)
>
> OpenStack 峰会是一项为期四天的关于 **各种开放基础设施 (OpenInfrastructure)** 的大会。会议包括主题演讲、话题演讲、圆桌研讨、动手实验与协同工作等。在会上将会听到各个开源基础设施项目的交流分享，包括但不限于 Ansible, Ceph, Kata Containers, Kubernetes, ONAP, OpenStack。

注意，这里面一来将 OpenStack 变成了几个主题之一，而不是大会的中心，二来继续强调了开放和基础设施。这正体现了过去一年来 OpenStack 基金会的转型。

实际上，在去年的悉尼峰会上，主题演讲已经开始提了 OpenInfra 了，但当时敏感到注意到这点的人可能还不多，只是我们一些 Kata Containers 项目最初的参与者们了解比较多。而当12月 KubeCon 2017 期间，我们在 OpenStack Foundation 旗下，却独立于 OpenStack 项目发布 Kata Containers 的时候，这个动作才比较明显。并且最近 Kata Containers 也将和 OCI 有进一步合作消息放出，这些都是 OpenStack 基金会的转型之举。

大家如果更多关注基金会的话，能注意到，在去年9月的旧金山，基金会还组织了个规模比较小，但是参与者包括了很多运营商、制造商的会议——OpenDev，这个会的主题并不是 OpenStack，而是时下热门话题之一边缘计算。也正是那次会议上，Hyper 和 Intel 开始具体推动了 Kata Containers 在 OpenStack 基金会旗下的合并工作。在 Kata 项目成立至今这段发展过程中，和基金会的员工们（上至CEO，下至 manager，当然他们全职员工本来就没几个人）在旧金山、丹佛、奥斯汀、悉尼、北京进行了多次交流，也算是从干活的角度见证了这一转型。

![基金会CEO Jonathan在OpenDev会上谈边缘计算的平台](/assets/opendev.jpg)

简单的总结一下， OpenStack 基金会2010年在奥斯汀成立的初衷，就不仅是山寨一份 AWS，而是剑指开源基础设施这片巨大的领域，联合IT设备商与IT大型消费者，通过开源的方式，让软件基础设施开放化、标准化，进入寻常企业，而如今，随着容器等基础技术的演进，边缘计算和物联网等新的应用场景出现，传统的 Nova 为核心的 OpenStack 软件虽然复杂而完整，却无法完全适配所有场景。于是，这次转型也正是回到基金会成立时的初心，理性面对 OpenStack 是开放基础设施的一个重要组成部分，但不是全部，将基金会影响范围扩大到了其他的开放基础设施项目。

所以，在今后的 OpenStack 基金会相关的场合，大家将看到越来越多的 OpenInfra ，越来越淡的 OpenStack，这并不是一种抛弃，而是一种回归和升华。
