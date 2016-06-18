date: 2010-04-08 08:08:08 +0800
update: ""
author: me
topic: ""
draft: false
top: false
title: CodeReview统计脚本
date: 2010-04-08 11:37:06 +0800
categories: []
tags: []
status: draft
type: post
published: false
---
<p><code><br />
grep -n -R '//\*CR\*' * |sed -ne 's/\([^:]*\)\/\([^.:]\+\.java\):\([0-9]\+\):\(.*\)\/\/\*CR\*\(.*\)$/@line \3 of \2 \n\tin Path: \.\/\1\n\t--&gt; \4\n\t\"Issue: \5/p'<br />

</code></p>
