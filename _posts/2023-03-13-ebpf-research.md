---
layout: post
title:  "探讨使用eBPF虚拟机替换K8S中的容器的可能性"
date:   2023-03-13 18:55:44 +0800
categories: jekyll update
---
## **K8S集群的工作原理**

在k8s集群的工作节点（Node）中，kubelet作为关键组件会通过CRI（Container Runtime Interface）调用指定的容器运行时，由其负责容器的创建、管理和监控。因此K8S要求用户**必须使用符合CRI的容器运行时**，如containerd和CRI-O等。运行时，K8S会使用指定的容器运行时在工作节点上创建Pod。在同一个Pod中的容器会共享存储空间、IP地址和端口，加强关联业务的通讯和文件共享。容器的加载则通过加载满足OCI标准的镜像去实现。

以containerd为例子，容器加载时，会先建立一个**shim**守护进程，负责为containerd提供API进行容器的管理。**shim**会调用一个负责容器进程运行的**runc**，从OCI镜像文件提取需要运行的应用运行。**runc**就是容器实际上的运行时。
![avatar](https://www.tutorialworks.com/assets/images/container-ecosystem.drawio.png?ezimgfmt=rs:704x1183/rscb6/ngcb6/notWebP)

## **可能需要完成的组件或接口**
* **一个使用eBPF虚拟机的runc**

   传统的runc的runtime应该无法满足ebpf程序的加载，或者并不是专门为了运行eBPF程序而设计的。因此需要重写一个为加载ebpf程序的OCI镜像的runc，用以完成最底层的容器的CRUD。
   
* **一个为容器运行时提供的容器管理API**

   新的runc可能需要为容器运行时提供一个不一样的API进行控制，因此也是需要为此进行开发的。
   

* **一个专用的处理eBPF程序镜像的容器运行时**
  
   一般的容器运行时会兼容不同程序的运行，在为程序选择运行时时可能会造成新能损失。若有确切需求，可以重写一份专用的处理eBPF程序镜像的容器运行时。该容器运行时需要与上方提及的API适配，并与满足上层的CRI，用以接入k8s。
   



可能总结的没有那么好，大家可以给我一点想法或者意见orz


***
## **参考链接：**
[1]The differences between Docker, containerd, CRI-O and runc

 <https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/>
