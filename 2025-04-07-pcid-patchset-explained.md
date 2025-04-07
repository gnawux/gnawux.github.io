title: "PCID patchset explained in LWN style"
date: 2025-04-07 23:28:08 +0800
update:
author: me
cover: 
categories:
    - linux
tags:
    - kernel
    - pcid
    - lwn
    - pvm
preview: "这篇文章并不是来自 LWN 的，只是作者作为 LWN 的老粉丝，很喜欢 Jonathan 的写作风格，忽然觉得这个 patchset 可以这么解释一下，因此做了个拙劣的风格模仿而已，东施效颦，大家见笑哈。"
draft: false

---


> 作者按：这篇文章并不是来自 LWN 的，只是作者作为 LWN 的老粉丝，很喜欢 Jonathan 的写作风格，因此做了个拙劣的风格模仿而已，东施效颦，大家见笑哈。

我们都知道 Linus Torvalds 对 x86 架构一向[颇有微词](https://lore.kernel.org/lkml/CAHk-=wimnCtaDhCswqBUag37J1ALDno5dGv4v8Emv0b7SgVgPw@mail.gmail.com/)，在2024年香港的 Open Source Summit 上，Linus 就直接指出，相对于新的 ARM 和 RISC-V 架构的处理器们，各 x86 处理器厂商都应该增加多 CPU 之间的用于 TLB 管理的信息带宽，让 TLB 管理的时候可以不再需要 TLB Shootdown，不要发这些不必要的 IPI 。似乎是在响应 Linus，AMD 开始为他们的处理器增加 TLB Broadcast 机制，然而，在他们的内核支持代码中也引入了不少 Bug。最近，蚂蚁集团的内核开发者侯文龙发出了一个 [Patchset](https://lore.kernel.org/lkml/cover.1743250122.git.houwenlong.hwl@antgroup.com/) ，旨在修正 AMD TLB Broadcast 机制的 PCID （Process Context IDentity）管理相关的 bug，并在邮件列表里引起了一些讨论，按照蚂蚁集团的资深内核维护者赖江山的说法，这个 Patchset 可能也揭露了关于 PCID 管理的更深远的问题。

## 回顾：Meltdown 侧信道攻击与 KPTI

PCID 是 Intel 在 2010 年的 Westmere 微架构处理器中开始引入的，用于标识不同的地址空间，从而可以在不同的地址空间之间切换的时候，保持 TLB，优化性能。不过，之前这个字段并未被广泛使用过，直到 2017 年，著名的 [Meltdown 侧信道攻击](https://lwn.net/Articles/743363/) 让人们意识到，即使代码里做了进程间的逻辑保护，利用 CPU 分支优化产生的细微性能变化，当缓存保护不严密的时候，仍然可以向狡猾的攻击者泄漏出内核地址空间里的信息。于是，为了抵御侧信道攻击，4.15-rc6 和 4.14 稳定内核正式引入了 KPTI （Kernel Page-Table Isolation），隔离用户空间和内核空间。KPTI 机制最大的争议在于，为了保护安全，对代码执行性能有普遍的严重的损失，而使用 PCID 来区分不同地址空间，可以显著减少 KPTI 带来的性能损失，大部分开发者也是在这个时候才关注到了这个标识的。

Linux 内核中使用 ASID （Address Space Identity）作为内存管理子系统中的核心部分，在 x86 架构下，ASID 直接对应到了 PCID，不过，在过去这个字段的使用却是比较有限的，因此最近也是暴露出不少问题，比如这次侯文龙试图修复的三个。

## 这个 Patchset 修复了什么

这组补丁修复了什么（修复内容是大模型总结的，具体可以参考上文相关链接）：

1.	** 修正可用 ASID 数量的计算**第一个补丁调整了 `global_asid_available` 的计算方式，该变量表示可用于分配的 ASID 数量。此前，该值被设置为 `MAX_ASID_AVAILABLE - TLB_NR_DYN_ASIDS - 1`，导致丢失了一个有效的 ASID。侯文龙的修复移除了多余的减法运算，确保使用了正确的数量。
2.	**修复 allocate_global_asid() 中的逻辑错误**第二个补丁修复了 `allocate_global_asid()` 函数中的逻辑错误。原始代码中包含了一个多余的检查，用于判断 `global_asid_available` 是否为零，但由于此前已在 `use_global_asid()` 中进行了验证，因此该检查是不必要的。移除这个检查可以简化代码，并避免不必要的警告。
3.	**重新定义 MAX_ASID_AVAILABLE**第三个补丁重新定义了 `MAX_ASID_AVAILABLE`，使其表示可用 ASID 的数量，而不是最大 ASID 值。这一变更确保了用于跟踪已分配 ASID 的位图包含所有有效的条目，避免了排除最高的 ASID。

目前，这组补丁已经引起了一番讨论，Rik van Riel 总体肯定了这些补丁，并给出了一些他的 Review 意见，并且，侯文龙也表示愿意根据意见做出新一版的修订。

## 未来：PCID 管理的更多问题

这组补丁让我们注意到，即使是在内存管理这么重要的子系统里，也有一些比较隐秘的角落，甚至有这样的 off-by-one 错误来浪费资源。当然，这个补丁本身也说明了 PCID 正在逐渐被更多场合应用，比如蚂蚁内核团队在去年 Linux Plumber Conference 介绍的 [PVM （Pagetable based Virtual Machine）](https://lpc.events/event/18/contributions/1766/)，也使用了 PCID 来加速页表的切换。随着 PCID 被越来越多地使用，更有效地管理 PCID 也将在不久的将来成为一个新的关注点。