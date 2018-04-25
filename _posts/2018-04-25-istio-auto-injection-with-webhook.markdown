---
layout:     post
title:      "Istio Sidecar自动注入原理"
subtitle:   "Kubernetes webhook扩展机制解析"
description: ""
date:       2018-04-25 20:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-4-25-istio-auto-injection-with-webhook/lion.jpg"
published: true
tags:
    - Kubernetes
    - Helm
    - Istio
---

## 目录
- - -
{:.no_toc}

* 目录
{:toc}

## 前言
- - -
Kubernets 1.9版本引入了Admission Webhook扩展机制，通过Webhook,开发者可以对Kubernets API Server的功能进行扩展，对API Server创建资源时对资源进行验证或者修改。

Istio 0.7版本就利用了Kubernets webhook实现了sidecar的自动注入。

## 什么是Admission
Admission是Kubernets中的一个概念，指的是API Server处理资源请求的一个阶段。如下图所示，Admission对应到请求通过鉴权之后，资源被保存到etcd之前的这段流程。 
![](\img\in-post\2018-4-25-istio-auto-injection-with-webhook\admission-phase.png)

## 参考
- - -

* [Extensible Admission is Beta](https://kubernetes.io/blog/2018/01/extensible-admission-is-beta)

