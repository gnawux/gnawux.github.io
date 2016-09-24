title: "Blog 迁移后记"
date: 2016-09-24 16:52:58 +0800
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
    - docker
    - image
preview: 吃自己的狗粮要吃得更正宗
draft: false

---

[前一偏贴出来](/meta/2016/09/24/blog-to-hyper/)有人评论

> 启动一个ubuntu的image, 然后exec进去安装软件。。。这不符合imutable server哲学

本来因为太简单，已经懒得再做 image 了，不过，既然有同学指出了，好吧，你们说得对，吃自己的狗粮要吃得正宗才对，于是，做个 image 吧。

## 制作 Image

`Dockerfile` 这么写

```
FROM ubuntu:xenial
MAINTAINER gnawux@gmail.com
RUN apt-get update; apt-get -y install git
COPY caddy /srv/
COPY entrypoint.sh /
ENTRYPOINT [ "/entrypoint.sh" ]
```

其中的 `entrypoint.sh` 这么写，这样就不用把密码放到 image 里了，嘻嘻

```
#!/bin/bash
PASSWORD=password
if [ $# -ge 1 ] ; then
	if [ "x$1" != "x${1#/}"  ] ; then
		exec "$@"
	fi
	PASSWORD=$1
fi

cat > srv/Caddyfile <<-ENDFILE
wangsiyi.net {
        gzip
        root wangsiyi.net
        git {
                repo https://github.com/gnawux/wangsiyi.net
                branch gh-pages
                hook /update ${PASSWORD}
                hook_type github
                interval -1
        }
}
wangxu.me {
        gzip
        root wangxu.me
        git {
                repo https://github.com/gnawux/gnawux.github.io
                hook /update ${PASSWORD}
                hook_type github
                interval -1
        }
}
ENDFILE

cd srv
./caddy -agree -email gnawux@gmail.com -host wangxu.me > /dev/stdout 2>/dev/stderr
```

上面注意， `entrypoint.sh` 脚本一定要 block 住，不要退出，刚才我一不小心…… 发现怎么都秒退，这个错误太初级了啊，留在 commit log 里实在是比较丢人。

然后，整个制作 image 也可以放在一个脚本里面，比如这样

```
#!/bin/bash

if [ ! -f caddy ] ; then
	wget -O - "https://caddyserver.com/download/build?os=linux&arch=amd64&features=git" | tar -zxvf -
	rm -rf init *.txt
fi

[ ! -f caddy ] && echo "caddy download failed" && exit 1

sudo docker build -t "gnawux/blog:latest" .
```

所有这些我放到 repo 里了： https://github.com/gnawux/xu-site ，本来弄个 repo 我想自己写个 server 的，最后就放个 dockerfile，我也是挺失落的啊。

## 运行

镜像制作好之后，推到 hub，就回到 hyper 来 run 吧

启动 ——

```
➜ ~ hyper run -p 80:80 -p 443:443 --name blog --hostname blog -it gnawux/blog xxxxxxx
Activating privacy features...2016/09/24 08:32:08 get directory at 'https://acme-v01.api.letsencrypt.org/directory': failed to get "https://acme-v01.api.letsencrypt.org/directory": Get https://acme-v01.api.letsencrypt.org/directory: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

擦，访问不了，忘了绑 fip 了 ——

```
➜ ~ hyper fip attach 209.177.91.195 blog
➜ ~ hyper start -a -i blog
Activating privacy features... done.
Cloning into 'wangsiyi.net'...
remote: Counting objects: 927, done.
remote: Total 927 (delta 0), reused 0 (delta 0), pack-reused 927
Receiving objects: 100% (927/927), 14.32 MiB | 4.47 MiB/s, done.
Resolving deltas: 100% (414/414), done.
Checking connectivity... done.
2016/09/24 08:35:22 https://github.com/gnawux/wangsiyi.net.git pulled.
Cloning into 'wangxu.me'...
remote: Counting objects: 17080, done.
remote: Compressing objects: 100% (103/103), done.
remote: Total 17080 (delta 78), reused 0 (delta 0), pack-reused 16951
Receiving objects: 100% (17080/17080), 28.90 MiB | 5.77 MiB/s, done.
Resolving deltas: 100% (7924/7924), done.
Checking connectivity... done.
2016/09/24 08:35:28 https://github.com/gnawux/gnawux.github.io.git pulled.
https://wangsiyi.net
https://wangxu.me
http://wangsiyi.net
http://wangxu.me
```

一切 OK，大家好好玩。
