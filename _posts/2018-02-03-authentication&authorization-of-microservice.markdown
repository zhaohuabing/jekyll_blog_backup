---
layout:     post 
title:      "微服务架构下的认证和鉴权实现方案"
subtitle:   ""
description: "微服务架构下的认证和鉴权实现方案。"
date:       2018-02-03 12:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-02-03-authentication&authorization-of-microservice/AuthenticationTrack.jpeg"
published: true
tags:
    - Microservice
    - Security
---

## 目录
{:.no_toc}

* 目录
{:toc}


## 前言

微服务架构的引入为软件应用带来了诸多好处：例如小团队管理，缩短开发周期，语言选择灵活性，增强服务伸缩能力等。与此同时，也引入了分布式系统的诸多复杂问题。其中一个挑战就是如何在微服务架构中实现一个灵活，安全，高效的认证和鉴权方案。本文将就此问题进行一些探讨。

## 单体应用的实现方式
在单体架构下，应用是一个整体，在应用中，一般会用一个安全模块来实现用户认证和鉴权。

用户登录时，应用的安全模块对用户身份进行验证，验证用户身份合法后，为该用户生成一个会话(Session)，该Session关联了一个唯一的Session Id。Session是应用中的一小块内存结构，保存了登录用户的信息，如User name, Role, Permission等。该Session的Session Id被返回给客户端，客户端将此记录下来并随着请求发送到应用，这样应用在接收到客户端访问请求时可以使用Session Id验证用户身份，不用每次请求时都输入用户名和密码进行身份验证。
> 备注：为了避免Session Id被第三者截取和盗用，Session设置有过期时间，客户端和应用之前的通讯也应使用HTTPS加密通信。

![单体应用用户登录认证序列图](\img\in-post\2018-02-03-authentication&authorization-of-microservice\monolith-user-login.png)

客户端将收到的Session Id保存在Cookie中，或者通过URL重写放到URL中。客户端访问应用时，Session Id随着HTTP请求发送到应用，客户端请求一般会通过一个拦截器处理所有收到的客户端请求。拦截器首先判断Session Id是否存在，如果该Session Id存在，就知道该用户已经登录。然后再通过用户的拥有的权限判断用户能否执行该此请求，以实现操作鉴权。
![单体应用用户操作鉴权序列图](\img\in-post\2018-02-03-authentication&authorization-of-microservice\monolith-user-request.png)

## 微服务架构下认证和鉴权面临的问题

## 微服务架构下认证和鉴权的技术方案

### 用户身份认证

### 用户权限控制

### 第三方应用接入

### 微服务之间的认证

## 参考

* [How We Solved Authentication and Authorization in Our Microservice Architecture](https://initiate.andela.com/how-we-solved-authentication-and-authorization-in-our-microservice-architecture-994539d1b6e6)

* [深入聊聊微服务架构的身份认证问题](http://www.primeton.com/read.php?id=2390)


