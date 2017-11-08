---
layout:     post
title:      "采用Istio实现灰度发布(金丝雀发布)"
subtitle:   "用户无感知的平滑业务升级"
date:       2017-11-07 15:00:00
author:     "赵化冰"
header-img: "img/in-post/istio-canary-release/canary_bg.jpg"
tags:
    - Microservice
    - Istio
    - Service Mesh
    - Cloud Native
    - 服务网格
    - 微服务
---

# 灰度发布（又名金丝雀发布）介绍

当应用上线以后，运维面临的一大挑战是如何能够在不影响已有线上业务的情况下对服务进行平滑升级。一般通过[灰度发布（又名金丝雀发布）](https://martinfowler.com/bliki/CanaryRelease.html)的发布流程来实现业务的平滑过渡。

灰度发布（金丝雀发布）的流程如下：

* 准备和生产环境隔离的“金丝雀”服务器。
* 将新版本的服务部署到“金丝雀”服务器上。
* 对“金丝雀”服务器上的服务进行自动化和人工测试。
* 测试通过后，将“金丝雀”服务器连接到生产环境，将少量生产流量导入到“金丝雀”服务器中。
* 如果在线测试出现问题，则通过把生产流量从“金丝雀”服务器中重新路由到老版本的服务的方式进行回退，修复问题后重新进行发布。
* 如果在线测试顺利，则逐渐把生产流量按一定策略逐渐导入到新版本服务器中。
* 待新版本服务稳定运行后，删除老版本服务。

# Istio实现灰度发布(金丝雀发布)的原理
从上面的流程可以看到，如果要实现一套灰度发布的流程，需要应用程序和运维流程对该发布过程进行支持，工作量和难度的挑战是非常大的。Istio通过高度的抽象和良好的设计采用一致的方式解决了该问题，Istio采用sidecar对应用流量进行了转发，通过Pilot下发路由规则，可以在不修改应用程序的前提下实现应用的灰度发布。

备注：采用kubernetes的[滚动升级(rolling update)](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/)功能也可以实现不中断业务的应用升级,但滚动升级是通过逐渐使用新版本的服务来替换老版本服务的方式对应用进行升级，在滚动升级不能对应用的流量分发进行控制，因此无法采用受控地把生产流量逐渐导流到新版本服务中，也就无法控制服务升级对用户造成的影响。

采用Istio后，可以通过定制路由规则将特定的流量（如指定特征的用户）导入新版本服务中，在生产环境下进行测试，同时通过渐进受控地导入生产流量，可以最小化升级中出现的故障对用户的影响。并且在同时存在新老版本服务时，还可根据应用压力对不同版本的服务进行独立的缩扩容，非常灵活。

![Istio灰度发布示意图](\img\in-post\istio-canary-release\canary-deployments.gif)

Istio和BookInfo应用程序安装可以参考[手把手教你从零搭建Istio及Bookinfo示例程序](http://zhaohuabing.com/2017/11/04/istio-install_and_example/)

# 未完待续