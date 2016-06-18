date: 2013-06-01 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: 在新 Thinkpad S3 上使用 bcache 作为 Linux 的根分区
date: 2013-06-01 15:57:55 +0800
categories:
- linux
tags:
- bcache
- initramfs
- linux
- scripts
- ubuntu
status: publish
type: post
published: true
---
<p>前两天采购了一台“全球首发"的 <a title="京东链接" href="http://item.jd.com/894057.html" target="_blank">Thinkpad S3 超级本</a>，激动之余，就把不会用的 Windows 8 给抹掉，装上 Linux 了。这台机器的配置如下</p>

<blockquote><p><span style="font-family: courier new,courier;">CPU i5-3337U<br />
内存 8GB<br />
芯片组 Intel HM77<br />
显卡 CPU集成 + </span>Radeon HD 8670M<br />
硬盘 500GB SATA + 24GB mSATA SSD</p></blockquote>

<p>嗯，其他先不说了，亮点基本就这些。新机器，一大堆支持得不好的东西，比如 suspend to ram 我到现在还没配好，先不说了，这次主要说硬盘。</p>

<h2>基本思路</h2>

<p>这台机器有一块过小的 SSD 和一块主硬盘，因为SSD太小了，以至于我不想把哪个分区整体放到上面，于是，想到了分级存储的策略——使用SSD作为缓存，让热数据缓存在SSD上，冷数据留在机械硬盘上，是的，这个和被广泛宣传的 Apple 的 Fusion Drive 很类似，当然，Apple 不是第一个开发这种技术的公司。</p>

<p>在 Linux 上，最广为传颂的这类技术就是<a title="Bcache Homepage" href="http://bcache.evilpiepirate.org" target="_blank"> bcache </a>了，但有一个问题，bcache 直到 3.10 才进入 kernel，而 3.10 到目前为止还只是 -rc3。也就是说，如果要装 Debian、Ubuntu、Fedora 的话，你至少要等半年，才能找到一张支持 bcache 的启动盘。不过，我还是想用下这个个 Distro，于是，就采用了这样一个思路（思路抄袭自<a href="https://wiki.archlinux.org/index.php/Bcache" target="_blank">这个 Arch 的 Wikipage</a>，有调整）:</p>

<ol>

<li>先在 SSD 上装一个临时的系统</li>

<li>升级内核，支持 bcache</li>

<li>在主硬盘上做一个分区，作为 bcache 的后端存储</li>

<li>把装好的系统迁移到上面的 bcache 分区</li>

<li>修改 grub 重新启动，引导到 bcache 上去</li>

<li>干掉 SSD 上的分区，作为 bcache 的缓存</li>

</ol>

<p>整体过程就是这样，中间波折无数，下面就介绍一下。具体的折腾过程中，有多次反复操作（不过装系统只有一次，我的系统观是——重装就是认输，只要有一丝希望，就反复折腾，拒绝重装，不抛弃、不放弃），后面的介绍省掉了一些无畏的折腾，尽量直接的介绍，不过各位重试的时候有什么我没写的问题，可以和我交流一下哈，没准我也遇到过了。</p>

<h2>引导、初次安装、定制内核</h2>

<p>装系统之前，我手头没有方便可用的 X86_64 架构的 Linux，只有一台 MBP，所以，有些东西比较费劲。启动盘的制作是在 MBP 上进行的，下载这个镜像</p>

<blockquote>

<pre>ubuntu-13.04-desktop-amd64.iso</pre>

</blockquote>

<p>不要那个 +mac 的，这样才能 UEFI 模式启动，在 Mac 下制作启动U盘的方法其实不困难：</p>

<pre>&lt;code&gt;hdiutil convert ubuntu-13.04-desktop-amd64.iso -format UDRW -o ubuntu-1304.img
dd if=ubuntu-1304.img.dmg of=/dev/disk2 bs=1m
diskutil ej&lt;/code&gt;ect /dev/disk2</pre>

<p>简单地说就是，先从光盘变成镜像，然后再把镜像 dd 到 U 盘去，嗯，各位用的时候小心点顺序 disk2 是我的 U 盘，我的 Mac 物理上有 SSD 和 SATA，所以 U 盘是 disk2 。</p>

