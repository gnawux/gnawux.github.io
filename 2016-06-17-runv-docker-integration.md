title: "RunV: 让 Docker 支持虚拟化容器"
date: 2016-06-17 15:06:08 +0800
update:
author: me
cover: 
categories:
    - hyper
tags:
    - hyper
    - announcement
    - runv
    - docker
    - containerd
    - container
    - oci
    - work
preview: 在此，我们很高兴地告知各位，我们已经初步完成了 runV (OCI 的基于虚拟化技术的容器运行时引擎) 与 Docker 的集成。这里，我们感谢 runV 团队的优秀工作，而且这次更新的一个特别之处在于，这些更新是由 Hyper 的开发团队和来自社区的参与者共同完成的，他们也是 runV 社区的重要部分，并在此接受致谢。
draft: false

---

在此，我们很高兴地告知各位，我们已经初步完成了 runV (OCI 的基于虚拟化技术的容器运行时引擎) 与 Docker 的集成。这里，我们感谢 runV 团队的优秀工作，而且这次更新的一个特别之处在于，这些更新是由 Hyper 的开发团队和来自社区的参与者共同完成的，他们也是 runV 社区的重要部分，并在此接受致谢。

从去年夏末 OCI （开放容器促进组织）在 Linux 基金会下成立的时候起，Hyper 的 runV 就成为了 [OCI 官方的基于虚拟化技术的容器运行时引擎实现[1]](https://github.com/opencontainers/runtime-spec/blob/master/implementations.md)，而另一个基于容器的实现就是 Docker 的 runC。很久以来，大家都期盼着能用 Docker 命令行同时启动 runC 和 runV 容器，不过，由于 Docker 的执行引擎在按照他们的节奏向前推进，这个愿望一直未能实现。

今年四月，[docker 发布了 1.11，集成了 containerd，连接了 runC [2]]([https://blog.docker.com/2016/04/docker-engine-1-11-runc/)，这为支持更多的 runtime 铺就了道路，于是，runV 与 Docker/Containerd 的集成就再次提上了台面。

作为基于 hypervisor 的 OCI runtime，Hyper 的 runV 与 Docker 1.11+/Containerd 的集成工作是比较容易的，[经过简单的调试，就可以让 Docker 和 containerd 直接对接到 runv 上[3]](https://github.com/hyperhq/runv#run-it-with-docker)。但由于 containerd 是为 runC 量身定制的， runC本身也是一个不断改进中的实现，命令行会不断变化，并且包含一些专有的特性，这个集成有不少局限，对于 tty, exec, 网络等方面的支持仍有不足。

不过，得益于 docker/containerd 提供的良好接口，我们给出了一个对 tty, exec, 网络等方面兼容性更好的过渡方案 —— 我们在 runv 中附带了 [runv-containerd 程序 [4]]([https://github.com/hyperhq/runv/blob/master/containerd/README.md)，基于 containerd并针对 hypervisor 进行了一些调整， 利用 runV ，现在 docker 可以直接创建功能齐备的虚拟化容器了。

````
# in terminal #1
runv-containerd --debug --driver libvirt --kernel /opt/hyperstart/build/kernel --initrd /opt/hyperstart/build/hyper-initrd.img
# in terminal #2
docker daemon -D -l debug --containerd=/run/runv-containerd/containerd.sock
# in terminal #3 for trying it
docker run -ti busybox
# ls   # (already in the terminal of the busybox container)
# exit # (quit the container)
````

未来随着 OCI/containerd/runV 的进一步发展，相信我们还可以做到更好更完美的集成。

[1] https://github.com/opencontainers/runtime-spec/blob/master/implementations.md

[2] https://blog.docker.com/2016/04/docker-engine-1-11-runc/

[3] https://github.com/hyperhq/runv#run-it-with-docker

[4] https://github.com/hyperhq/runv/blob/master/containerd/README.md


 
