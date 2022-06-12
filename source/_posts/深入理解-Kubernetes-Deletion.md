---
title: 深入理解 Kubernetes Deletion
tags: [kubernetes]
copyright: true
date: 2021-07-21 12:08:12
permalink:
categories: golang
description: 深入理解 kubernetes Deletion
image: https://static001.geekbang.org/resource/image/01/71/01df65e1f4d3047fb6cc02cdf5db3a71.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->
`kubectl delete` 这个命令我们几乎每天都在使用，看起来很容易理解，不就是删除 Kubernetes 的某种资源吗？但是要完全理解 Delete 操作还是很有挑战的，理解删除操作背后真正的原理，能够帮助你从容应对一些奇葩场景。这篇文章将从以下几个方面详细解释删除操作背后的故事：

1. 基本的删除操作
2. Finalizers 和 Owner Reference 会对删除操作产生什么影响
3. 如何利用  propagation policy 改变删除的顺序
4. 通过 examples 演示上述删除操作

简单起见，所有示例都将使用 ConfigMaps 和 Basic Shell 命令来演示操作过程。

### The basic `delete`

Kubernetes 有几种不同的命令，您可以使用它允许您创建，读取，更新和删除对象。 出于本博客文章的目的，我们将专注于四个 kubectl 命令：create`, `get`, `patch`, 和 `delete。

下面是几个最简单的 `kubectl delete ` 使用场景：

```shell
kubectl create configmap mymap
configmap/mymap created
```

```shell
kubectl get configmap/mymap
NAME    DATA   AGE
mymap   0      12s
```

```shell
kubectl delete configmap/mymap
configmap "mymap" deleted
```

```shell
kubectl get configmap/mymap
Error from server (NotFound): configmaps "mymap" not found
```

演示了一个 configMap 从创建、查询、到删除、再查询的过程，这个过程可以用下面的状态图表示：

