---
layout:     post
title:      "Istio灰度发布实践"
subtitle:   "采用Istio实现对用户无感知的平滑业务升级"
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
当应用上线以后，运维面临的一大挑战是如何能够在不影响上线业务的情况下对服务进行平滑升级。一般通过[灰度发布（又名金丝雀发布）](https://martinfowler.com/bliki/CanaryRelease.html)的发布流程来实现业务的平滑过渡。如果要自己实现一套灰度发布的流程，工作量和难度的挑战是非常大的。然后通过Istio的带权重路由规则，可以轻松地将应用流量逐渐从老版本的服务组件迁移到新版本的服务中，以实现应用新版本的灰度发布。


采用kubernetes的[应用版本升级](https://kubernetes.io/docs/tutorials/kubernetes-basics/update-intro/)功能也可以实现不中断业务的应用升级。但采用Istio后，可以通过定制路由规则将特定的流量（如指定特征的用户）导入新版本服务中，在生产环境下进行测试，同时屏蔽对VIP用户的影响，待测试充分后再逐渐把流量导入到新版本服务中。并且在有多个版本服务时，还可对不同版本的服务进行独立的Scaling。

![Istio灰度发布示意图](\img\in-post\istio-traffic-shifting\canary-deployments.gif)

Istio和BookInfo应用程序安装可以参考[手把手教你从零搭建Istio及Bookinfo示例程序](http://zhaohuabing.com/2017/11/04/istio-install_and_example/)

##未完待续