<p>这个镜像制作的 U 盘是可以在超级本上 UEFI 引导的，能 UEFI 引导非常重要，因为这样才能安装支持 EFI 的 grub，我一开始没注意这点，后来又修复了半天引导程序。</p>

<p>嗯，另外，分区的时候，要留出单独的 boot 分区，毕竟 grub 是不认 bcache 的。原有的 EFI 分区啥的也没有动，我把 grub 也做在 HDD 上了。</p>

<p>然后是定制内核，本来我注意到 Ubuntu PPA 里面有主线内核的 Builds，于是就下了 3.10.0-rc3 的 deb，启动之后发现没有 bcache，拿来 config 一看，居然没选上 bcache，摔。只好自己下 kernel 来 build。为了图省事，直接拿了 ubuntu 的 config，只多选上了 bcache （在 device drivers --&gt; md 里面，和 raid 之类的在一起 ），不过，话说时间是守恒的，你在 config 的时候偷懒没做精简，就在 build 的时候遇到麻烦…… 那些没用的杂七杂八模块好废时间啊，build 了一个小时…… 下次一定好好精简。</p>

<p>顺便说一句，在 debian/ubuntu 下 build kernel，最好安装 kernel-package 包，然后用 fakeroot make-kpkg 来制作 kernel deb 包，这样比较好管理。我的新 kernel 看起来是这样的</p>

