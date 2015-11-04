---
layout: post
title: "Kubernetes 中的 Service Account"
description: "Service Account 是 Kubernetes 用于集群内运行的程序，进行服务发现时调用 API 的帐号"
category: cloud
tags: ["cloud","hyper","kubernetes","hypernetes","service discovery"]
---
{% include JB/setup %}

> 照例广告放最前：Hyper 作为一家新的、小的、有梦想的、不做“Copy-to-China”的创业公司，我们欢迎想创新的、有想法的、有技术的小伙伴加盟，并且我们允许远程办公。

Kubernetes 中的 *Service Account* 是个比较难以理解的概念，什么是 Service Account，到底是做什么的？[文档](http://kubernetes.io/v1.0/docs/user-guide/service-accounts.html)中如是说——

> A service account provides an identity for processes that run in a Pod. （服务帐号为 Pod 中的进程提供了一个 id）

不过，这个 Pod 中的进程的 id 是做什么用的，在用不到的时候还真让人费脑筋。[管理文档](http://kubernetes.io/v1.0/docs/admin/service-accounts-admin.html)里说得更详细一点，但是仍然没有提到是做什么用的。

在这种时候，一个好的例子往往胜过文档的解释。实际上，kubernetes 的官方示例里就有 Service Account 的应用。仔细看这个例子——[在 Kubernetes 中运行 Cassandra](https://github.com/kubernetes/kubernetes/blob/release-1.0/examples/cassandra/README.md)。

大家知道，在 Cassandra 这种全对称结构的集群里，最先启动的种子节点是最重要的，其他节点都要加入到种子节点的集群中才能保证启动的是一个集群而不会分裂成多个集群，Cassandra、Akka 集群都有这个要求。

然而，在这个例子中，是先启动一个节点，然后直接提高 Replica 数量，来做到多节点的，后面的节点是怎么找到种子节点的呢？仔细看例子的文档——

>However it also adds a custom SeedProvider to Cassandra. In Cassandra, a `SeedProvider` bootstraps the gossip protocol that Cassandra uses to find other nodes. The `KubernetesSeedProvider` discovers the Kubernetes API Server using the built in Kubernetes discovery service, and then uses the Kubernetes API to find new nodes

这里提到了，image 中的 Cassandra 有一个特殊的 `KubernetesSeedProvider` ，由它调用 Kubernetes 的 API 来获得集群中已经存在的节点的。注意，这里就是在 Pod 中运行的进程调用 Kubernetes API 的地方，也就是 Service Account 工作的地方。

代码之前，了无秘密，看[这段代码](https://github.com/kubernetes/kubernetes/blob/release-1.0/examples/cassandra/java/src/io/k8s/cassandra/KubernetesSeedProvider.java#L101)

<pre>
    public List<InetAddress> getSeeds() {
        List<InetAddress> list = new ArrayList<InetAddress>();
        String host = "https://kubernetes.default.cluster.local";
        String serviceName = getEnvOrDefault("CASSANDRA_SERVICE", "cassandra");
        String podNamespace = getEnvOrDefault("POD_NAMESPACE", "default");
        String path = String.format("/api/v1/namespaces/%s/endpoints/", podNamespace);
        try {
            String token = getServiceAccountToken();
</pre>

这里，给出了访问的 endpoints API，并且要从本地取出 Service Account 的 Token，来获得服务发现的信息。

综上，Service Account 是 Kubernetes 用于集群内运行的程序，进行服务发现时调用 API 的帐号，帐号的 token 会直接挂载到 Pod 中，可以供程序直接使用。
