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
用户登录时，应用的安全模块对用户身份进行验证，验证用户身份合法后，为该用户生成一个会话(Session)，该Session关联了一个唯一的Session Id。Session是应用中的一小块内存结构，保存了登录用户的信息，如User name, Role, Permission等。该Session的Session Id被返回给客户端，客户端将Session Id以cookie或者URL重写的方式记录下来，并在后续请求中发送给应用，这样应用在接收到客户端访问请求时可以使用Session Id验证用户身份，不用每次请求时都输入用户名和密码进行身份验证。
> 备注：为了避免Session Id被第三者截取和盗用，Session设置有过期时间，客户端和应用之前的通讯也应使用HTTPS加密通信。
![单体应用用户登录认证序列图](\img\in-post\2018-02-03-authentication&authorization-of-microservice\monolith-user-login.png)
<center>单体应用用户登录认证序列图</center>

客户端访问应用时，Session Id随着HTTP请求发送到应用，客户端请求一般会通过一个拦截器处理所有收到的客户端请求。拦截器首先判断Session Id是否存在，如果该Session Id存在，就知道该用户已经登录。然后再通过用户的拥有的权限判断用户能否执行该此请求，以实现操作鉴权。
![单体应用用户操作鉴权序列图](\img\in-post\2018-02-03-authentication&authorization-of-microservice\monolith-user-request.png)
<center>单体应用用户操作鉴权序列图</center>

## 微服务架构下认证和鉴权面临的问题
在微服务架构下，一个应用被拆分为多个微服务，每个微服务实现原来单体应用中一个模块的业务功能。应用拆分后，对每个微服务的访问请求都需要进行认证和鉴权。如果参考单体应用的实现方式会遇到下述问题：
* 认证和鉴权逻辑需要在每个微服务中进行处理，导致这部分代码逻辑重复。虽然我们可以使用lib复用部分代码，但这又会导致所有微服务对特定代码库及其版本存在依赖，影响微服务语言/框架选择的灵活性。
* 微服务应遵循单一职责原理，一个微服务只处理单一的业务逻辑。认证和鉴权的公共逻辑不应该放到微服务实现中。 
* 为了充分利用微服务架构的好处，实现微服务的水平扩展(Scalability)和弹性(Resiliency),微服务最好是无状态的。因此不建议使用session这种有状态的方案。
* 微服务架构下的认证和鉴权涉及到场景更为复杂，涉及到用户访问微服务应用，第三方应用访问微服务应用，应用内多个微服务之间相互访问等多种场景，每种场景下的认证和鉴权方案都需要考虑到，以保证应用程序的安全性。
![微服务认证和鉴权涉及到的三种场景](\img\in-post\2018-02-03-authentication&authorization-of-microservice\auth-scenarios.png)
<center>微服务认证和鉴权涉及到的三种场景</center>

## 微服务架构下认证和鉴权的技术方案

### 用户身份认证
一个完整的微服务应用是由多个相互独立的微服务进程组成的，对每个微服务的访问都需要进行用户认证。如果将用户认证的工作放到每个微服务中，应用的认证逻辑将会非常复杂。因此需要考虑一个SSO（单点登录）的方案，即用户只需要登录一次，就可以访问所有微服务提供的服务。 由于在微服务架构中，一般以API Gateway作为对外提供服务的入口，因此可以考虑在API Gateway处提供统一的用户认证。

### 用户权限控制

### 第三方应用接入

### 微服务之间的认证

## 参考

* [How We Solved Authentication and Authorization in Our Microservice Architecture](https://initiate.andela.com/how-we-solved-authentication-and-authorization-in-our-microservice-architecture-994539d1b6e6)

* [深入聊聊微服务架构的身份认证问题](http://www.primeton.com/read.php?id=2390)


