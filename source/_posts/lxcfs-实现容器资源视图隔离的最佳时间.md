---
title: lxcfs 实现容器资源视图隔离的最佳实践
tags: [docker]
copyright: true
date: 2020-07-04 20:49:46
permalink:
categories: docker
description: lxcfs 实现容器资源隔离的最佳实践
image: https://static001.geekbang.org/resource/image/f3/b0/f332a055fd82710130f9cccd3ce560b0.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->
>
>
> LXCFS is a small FUSE filesystem written with the intention of making Linux containers feel more like a virtual machine. It started as a side-project of LXC but is useable by any runtime.

用人话解释一下就是：

>
>
> xcfs 是一个开源的 FUSE（用户态文件系统）实现来支持 LXC 容器，它也可以支持 Docker 容器。让容器内的应用在读取内存和 CPU 信息的时候通过 lxcfs 的映射，转到自己的通过对 cgroup 中容器相关定义信息读取的虚拟数据上。

## 	什么是资源视图隔离？

容器技术提供了不同于传统虚拟机技术的环境隔离方式。通常的Linux容器对容器打包和启动进行了加速，但也降低了容器的隔离强度。其中Linux容器最为知名的问题就是资源视图隔离问题。

容器可以通过 cgroup 的方式对资源的使用情况进行限制，包括: 内存，CPU 等。但是需要注意的是，如果容器内的一个进程使用一些常用的监控命令，如: free, top 等命令其实看到还是物理机的数据，而非容器的数据。这是由于容器并没有做到对 `/proc`, `/sys` 等文件系统的资源视图隔离。

## 	为什么要做容器的资源视图隔离？

