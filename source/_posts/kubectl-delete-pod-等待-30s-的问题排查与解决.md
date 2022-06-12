---
title: kubectl delete pod 等待 30s 的问题排查与解决
tags: [docker,kubernetes]
copyright: true
date: 2020-11-23 13:56:03
permalink:
categories: docker
description: kubectl delete pod 等待 30s 的问题排查与解决
image: https://static001.geekbang.org/resource/image/0f/76/0f0963bc50f97aa1171fa81ae6e96776.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->
我们平时在使用 Kubernetes 的时候，一般会使用 `kubectl delete pod podName` 的方式删除一个容器。 但我们发现，每次执行 `kubectl delete pod` 之后都要等待 30s kubectl 才会返回。也许你会觉得 30s 是一个可以忍受的时间， 只要最终能删掉 pod 就行，但这样的问题真的只有 30s 这么简单吗？ 为什么不能很快关闭容器呢？ 或者为什么恰好就是 30s 呢？下面就让我们从根源弄清楚这件事情并解决它。

### kubectl delete 如何关闭并删除容器？
`kubectl delete` 命令是在对容器发出停止命令时使用的，从发送信号上来讲，它将发送 `SIGTERM` 信号给容器，通知其结束运行。

> SIGINT 一般用于关闭前台进程，SIGTERM 会要求进程自己正常退出。

当我们在 shell 中给进程发送 SIGTERM 和 SIGINT 信号的时候，这些进程往往都能正确的处理。 但是在容器中却不生效, 这是因为在 docker 中，只会将 SIGTERM 等所有的 signal 信号发送给 PID 为 1 的进程，当我们 docker 中运行的进程的 `PID` 不是 1 时，就不会收到这样的信号。

### 那么为什么是等待 30s ?

![](https://tva1.sinaimg.cn/large/0081Kckwgy1gkz4dhe673j31eg078jt2.jpg)

上图是官方文档的一段截图，Kubernetes 的 pod 中有一个参数 `terminationGracePeriodSeconds`，此时，k8s 等待指定的时间称为优雅终止宽限期。默认情况下，这个值是 30s。值得注意的是，这个过程中 preStop Hook 和 SIGTERM 信号并行发生。Kubernetes 不会等待 preStop Hook 完成。
这意味着如果你的应用程序完成关闭并在 terminationGracePeriod 完成之前退出，Kubernetes 会立即进入下一步。如果你的 Pod 通常需要超过 30s 才能关闭，那么必须增加这个参数的大小，可以通过在 pod 模板中设置 terminationGracePeriodSeconds 来实现。

如果达到上面的时间限制，k8s 将会通过给内核发送 `SIGKILL` 从而强制结束容器。
这就解释了为什么每次 `delete pod` 都要等待 30s，根本原因还是容器中的 java 进程的 PID 不是 1，所以该进程不会理解收到 `SIGTERM`，在等待 `default terminationGracePeriodSeconds` 的时长后被强制结束。

### 强制关闭容器的后果是什么？

现在我们已经了解了 `kubectl delete pod` 等待 30s 的原因了。 我们再来看看另一个问题： 强制关闭容器，真的就没问题吗？

或许你能想到，很多进程在结束阶段会做一些清理工作：比如删除临时目录、执行 shutdown hook 等。 但是当进程被强制关闭时，这些任务就不会被执行，那么我们就可能得到一些并不期望的结果。

以 Eureka 为例，Eureka client 在结束进程时，需要向 Eureka server 发送 shutdown 信号，以注销 client。 这本来没什么问题，因为 Eureka server 即使没有收到这样的信息，也会定期清理 client 信息。 但是 Eureka server 还有一个 self preservation 模式，以防止意外的网络事件导致大量的 client 下线。 这就有可能导致 Eureka 集群的注册表中出现大量的 client 信息，但它们其实已经关闭了。

### 那么如何优雅地关闭容器?
通过上面的分析我们不难找出解决这个问题的方法，就是让容器中的启动的进程 PID 为 1。
先看下之前的容器中 java 进程的 PID，确实不是 1， 是 8：

![](https://tva1.sinaimg.cn/large/0081Kckwgy1gkz4jdcsdyj314y0560tz.jpg)

可以看到 PID 为 1 的是启动 java 进程的 shell 脚本。

这就要追溯到 Dockerfile 的 `ENTRYPOINT` 的两种写法，即 exec 和 shell，两者的区别在于：

- exec 形式的命令会使用 PID 1 的进程；

- shell 形式的命令会被执行为 /bin/sh -c <command>，不会执行在 PID 1 上，也就不会收到 signal。

我们检查了下启动 java 进程的脚本，发现确实是用 `exec` 的方式启动的，那为什么 java 进程的 PID 还不是 1 呢？

原因是 `exec` 形式的 `ENTRYPOINT` 只能解决 `无需任何准备工作就启动进程` 的场景，而不能解决一些需要准备工作的复杂场景。举个栗子，我们的 ENTRYPOINT 往往需要执行一个 shell 脚本，然后在脚本的最后才会去执行 `java -jar xxx`，这时候，我们的 java 进程就无法成为 PID 1 进程。

我们可以在 shell 中使用 exec 命令来解决。这个命令的作用就是使用新的进程替代原有的进程，并保持 PID 不变。 这就意味着我们可以在执行 java 命令的时候使用它，从而替换掉 PID 1 的 shell 脚本。

entrypoint.sh demo
```shell
#!/bin/sh
echo "prepare..."
exec java -jar app.jar
```
修改后我们再来看下容器中的 java 进程的 PID：

![](https://tva1.sinaimg.cn/large/0081Kckwgy1gkz5cr2pqrj313a052t9w.jpg)

使用 exec 之后，容器中 java 进程的 PID 成为 1，我们发出 delete pod 请求能让进程接收到 SIGTERM 信号，执行相应的操作后立即退出。

### 篇外：docker 是如何创建容器的 PID 为 1 的进程?
当我们执行 `docker run -it busybox /bin/sh` 的时候，对于操作系统来讲是个进程, 操作系统会分配一个 PID 给这个进程, 比如这个进程号是 PID=8, 对于操作系统全局来讲它是 PID=8 的一个进程, 但是我们进入容器执行命令ps, 会发现 `/bin/sh` 的 PID=1。
这种就是 docker 的 namespace 机制, 对于全局来讲, 这条 docker 命令的的 PID 可能是 8, 但是对于容器内部来讲 , 它构建了一个假的命名空间, 使 `/bin/sh` 的 PID 进程等于 1。

其中用到的技术就是Linux的创建线程的 system call, 对应的系统调用函数就是 `clone()`。 

```
int pid = clone(main_function, stack_size, SIGCHLD, NULL);
```
这个系统调用创建一个新的进程，并且返回它的 PID，而当我们用 `clone()` 系统调用创建一个新进程时，就可以在参数中指定 `CLONE_NEWPID` 参数：

```
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL);
```
这时，新创建的这个进程将会看到一个全新的进程空间，在这个进程空间里，它的 PID 是 1。但是在宿主机真实的进程空间里，这个进程的 PID 还是系统的数值 8。

以上。

<hr />
