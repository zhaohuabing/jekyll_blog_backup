---
layout:     post
title:      "Istio Sidecar自动注入原理解析"
subtitle:   "Kubernetes webhook扩展机制"
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
Kubernets 1.9版本增加了Admission Webhook扩展机制，通过该扩展机制，可以对API Server的功能进行扩展。当一个Resource在API Sever中被创建时，可以通过自定义的webhook对创建的资源进行验证或者修改。Istio 0.7版本就利用了Kubernets webhook实现了sidecar的自动注入。

## 未完待续

## 参考
- - -

* [Extensible Admission is Beta](https://kubernetes.io/blog/2018/01/extensible-admission-is-beta)