1. 从容器的视角来看，通常有一些业务开发者已经习惯了在传统的物理机，虚拟机上使用 `top`, ``free` 等命令来查看系统的资源使用情况，但是容器没有做到资源视图隔离，那么在容器里面看到的数据还是物理机的数据。

2. 从应用程序的视角来看，在容器里面运行进程和在物理机虚拟机上运行进程的运行环境是不同的。并且有些应用在容器里面运行进程会存在一些安全隐患:

   对于很多基于 JVM 的 java 程序，应用启动时会根据系统的资源上限来分配 JVM 的堆和栈的大小。而在容器里面运行运行 JAVA 应用由于 JVM 获取的内存数据还是物理机的数据，而容器分配的资源配额又小于 JVM 启动时需要的资源大小，就会导致程序启动不成功。

   对于需要获取 host cpu info 的程序，比如在开发 golang 服务端需要获取 golang中 `runtime.GOMAXPROCS(runtime.NumCPU())` 或者运维在设置服务启动进程数量的时候( 比如 nginx 配置中的 worker_processes auto )，都喜欢通过程序自动判断所在运行环境CPU的数量。但是在容器内的进程总会从`/proc/cpuinfo`中获取到 CPU 的核数，而容器里面的`/proc`文件系统还是物理机的，从而会影响到运行在容器里面服务的运行状态。

   如何做容器的资源视图隔离？

   ## lxcfs 横空出世就是为了解决这个问题。

   lxcfs 是通过文件挂载的方式，把 cgroup 中关于系统的相关信息读取出来，通过 docker 的 volume 挂载给容器内部的 proc 系统。 然后让 docker 内的应用读取 proc 中信息的时候以为就是读取的宿主机的真实的 proc。

   下面是 lxcfs 的工作原理架构图：

   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ggf3n2n48tj318e0msmz5.jpg" alt="lxcfs" style="zoom:50%;" />

   解释一下这张图，当我们把宿主机的 `/var/lib/lxcfs/proc/memoinfo` 文件挂载到 Docker 容器的 `/proc/meminfo` 位置后，容器中进程读取相应文件内容时，lxcfs 的 `/dev/fuse` 实现会从容器对应的 Cgroup 中读取正确的内存限制。从而使得应用获得正确的资源约束。 cpu 的限制原理也是一样的。

   ## 通过 lxcfs 实现资源视图隔离

   ### 安装 lxcfs

   ```bash
   wget <https://copr-be.cloud.fedoraproject.org/results/ganto/lxc3/epel-7-x86_64/01041891-lxcfs/lxcfs-3.1.2-0.2.el7.x86_64.rpm>
   rpm -ivh lxcfs-3.1.2-0.2.el7.x86_64.rpm
   ```

   检查一下安装是否成功

   ```bash
   [root@ifdasdfe2344 system]# lxcfs -h
   Usage:

   lxcfs [-f|-d] [-p pidfile] mountpoint
     -f running foreground by default; -d enable debug output
     Default pidfile is /run/lxcfs.pid
   lxcfs -h
   ```

   ### 启动 lxcfs

   1. 直接后台启动

   ```bash
   lxcfs /var/lib/lxcfs &
   ```

   2. 通过 systemd 启动（推荐）

   ```bash
   touch /usr/lib/systemd/system/lxcfs.service

   cat > /usr/lib/systemd/system/lxcfs.service <<EOF
   [Unit]
   Description=lxcfs

   [Service]
   ExecStart=/usr/bin/lxcfs -f /var/lib/lxcfs
   Restart=on-failure
   #ExecReload=/bin/kill -s SIGHUP $MAINPID

   [Install]
   WantedBy=multi-user.target
   EOF

   systemctl daemon-reload
   systemctl start lxcfs.service
   ```

   检查启动是否成功

   ```bash
   [root@ifdasdfe2344 system]# ps aux | grep lxcfs
   root      3276  0.0  0.0 112708   980 pts/2    S+   15:45   0:00 grep --color=auto lxcfs
   root     18625  0.0  0.0 234628  1296 ?        Ssl  14:16   0:00 /usr/bin/lxcfs -f /var/lib/lxcfs
   ```

   启动成功。

   ## 验证 lxcfs 效果

   ### 未开启 lxcfs

   我们首先在未开启 lxcfs 的机器上运行一个容器，进入到容器中观察 cpu, memory 的信息。为了看出明显的差别，我们用了一台高配置服务器（32c128g）。

   ```bash
   # 执行以下操作
   systemctl stop lxcfs

   docker run -it ubuntu /bin/bash # 进入到 nginx 容器中

   free -h
   ```

   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ggf3p6a8yyj32jy07uk16.jpg" alt="image-20200704180911456" style="zoom:50%;" />

   通过上面的结果我们可以看到虽然是在容器中查看内存信息，但是显示的还是宿主机的 meminfo。

   ```bash
   # 看一下 CPU 的核数
   cat /proc/cpuinfo| grep "processor"| wc -l
   ```

   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ggf3q20it9j31z003yq6j.jpg" alt="image-20200704181007610" style="zoom:50%;" />

   结果符合我们的猜想，没有开启 lxcfs ，容器所看到的 cpuinfo 就是宿主机的。

   ### 开启 lxcfs

   ```bash
   systemctl start lxcfs

   # 启动一个容器，用 lxcfs 的 /proc 文件映射到容器中的 /proc 文件，容器内存设置为 256M：
   docker run -it -m 256m \\
         -v /var/lib/lxcfs/proc/cpuinfo:/proc/cpuinfo:rw \\
         -v /var/lib/lxcfs/proc/diskstats:/proc/diskstats:rw \\
         -v /var/lib/lxcfs/proc/meminfo:/proc/meminfo:rw \\
         -v /var/lib/lxcfs/proc/stat:/proc/stat:rw \\
         -v /var/lib/lxcfs/proc/swaps:/proc/swaps:rw \\
         -v /var/lib/lxcfs/proc/uptime:/proc/uptime:rw \\
         ubuntu:latest /bin/bash

   free -h
   ```

   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ggf3qo5o8pj32he08012z.jpg" alt="image-20200704181044965" style="zoom:50%;" />

   可以看到容器本身的内存被正确获取到了，对于内存的资源视图隔离是成功的。

   ```bash
   # --cpus 2，限定容器最多只能使用两个逻辑CPU

   docker run -it --rm -m 256m  --cpus 2  \\
         -v /var/lib/lxcfs/proc/cpuinfo:/proc/cpuinfo:rw \\
         -v /var/lib/lxcfs/proc/diskstats:/proc/diskstats:rw \\
         -v /var/lib/lxcfs/proc/meminfo:/proc/meminfo:rw \\
         -v /var/lib/lxcfs/proc/stat:/proc/stat:rw \\
         -v /var/lib/lxcfs/proc/swaps:/proc/swaps:rw \\
         -v /var/lib/lxcfs/proc/uptime:/proc/uptime:rw \\
         ubuntu:latest /bin/sh
   ```

   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ggf3s9iz85j31lc05qdj8.jpg" alt="image-20200704181153515"  />

   cpuinfo 也是我们所限制容器所能使用的逻辑 cpu 的数量了。指定容器只能在指定的 CPU 数量上运行应当是利大于弊的，就是在创建容器的时候需要额外做点工作，合理分配 cpuset。

   ## lxcfs 的 Kubernetes实践

   在kubernetes中使用lxcfs需要解决两个问题：

   第一个问题是每个node上都要启动 lxcfs；

   第二个问题是将 lxcfs 维护的 /proc 文件挂载到每个容器中；

   ## DaemonSet方式来运行 lxcfs FUSE文件系统

   针对第一个问题，我们使用 daemonset 在每个 k8s node 上都安装 lxcfs。

   直接使用下面的 yaml 文件：

   ```bash
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: lxcfs
     labels:
       app: lxcfs
   spec:
     selector:
       matchLabels:
         app: lxcfs
     template:
       metadata:
         labels:
           app: lxcfs
       spec:
         hostPID: true
         tolerations:
         - key: node-role.kubernetes.io/master
           effect: NoSchedule
         containers:
         - name: lxcfs
           image: registry.cn-hangzhou.aliyuncs.com/denverdino/lxcfs:3.0.4
           imagePullPolicy: Always
           securityContext:
             privileged: true
           volumeMounts:
           - name: cgroup
             mountPath: /sys/fs/cgroup
           - name: lxcfs
             mountPath: /var/lib/lxcfs
             mountPropagation: Bidirectional
           - name: usr-local
             mountPath: /usr/local
         volumes:
         - name: cgroup
           hostPath:
             path: /sys/fs/cgroup
         - name: usr-local
           hostPath:
             path: /usr/local
         - name: lxcfs
           hostPath:
             path: /var/lib/lxcfs
             type: DirectoryOrCreate
   ```

   ```bash
   kubectl apply -f lxcfs-daemonset.yaml
   ```

   <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ggf3srgk92j32j006sgtt.jpg" alt="image-20200704181243710" style="zoom:50%;" />

   可以看到 lxcfs 的 daemonset 已经部署到每个 node 上。

   ### 映射 lxcfs 的 proc 文件到容器

   针对第二个问题，我们两种方法来解决。

   第一种就是简单地在 k8s deployment 的 yaml 文件中声明对宿主机 `/var/lib/lxcfs/proc` 一系列文件的挂载。

   第二种方式利用Kubernetes的扩展机制 Initializer，实现对 lxcfs 文件的自动化挂载。但是 `InitializerConfiguration` 的功能在 k8s 1.14 之后就不再支持了，这里不再赘述。但是我们可以实现 admission-webhook （准入控制（Admission Control）在授权后对请求做进一步的验证或添加默认参数, https://kubernetes.feisky.xyz/extension/auth/admission）来达到同样的目的。

   ```bash
   # 验证你的 k8s 集群是否支持 admission
   $ kubectl api-versions | grep admissionregistration.k8s.io/v1beta1
   admissionregistration.k8s.io/v1beta1
   ```

   关于 admission-webhook 的编写不属于本文的讨论范围，可以到官方文档中深入了解。

   这里有一个实现 lxcfs admission webhook 的范例，可以参考：https://github.com/hantmac/lxcfs-admission-webhook

   ## 总结

   本文介绍了通过 lxcfs 提供容器资源视图隔离的方法，可以帮助一些容器化应用更好的识别容器运行时的资源限制。

   同时，在本文中我们介绍了利用容器和 DaemonSet 的方式部署 lxcfs FUSE，这不但极大简化了部署，也可以方便地利用 Kubernetes 自身的容器管理能力，支持 lxcfs 进程失效时自动恢复，在集群伸缩时也可以保证节点部署的一致性。这个技巧对于其他类似的监控或者系统扩展都是适用的。

   另外我们介绍了利用Kubernetes的 admission webhook，实现对 lxcfs 文件的自动化挂载。整个过程对于应用部署人员是透明的，可以极大简化运维复杂度。
<hr />