![state diagra for delete](https://tva1.sinaimg.cn/large/008i3skNgy1gsnil61b40j30fx0710sw.jpg)

这种 `delete` 操作是非常简单直观的，但当你遇到 `Finalizer`和 `Owner References` 的时候就会出现各种难以理解的现象。

### Understanding Finalizers

在理解 Kubernetes 的资源删除时，了解 `Finalizers` 的工作原理能够在你无法删除资源时给你一些解决问题的灵感。

`Finalizers` 是触发 `pre-delete` 操作的关键，能够控制资源的垃圾回收。`Finalizers` 设计的初衷是帮助 controller 在处理资源删除之前，优先处理一些 clean up 的逻辑。但是 `Finalizers` 并不包含代码逻辑，使用上跟 `Annotation` 有些相似，很容易被添加或者删除。

你可以已经见过下面的 `Finalizers`:

```shell
kubernetes.io/pv-protection
kubernetes.io/pvc-protection
```

这两个 `Finalizer` 是用来防止误删除 volume 的，类似功能的 Finalizer 还有很多。

下面是一个自定义的 configmap，只包含一个 `Finalizer`:

```shell
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: mymap
  finalizers:
  - kubernetes
EOF
```

 负责管理 configmap 的 controller 并不知道该如何处理 `kubernetes` 这个 Finalizer。现在我尝试去删除这个 configmap:

```shell
kubectl delete configmap/mymap &
configmap "mymap" deleted
jobs
[1]+  Running kubectl delete configmap/mymap
```

Kubernetes 会返回一个信息告诉你这个 configmap 已经被删除了，然而并不是传统意义上的删除，而是出于一个 `deleting` 的状态。当我们再次执行 `get` 操作去获取该 configmap，会发现 configmap 资源已经被修改过了，deletionTimeStamp 设置了值。

```shell
kubectl get configmap/mymap -o yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: "2020-10-22T21:30:18Z"
  deletionGracePeriodSeconds: 0
  deletionTimestamp: "2020-10-22T21:30:34Z"
  finalizers:
  - kubernetes
  name: mymap
  namespace: default
  resourceVersion: "311456"
  selfLink: /api/v1/namespaces/default/configmaps/mymap
  uid: 93a37fed-23e3-45e8-b6ee-b2521db81638
```

换句话说，这个 configmap 资源并没有被删除而是被更新了，这是因为 Kubernetes 看到资源中有一个 `Finalizer` 后把它置为了 read-only 状态。这个 configmap 只有在移除了 `Finalizer` 后，才会真正从集群中删除。

我们使用 `patch` 命令移除 `Finalizer` 来验证一下上面的说法。

```shell
kubectl patch configmap/mymap \
    --type json \
    --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]'
configmap/mymap patched
[1]+  Done  kubectl delete configmap/mymap
```

此时我们再去 `get` configmap，发现已经获取不到了。

```shell
kubectl get configmap/mymap -o yaml
Error from server (NotFound): configmaps "mymap" not found
```

![state diagram for finalize](https://tva1.sinaimg.cn/large/008i3skNgy1gsnkddcs46j30l10b9t95.jpg)

上面展示了当一个资源带有 `Finalizer` 时，执行删除操作后的状态流转图。

### Owner References

Owner reference 描述了多种资源之间的关联关系。它们是关于资源的属性，可以彼此指定关联关系，因此可以做到级联删除。

Kubernetes 中最常见的具备 owner reference 关系的场景就是 pod 将 replica set 作为自己的 owner，所以当 deployment 或者 statefulSet 删除的时候，作为子资源的 replica set 和 pod 都将被删除。

下面的例子解释了 owner reference 的工作原理。我们首先创建了一个父资源，然后创建子资源的时候指定其 owner reference：

```shell
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: mymap-parent
EOF
CM_UID=$(kubectl get configmap mymap-parent -o jsonpath="{.metadata.uid}")

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: mymap-child
  ownerReferences:
  - apiVersion: v1
    kind: ConfigMap
    name: mymap-parent
    uid: $CM_UID
EOF
```

当删除子资源时，父资源并不会被删除：

```shell
kubectl get configmap
NAME           DATA   AGE
mymap-child    0      12m4s
mymap-parent   0      12m4s

kubectl delete configmap/mymap-child
configmap "mymap-child" deleted

kubectl get configmap
NAME           DATA   AGE
mymap-parent   0      12m10s
```

我们再根据上面的 yaml 建立资源，然后我们删除父资源，这时发现所有的资源都被删除了：

```shell
kubectl get configmap
NAME           DATA   AGE
mymap-child    0      10m2s
mymap-parent   0      10m2s

kubectl delete configmap/mymap-parent
configmap "mymap-parent" deleted

kubectl get configmap
No resources found in default namespace.
```

简而言之，当父 - 子资源之间存在 owner reference 关系的时候，我们删除父资源，子资源也会随之删除，这叫做 `cascade` （级联删除）。默认情况下，`cascade` 的值是 true，你可以执行 `kubectl delete` 的时候加上参数 `--cascade=false` ，这样就可以只删除父资源而保留子资源。

在以下示例中，有一对父子资源，如果我使用 `--cascade = false` 删除父资源，但子资源仍然存在：

```shell
kubectl get configmap
NAME           DATA   AGE
mymap-child    0      13m8s
mymap-parent   0      13m8s

kubectl delete --cascade=false configmap/mymap-parent
configmap "mymap-parent" deleted

kubectl get configmap
NAME          DATA   AGE
mymap-child   0      13m21s
```

`——cascade` 参数能够控制 API 中的删除传播策略（ propagation policy），它允许控制删除对象的顺序。在下面的示例中，使用 API 访问创建一个带有 background 传播策略的自定义 delete API 调用：

```
kubectl proxy --port=8080 &
Starting to serve on 127.0.0.1:8080

curl -X DELETE \
  localhost:8080/api/v1/namespaces/default/configmaps/mymap-parent \
  -d '{ "kind":"DeleteOptions", "apiVersion":"v1", "propagationPolicy":"Background" }' \
  -H "Content-Type: application/json"
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Success",
  "details": { ... }
}
```

要注意，不能使用 kubectl 在命令行上指定传播策略。必须使用自定义 API 调用来指定。需要创建一个代理，这样就可以从客户端访问 API Server，并执行 curl 命令来执行该删除命令。

一共有三种传播策略：

`Foreground`：先删除子资源再删除父资源（post-order）

`Background`：先删除父资源再删除子资源（pre-order）

`Orphan`：忽略 owner reference

要注意，假如一个资源中同时设置了 owner reference 和 finalizer，finalizer 的优先级是最高的。

### Forcing a Deletion of a Namespace

你可以已经遇到过执行 `kubectl delete ns NS` 后，ns 无法删除的情况，这时候你可以通过 update 所删除 ns 的 `Finalizer` 字段来实行强制删除。这个操作会通知到 namespace controller 移除 finalizer ：

```shell
cat <<EOF | curl -X PUT \
  localhost:8080/api/v1/namespaces/test/finalize \
  -H "Content-Type: application/json" \
  --data-binary @-
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "test"
  },
  "spec": {
    "finalizers": null
  }
}
EOF

```

这个操作要谨慎，因为它可能只删除名称空间，而将孤立资源留在现在不存在的名称空间中，会让集群处于让人很迷惑的状态。如果发生这种情况，可以手动重新创建该名称空间，有孤立的资源对象将重新出现在刚刚创建的名称空间下，可以手动清理和恢复。

### Key Takeaways

上面的例子说明，`Finalizer` 能够阻止 Kubernetes 删除资源，但是通常在代码中添加 `Finalizer` 是有原因的，因此应该在手动删除它之前进行检查。Owner reference 支持级联删除资源，但 `Finalizer` 的优先级更高。最后，可以使用传播策略通过自定义 API 调用指定删除顺序，从而控制对象的删除方式。通过上面的例子，相信大家对 Kubernetes 中的删除工作原理有了更多的了解，现在可以使用测试集群自己尝试一下。

原文地址：https://kubernetes.io/blog/2021/05/14/using-finalizers-to-control-deletion/



<hr />
