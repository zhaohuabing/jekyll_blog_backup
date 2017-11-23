---
layout:     post 
title:      "Nginx ingress controller"
subtitle:   "如何向外暴露Kubernetes集群中的服务"
date:       2017-11-20 12:00:00
author:     "赵化冰"
header-img: "img/post-bg-2015.jpg"
published: true
tags:
    - Kubernetes
    - Ingress
    - Nginx
    - 微服务
    - 云原生
---

## 目录
{:.no_toc}

* 目录
{:toc}


## 前言

这篇文章将介绍如何使用Ingress Controller将Kubernetes集群中运行的应用暴露到Internet上。

我们首先来了解一些Kubernetes中关于Ingress和Service的基本概念。

简单地说，Service是一个对Pod的逻辑抽象通信层。在Kubernetes集群中，Pod是应用部署的基本单元，会不停地被创建，销毁，缩扩容等，kubernetes在创建Pod时可以选择集群中的任何一台空闲的Host，因此其网络地址也不是固定的。由于Pod的这一特点，一般不能够直接通过Pod去访问应用。

为了解决该问题，Kubernetes采用了Service的概念，Service是对后端提供某一种服务的Pod集合的抽象，Service会绑定到一个固定的虚拟IP上，该虚拟IP只在Kubernetes Cluster中可见，但其实该IP并不对应一个虚拟或者物理设备，而只是IPtable中的规则，然后再通过IPtable将服务请求路由到后端的Pod中。通过这种方式，可以确保服务客户端一直可以稳定地访问Pod提供的服务，而不用关心Pod的创建、删除、迁移等变化。

Service的类型(ServiceType)决定了Service如何暴露其提供的服务，根据类型不同，服务可以只在Kubernetes cluster中可见，也可以暴露到Cluster外部。Service的类型有：

ClusterIP 这种类型的服务会提供一个只能在Cluster内部可以访问的内部虚拟IP，这是Service的缺省类型。

NodePort 这种类型的服务会在Cluster中的每台主机上选择一个端口暴露该服务。注意每台主机上的该端口都可以访问到该服务，发送到该主机端口的请求会被kubernetes路由到提供服务的Pod上。采用这种服务类型，可以在kubernetes cluster网络外通过主机IP：端口的方式访问到服务。

LoadBalancer Supported on Amazon and Google cloud, this creates the cloud providers your using load balancer. So on Amazon it creates a ELB that points to your service on your cluster.

ExternalName Create a CNAME dns record to a external domain.

For more information about Services look at https://kubernetes.io/docs/user-guide/services/

A Ingress is rules on how to access a Service from the internet.

For more information about Ingresses look at https://kubernetes.io/docs/user-guide/ingress/

Scenario

