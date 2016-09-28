date: 2008-06-04 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: Using OCFS2, A Clustering File System
date: 2008-06-04 12:35:00 +0800
categories:
- translation
tags:
- filesystem
- ocfs2
- translation
status: publish
type: post
published: true
---
<p><a href="http://www.linux-mag.com/id/6070">http://www.linux-mag.com/id/6070</a>

<p>了解一下下一代 Oracle 集群文件系统，GFS的一个替代品。</p>

<p><b><a href="http://www.linux-mag.com/author/55">Jeremy Garcia</a></b></p>

<p>Monday, June 2nd, 2008   <br />翻译：王旭    <br />2008年6月3日，星期二</p>

<p>传统的 Linux 重，如果你需要从多台机器上访问一个文件系统，那么就必须在一台机器上加载它，然后通过 NFS、CIFS 或其他方式 export 它。这样做是稳定、成熟而且有良好说明文档的，不过，这样做有它的明显不足，性能还不是最显著的一个，这样还会带来锁问题、潜在的安全问题、还有最糟的单点故障问题。</p>

<p>如果你的 NFS 服务器宕了，所有的客户端都会立刻无法访问。幸运的是，还有另一个方法来达到这个功能。集群文件系统是一种同时加载到多台服务器上的文件系统。实现一个集群文件系统在过去是非常复杂而昂贵的。通常包含了高价的SAN，光纤通道 HBA 卡、交换机，以及私有版权的文件系统。</p>

<p>现在，我们有了 iSCSI 和开源的集群文件系统可用。最流行的集群文件系统是 GFS，这是 Red Hat 在收购 Sistina 之后开源的集群文件系统。如今，GFS 随 Fedora 和 CentOS 一起发布，并在 RHEL 上提供了可选支持。从 2.6.19 内核开始，GFS 进入了主线内核。</p>

<p>另一个可选的文件系统是OCFS2，这是下一代的 Oracle 集群文件系统。这是一种可扩展的、POSIX兼容的文件系统。与前一个版本（OCFS）不同，OCFS2 是一个通用文件系统。到本文写作时为止，你可能会注意到 OCFS2 不支持 mmap (译注：VFS 的文件操作方法中声明了 mmap 方法，用于将文件映射到指定的地址空间上；译注2：据Colyli介绍，OCFS2 1.4里，已经实现了mmap方法，因为他正在调试一个相关bug)。尽管在不久的将来 OCFS2 将会支持它，但如果你现在就需要 mmap 的支持，你最好还是去看看 GFS。</p>

<p>OCFS2 的用户空间工具在多个发布版中可用，可以从 Oracle 或发布版处获得。OCFS 以 GPL 发布。<a href="http://oss.oracle.com/projects/ocfs2/">这里</a>是项目的主页。要使用 OCFS2，你需要安装 ocfs2-tools 包和相应的内核模块（ocfs2-`uname -r`）。</p>

<p>ocfs2console 包不是必须的，但为了使用方便，建议安装。安装了这些工具后，我们就可以建立一个 OCFS2 分区了。设置共享存储不在本文的讨论范围之内，我们假设你已经完成了这一步了。本文的例子是在 RHEL 5.1 环境中进行的。</p>

<p>不同的发布版中，一些命令或文件的位置可能不尽相同。即使 Oracle 没有提供当前的 RHEL 的内核对应的 OCFS 内核模块 RPM 包，你也可以从官方的 ocfs2 tarball 轻松地编译一个出来。和GFS一样，现在OCFS2也是主线内核的一部分了，所以，如果你用的发布版足够新的话，找内核模块包之前可以看看是不是已经支持OCFS2了。</p>

<p>首先，你需要编辑 /etc/sysconfig/o2cb，确保它包含这行内容：</p>

<pre>O2CB_ENABLED=true<br /></pre>

<p>接下来，在每台主机上创建 /etc/ocfs2/cluster.conf 文件。下面是用于三个节点的集群的设置。对于多个接入IP的主机，节点的参数应该对应于主机名，不论它在使用哪个IP。</p>

<pre>node:<br />        ip_port = 7777<br />        ip_address = 192.168.100.1<br />        number = 0<br />        name = host1.domain.com<br />        cluster = ocfs2<br />node:<br />        ip_port = 7777<br />        ip_address = 192.168.100.2<br />        number = 1<br />        name = host2.domain.com<br />        cluster = ocfs2<br />node:<br />        ip_port = 7777<br />        ip_address = 192.168.100.3<br />        number = 2<br />        name = host3.domain.com<br />        cluster = ocfs2<br />cluster:<br />        node_count = 3<br />        name = ocfs2<br /></pre>

<p>现在，可以运行 /etc/init.d/o2cb start 来启动集群服务。如果一切都顺利的话，运行 /etc/init.d/o2cb status 可以看到类似这样的结果：</p>

<pre>Module &quot;configfs&quot;: Loaded<br />Filesystem &quot;configfs&quot;: Mounted<br />Module &quot;ocfs2_nodemanager&quot;: Loaded<br />Module &quot;ocfs2_dlm&quot;: Loaded<br />Module &quot;ocfs2_dlmfs&quot;: Loaded<br />Filesystem &quot;ocfs2_dlmfs&quot;: Mounted<br />Checking O2CB cluster ocfs2: Online<br />  Heartbeat dead threshold: 31<br />  Network idle timeout: 30000<br />  Network keepalive delay: 2000<br />  Network reconnect delay: 2000<br />Checking O2CB heartbeat: Active<br /></pre>

<p>现在，你可以格式化你的 OCFS2 分区了。</p>

<pre># mkfs.ocfs2 -b 4k -C 32K -L &quot;label&quot; -N 4 /dev/sdxX<br /></pre>

<p>现在在每台主机上加载你的分区。</p>

<pre># mount -L &quot;label&quot; /dir<br /></pre>

<p>如果一切正常的话，你应该把它添加到你的 fstab 里面，并确保 o2cb 和 ocfs2 服务在系统引导时启动。</p>

<pre>/sbin/chkconfig o2cb on<br />/sbin/chkconfig ocfs2 on<br /></pre>

<p>本文讨论了所有的基础只是，应该足可以搭建一个可用的 OCFS2 了。如果你计划在生产环境中使用 OCFS2，你应该看看 FAQ，其中包含乐更多的高级话题，如更改大小，quorum and fencing, 限制，以及 rolling upgrades。</p>

<p><a href="http://www.linux-mag.com/author/55"></a>Jeremy Garcia 是 LinuxQuestions 的创始人和管理员，那是一个使用 SpamAssassin 过滤邮件的自由、友善、活跃的 Linux 社区。请把问题和反馈发送到 jeremy@linuxquestions.org。</p>