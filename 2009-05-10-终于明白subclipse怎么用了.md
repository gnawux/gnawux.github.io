date: 2009-05-10 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: 终于明白subclipse怎么用了
date: 2009-05-10 00:06:04 +0800
categories: []
tags:
- eclipse
- java
- subclipse
- subversion
- svn
status: publish
type: post
published: true
---
<p>原来都是svn出来，加到eclipse里面，两个独立操作。前两天想用subclipse，却发现新建的svn project里的东西不是java project的东东，不能被build。</p>

<p>今天终于明白了，subclipse装上之后，svn里的项目要import进来，然后在这个source上建java project而不是直接建svn的project。</p>

<p>呵呵，凑合用了这么久才明白这么简单个东东，请大家笑话我吧，脸红ing</p>
