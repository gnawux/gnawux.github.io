title: "Blog 迁移"
date: 2016-09-24 11:08:08 +0800
update:
author: me
cover:
categories:
    - meta
tags:
    - meta
    - hyper
    - caddy
    - blog
    - letsencrypt
preview: 终于让自己的 Blogs 吃上了自己的狗粮
draft: false

---

## 由来已久的念头

自从我把 [Blog 迁移到 Github (迁移声明)](/meta/2015/02/24/blog-update/index.html)，生活重新变得平静了，不用管 Wordpress 的升级了，不用管主机的事情了，什么都不用操心，DNS 指到 Github 就好了……

自从我用上了我司优秀青年严松的 [ink](https://github.com/InkProject/ink) 之后， blog 文章写作和发布也变得更简单通透了——如果有啥不满意就改改 blog 引擎，给作者发 PR，作为 CTO 强迫他[接受 PR](https://github.com/InkProject/ink/pulls?q=is%3Apr+author%3Agnawux)。

然而，还是有一点点不爽——放在 Github 的 blog，是很难搞 SSL 的，即使是如今有 [Let's Encrypt](https://letsencrypt.org/) 这样的免费服务，仍然无法方便利用……

随着，我家自己的 [Hyper_](https://hyper.sh) 上线，容器部署超简单，三秒上线，可以用小尺寸容器、独立 Floating IP，完美配合，于是，一直想着把自己和儿子的 Blog 迁移到 Hyper_ 上来。

## 更多需求

至于一直为啥没迁移，实际上我有点想做的事情——

- Github 自动触发更新：我的 blog 现在都是 github page 了，我想更新 Github 自动生效，无需手动部署
- 简单的 image，不想弄复杂的程序和脚本
- 配置 TLS：一直没弄明白 Let's Encrypt

尤其是自动更新，乃至我都想自己写个 server—……

## 终于找到了

Hyper_ 正式上线一个月，我发现好多用户都在用一个叫 [Caddy](https://caddyserver.com/) 的东西，于是就问了下同事，说是新一代的支持 HTTP/2 的 Server。想着是不是可能有我的需求，或者，反正是 Go 写的，不行没准能改改呢，于是就去看了一眼。

哈，我找到了什么 —— https://caddyserver.com/docs/git ——我最主要的需求居然已经被满足了，我可以这么配置

```
wangsiyi.net {
        gzip
        root wangsiyi.net
        git {
                repo https://github.com/gnawux/wangsiyi.net
                branch gh-pages
                hook /somehandle mypassword
                hook_type github
                interval -1
        }
}
wangxu.me {
        gzip
        root wangxu.me
        git {
                repo https://github.com/gnawux/gnawux.github.io
                hook /somehandle mypassword
                hook_type github
                interval -1
        }
}
```

就这么简单，起服务，自动拉 repo；然后，在 Github 配上 webhook，有 push 的时候，自动更新 webroot 的内容……

（此处切换 DNS A 记录，并等待更新完毕——这个传说会很久，不过如今实际上还是挺快的）

于是，在 hyper 上开了一个 server （绑上 fip）来试下

```
hyper run -d --name blogs -p 80:80 -p 443:443 ubuntu
hyper fip attach 2xx.xx.xx.xx blogs
```

用 `hyper exec -it blogs` 登上去，装上 `wget`, `git`，拉个有 `git` 模块的 Caddy

```
wget -O caddy.tgz "https://caddyserver.com/download/build?os=linux&arch=amd64&features=git"
```

解开之后，放上刚才的 `Caddyfile`，运行 Caddy。 我看到了什么 —— 居然还啥都没做，就自动配好了 Let's Encrypt 的 CA，现在，我的网站已经支持 SSL 了，Github hook 的配置也完全无痛，设个链接和密码就直接工作了，完全不需要调试。

好吧，有点简单得超出我的预期，嗯，过两天再搞自己的 image 吧，本来就想试试的，没想到就 piu 地一下就切好了，真是太出乎预料了，欢迎大家访问：

- 我的 Blog（就是本站）: https://wangxu.me
- 我家儿子的 Blog（有一段没更新了，很快更新）: https://wangsiyi.net

以及，我闺女的 Blog （ Coming soon...）