<pre>&lt;code&gt;gnawux@square:~$ ls src/*.deb
src/linux-image-3.10.0-rc3-bcache_3.10.0-rc3-bcache-10.00.Custom_amd64.deb
gnawux@square:~$ uname -a
Linux square 3.10.0-rc3-bcache #1 SMP Wed May 29 11:28:22 CST 2013 x86_64 x86_64 x86_64 GNU/Linux
&lt;/code&gt;</pre>

<p>-bcache 是我自己加的后缀，嗯，接下来重启到新内核，就支持 bcache 了</p>

<h2>启用 bcache，引导系统</h2>

<p>现在重启之后，系统已经支持 bcache 了。需要做三件事情</p>

<ol>

<li>打开 bcache</li>

<li>把系统迁移上去</li>

<li>修改 grub，引导新系统</li>

</ol>

<p>打开 bcache 之前，最好准备一下用户态工具</p>

<blockquote><p>http://evilpiepirate.org/git/linux-bcache.git</p></blockquote>

<p>git下来，然后 <code>make install</code> 就可以了，需要 <code>uuid-dev</code> 包。用户态工具主要包含了 udev 工具链上的规则，并且会加入到 initramfs 里面去，还包含了制作 bcache 分区的工具。</p>

<p>下面要制作 bcache 后端分区，后端就是机械硬盘上的大分区</p>

<pre>&lt;code&gt;make-bcache -B /dev/sda8
&lt;/code&gt;</pre>

<p>就酱紫就可以了，然后告诉 kernel 这货可以用了</p>

<pre>&lt;code&gt;echo /dev/sda8 &amp;gt; /sys/fs/bcache/register
&lt;/code&gt;</pre>

<p>不过我记得有了 udev rule，这个实际不用 echo 了，当然，多来一次没啥坏影响。</p>

<p>之后，在 /dev/bcache0 上 makefs，然后迁移系统。我是懒得再用 bootstrap 方式重装一个系统了，我选择直接复制，大概过程是这样的：</p>

<pre>&lt;code&gt;mount -o bind / /mnt/orig
mount /dev/bcache0 /mnt/new
cd /mnt/orig
tar -cvpf - * | tar -C /mnt/new/ -xf -
&lt;/code&gt;</pre>

<p>然后修改新系统的 fstab、修改 grub.cfg 的 kernel 命令行，让 root 指向 bcache0，然后重启就可以成功了。</p>

<p>成功引导之后，把 SSD 上的老的系统做掉，变成 bcache 的 cache 分区</p>

<pre>&lt;code&gt;make-bcache -C /dev/sdb1
echo /dev/sdb1 &amp;gt; /sys/fs/bcache/register
&lt;/code&gt;</pre>

<p>这时，/sys/fs/bcache/ 下就会出现这个 cache 分区的 UUID，比如</p>

<pre>&lt;code&gt;root@square:~# ls /sys/fs/bcache/
6a6b9acd-e205-4a10-a64f-6c048841c824  register  register_quiet
&lt;/code&gt;</pre>

<p>现在，让这个 cache 服务于 bcache0 ，这既可以在 bcache0 上操作，也可以在后端设备 sda8 (我的分区号是这个)上操作，比如</p>

<pre>&lt;code&gt;echo 6a6b9acd-e205-4a10-a64f-6c048841c824 &amp;gt; /sys/block/bcache0/bcache/attach
&lt;/code&gt;</pre>

<p>或者</p>

<pre>&lt;code&gt;echo 6a6b9acd-e205-4a10-a64f-6c048841c824 &amp;gt; /sys/block/sda/sda8/bcache/attach
&lt;/code&gt;</pre>

<p>之所以给出后面的方法，是因为如果由于没挂上 cache 之类的问题，bcache0 没有出现，仍然可以用后面的操作。现在再来，就已经完全打开 bcache 了。</p>

<h2>修复关机和开机的问题</h2>

<p>完全打开 bcache 之后，发现关机不能……开机没问题，但关机按电源是无法接受的啊，因为这之后有 IOError，判断应该是 driver 的某个问题导致导致的，比如和 bcache 在关机过程中的操作顺序有关系，研究之后，发现在关机的时候 detach 掉 cache 分区就可以正常关机了，编辑 <code>/etc/init.d/umountroot</code> 文件，在 ro remount 跟分区之前，加入一段</p>

<pre class="brush: bash; gutter: true">    while [ -e /sys/block/bcache0/bcache/cache ]
    do
            echo &quot;detach bcache&quot;
            echo &quot;6a6b9acd-e205-4a10-a64f-6c048841c824&quot; &gt; /sys/block/bcache0/bcache/detach
            sleep 2
    done</pre>

<p>这段的意思是，只要 cache 还在，或者说还没有完全把 cache 刷入磁盘，就一直 detach，这样，关机就正常了。</p>

<p>关机正常之后，开机进不了根分区了，似乎是只要关机前 detach 掉 cache，开机的时候 bcache0 就不会正常产生了，因为我们没有支持 bcache 的启动盘，所以必须在没有系统的情况下就地解决这个问题。这时，在 initramfs 的界面里，输入上面那句：</p>

<pre>&lt;code&gt;echo 6a6b9acd-e205-4a10-a64f-6c048841c824 &amp;gt; /sys/block/sda/sda8/bcache/attach
&lt;/code&gt;</pre>

<p>然后，ctrl-d 退出 busybox，就可以继续正常启动了，下面的操作显然是要对 initramfs 下手了。对于 ubuntu 的 <code>initramfs-tools(8)</code> 来说，启动脚本位于 <code>/etc/initramfs-tools/scripts/</code> 下面，其中，<code>local-top/</code> 目录中的脚本是在本地文件系统设备出现之前执行的，我们要在这里确保根分区设备 <code>/dev/bcache0</code> 就位，最后我是使用了这样一个脚本：</p>

<pre class="brush: bash; gutter: true">root@square:~# cat /etc/initramfs-tools/scripts/local-top/bcache
PREREQ=&quot;udev&quot;
prereqs()
{
    echo &quot;$PREREQ&quot;
}
case $1 in
# get pre-requisites
prereqs)
    prereqs
    exit 0
    ;;
esac

#. /scripts/functions

timeout=16
while [ ! -d /sys/block/sda/sda8/bcache ] ;
do
   if [ $timeout -eq 0 ] ; then
    break
   fi
   echo  &quot;$((timeout=timeout-1))&quot; &gt; /dev/null
   sleep 1
done

[ ! -e /sys/block/sda/sda8/bcache/cache ] &amp;&amp; echo 6a6b9acd-e205-4a10-a64f-6c048841c824 &gt; /sys/block/sda/sda8/bcache/attach</pre>

<p>原理就是等待 bcache 设备出现，然后把 cache 给它 attach 上，然后用 update-initramfs 重新生成 initrd 就行了。</p>

<h2>小结和一些有用的链接</h2>

<p>总结起来就是——别怕出错，一直往下做，不管咋的，都会有办法的，计算机就在手上，电源都可以按，还怕啥。出了问题要冷静，找到问题，捅一下可能就好了，不行就多试几次。</p>

<p>给一个参考链接吧，用来修复Ubuntu的引导程序的，有时候有用，嗯</p>

<ul>

<li>https://help.ubuntu.com/community/Boot-Repair</li>

</ul>
