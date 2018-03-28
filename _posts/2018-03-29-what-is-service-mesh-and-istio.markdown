---
layout:     post 
title:      "微服务基础架构：Service Mesh模式和Istio开源项目"
subtitle:   ""
description: "微服务基础架构：Service Mesh模式和Istio开源项目。"
date:       2018-03-29 12:00:00
author:     "赵化冰"
header-img: "img/in-post/istio-install_and_example/post-bg.jpg"
published: true
tags:
    - Microservice
    - Service Mesh
    - Istio
---

## 目录
{:.no_toc}

* 目录
{:toc}


## 什么是服务网格(Service Mesh)，我们为何需要它？
作为一种架构模式，微服务将复杂系统切分为数十乃至上百个小服务，每个服务一般只负责实现一个独立的业务逻辑。这些小服务易于被小型的软件工程师团队所理解和修改，并带来了语言和框架选择灵活性，加快应用开发上线时间，服务可以根据不同的工作负载进行独立缩扩容等优势。

在带来便利的同时，微服务架构也引入了对大量服务的连接、管理和监控的复杂性。应用被拆分为多个微服务进程后，进程内的方法调用变成了了进程间的远程调用，带来了分布式系统的一系列问题，例如：

* 如何找到服务的提供方？
* 如何保证远程方法调用的可靠性？
* 如何保证服务调用的安全性？
* 如何降低服务调用的延迟？
* 如何进行端到端的调试？

另外生产部署中的微服务实例也增加了运维的难度,例如：

* 如何收集大量微服务的性能指标已进行分析？
* 如何在不影响上线业务的情况下对微服务进行升级？
* 如何测试一个微服务集群部署的容错和稳定性？

这些问题涉及到成百上千个服务的通信、管理、部署、版本、安全、故障转移、策略执行、遥测和监控等，实现它们并非易事。

让我们来回顾一下微服务架构的发展过程。在出现服务网格之前，我们最开始在微服务应用程序内理服务之间的通讯逻辑，包括服务发现，熔断，重试，超时，加密，限流等逻辑。
![](\img\in-post\2018-03-29-what-is-service-mesh-and-istio\1.png)
在一个分布式系统中，这部分逻辑比较复杂，为了为微服务应用提供一个稳定、可靠的基础设施层，避免大家重复造轮子，并减少犯错的可能，一般会通过对这部分负责服务通讯的逻辑进行抽象和归纳，形成一个代码库供各个微服务应用程序使用，如下图所示：
![](\img\in-post\2018-03-29-what-is-service-mesh-and-istio\2.png)
公共的代码库减少了应用程序的开发和维护工作量，降低了由应用开发人员单独实现微服务通讯逻辑出现错误的机率，但还是存在下述问题：
* 微服务通讯逻辑对应用开发人员并不透明，应用开发人员需要理解并正确使用代码			库，不能将其全部精力聚焦于业务逻辑。
* 需要针对不同的语言/框架开发不同的代码库，反过来会影响微服务应用开发语言			和框架的选择，影响技术选择的灵活性。
* 随着时间的变化，代码库会存在不同的版本，不同版本代码库的兼容性和大量运行			环境中微服务的升级将成为一个难题。

可以将微服务之间的通讯基础设施层类和TCP/IP协议栈进行类比。TCP/IP协议栈为操作系统中的所有应用提供通信服务，但并不需要和应用部署在一个进程中，应用也并不关心TCP/IP协议的实现。同样地，如果将为微服务提供通信服务的这部分逻辑从应用程序进程中抽取出来，作为一个单独的进程进行部署，并将其作为服务间的通信代理，可以得到如下图所示的架构：
![](\img\in-post\2018-03-29-what-is-service-mesh-and-istio\sidecar.png)
因为通讯代理进程伴随应用进程一起部署，因此形象地把这种部署方式称为“sidecar”/边车（即三轮摩托的挎斗）。

应用间的所有流量都需要经过代理，由于代理以sidecar方式和应用部署在同一台主机上，应用和代理之间的通讯可以被认为是可靠的。由代理来负责找到目的服务并负责通讯的可靠性和安全等问题。

当服务大量部署时，随着服务部署的sidecar代理之间的连接形成了一个如下图所示的网格，该网格成为了微服务的通讯基础设施层，承载了微服务之间的所有流量，被称之为Service Mesh（服务网格）。
![](\img\in-post\2018-03-29-what-is-service-mesh-and-istio\mesh.png)

_服务网格是一个基础设施层，用于处理服务间通信。云原生应用有着复杂的服务拓扑，服务网格保证请求可以在这些拓扑中可靠地穿梭。在实际应用当中，服务网格通常是由一系列轻量级的网络代理组成的，它们与应用程序部署在一起，但应用程序不需要知道它们的存在。_

_William Morgan _[_WHAT’S A SERVICE MESH? AND WHY DO I NEED ONE?_](https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/)_
为了更方便地对代理组成的网格进行控制，采用控制面板对代理的行为进行控制和度量指标收集。

这里我们可以类比SDN的概念，控制面就类似于SDN网管中的控制器，负责路由策略的指定和路由规则下发；数据面类似于SDN网络中交换机，负责数据包的转发。由于微服务的所有通讯都有服务网格基础设施层提供，通过控制面板和数据面板的配合，可以对这些通讯进行监控、托管和控制，以实现微服务灰度发布，调用分布式追踪，故障注入模拟测试，动态路由规则，微服务闭环控制等管控功能。
![](\img\in-post\2018-03-29-what-is-service-mesh-and-istio\controlplane.png)

## Istio服务网格



## 参考

* [How We Solved Authentication and Authorization in Our Microservice Architecture](https://initiate.andela.com/how-we-solved-authentication-and-authorization-in-our-microservice-architecture-994539d1b6e6)
* [How to build your own public key infrastructure](https://blog.cloudflare.com/how-to-build-your-own-public-key-infrastructure/)
* [OAuth 2.0 Authorization Code Request](https://www.oauth.com/oauth2-servers/access-tokens/authorization-code-request/)
* [PKI/CA工作原理及架构](https://www.jianshu.com/p/c65fa3af1c01)
* [深入聊聊微服务架构的身份认证问题](http://www.primeton.com/read.php?id=2390)


