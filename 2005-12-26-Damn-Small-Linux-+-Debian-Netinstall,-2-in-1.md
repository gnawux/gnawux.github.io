date: 2005-12-26 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: Damn Small Linux + Debian Netinstall, 2 in 1
date: 2005-12-26 15:04:00 +0800
categories:
- works
tags:
- boot
- linux
- works
status: publish
type: post
published: true
---
<p>一张小光盘可以放 200MB，一个 Damn Small Linux 只有 50MB，有点浪费，   <br />young 建议我把两个 iso 合并一下，做成一个。    <br />今天我把 Damn Small Linux 2.1rc2 (49MM) 和 Debian 3.1r0a (Sarge) Net    <br />Install (109M) 合并起来做了一个 ISO，既可以运行 DSL Live CD，也可以    <br />网络安装 Debian，呵呵    <br />主要操作过程：    <br />1. 从两个 iso 中提取数据文件，放在一个目录中，注意 netinstall 中有一个    <br />&#160;&#160; .disk/ 隐藏目录，如果没有无法安装，我吃了这个不小的亏。    <br />2. 建立一个 isolinux 目录，参考原来两张光盘的 isolinux 目录，放入    <br />&#160;&#160; isolinux.bin 和 isolinux.cfg，以及一些说明性信息，boot.cat 不需要;    <br />&#160;&#160; isolinux.bin 对所有光盘都是一样的，不用改动; isolinux.cfg 和 lilo.conf    <br />&#160;&#160; 有点类似，看看就会了，其他文件照猫画虎就行了，放置好 kernel 文件和    <br />&#160;&#160; initrd 文件    <br />3. 重新生成 md5sum.txt，为所有东西做个 md5sum    <br />4. 生成 iso:    <br />&#160;&#160;&#160;&#160;&#160; mkisofs -o twins.iso -b isolinux/isolinux.bin -c isolinux/boot.cat \    <br />&#160;&#160;&#160;&#160;&#160; -no-emul-boot -boot-load-size 4 -boot-info-table -J -r \    <br />&#160;&#160;&#160;&#160;&#160;&#160; -input-charset iso8859-1&#160; twins/    <br />&#160;&#160; 最后的 twins/ 是我生成iso的源文件目录，开始的 twins.iso 是输出的 iso    <br />&#160;&#160; 文件名。    <br />其实主要就是 isolinux 这个引导程序，呵呵，不是很复杂。</p>