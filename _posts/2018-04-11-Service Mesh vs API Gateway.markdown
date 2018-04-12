---
layout:     post
title:      "（译文）Service Mesh 和 API Gateway的关系探讨"
subtitle:   ""
description: ""
date:       2018-04-11 09:32:00
author:     "赵化冰"
header-img: "img/in-post/2018-04-11-service-mesh-vs-api-gateway/background.jpg"
published: true
tags:
    - Microservice
    - Service Mesh
    - API Gateway
    - 服务网格
    - 微服务
    - API 网关
---

## 目录
{:.no_toc}

* 目录
{:toc}

## Service Mesh vs API Gateway

在我[前一篇关于Service Mesh的文章中(https://medium.com/microservices-in-practice/service-mesh-for-microservices-2953109a3c9a)有几个关于Service Mesh和API Gateway之间关系的问题。因此在我打算在本篇文章中讨论Service Mesh和API Gateway的用途。

In one of my previous articles on service mesh, there were a couple of questions related to the relationship between Service Mesh and API Gateway. So, in this post, I’m planning to discuss the usage of Service Mesh and API Gateway. 

为了区分API Gateway和Service Mesh，让我们先分别看看两者的主要特征。

In order to differentiate API Gateways and service mesh, let’s have a closer look at the key characteristics API Gateways and Service Mesh.

## API Gateway: 将你的服务作为被管理的API暴露（给外部系统）ExposesExposes your services as managed APIs

使用API Gateway的主要目的是将你的微服务作为被管理的API暴露（给外部系统）。因此，我们在API Gateway层开发的API或者边界服务对外提供了业务功能。

The key objective of using API Gateway is to expose your (micro) services as managed APIs. So, the API or Edge services that we develop at the API Gateway layer serves a specific business functionality.

API/边界服务调用下游微服务，通过组合/混装多个下游微服务形成业务逻辑。

API/Edge services call the downstream (composite and atomic) microservices and contain the business logic that creates compositions/mashups of multiple downstream services.

API/Edge服务需要以一种可靠的方式调用下游服务，因此需要应用断路器，超时，负载均衡/故障转移等可靠性模式。因此大部分的API Gateway解决方案都内置了这些特性。

API/Edge services also need to call the downstream services on the resilient manner and apply various stability patterns such as Circuit Breakers, Timeouts, Load Balancing/Failover. Therefore most of the API Gateway solutions out there have these features built in.

API Gateway也需要内置对以下特性的支持，包括：服务发现，分析（可见性：性能指标，监控，分布式日志，分布式调用追踪） 和安全。

API Gateways also come inbuilt support for service discovery, analytics(observability: Metrics, monitoring, distributed logging, distributed tracing.) and security.

API Gateway 和API管理生态系统的其他这些组件的关系紧密，比如： API 市场/商店， API 发布门户。

API Gateways closely work with several other components of the API Management ecosystem, such as API marketplace/store, API publishing portal.

## Service Mesh

现在我们来看看Service Mesh有哪些不同。

Now let’s look at how we can differentiate Service Mesh.

Service Mesh是一个网络通信基础设施， 可以用于将网络功能从你的服务代码中剥离出来。

Service Mesh is a network communication infrastructure which allows your to decouple and offload most of the application network functions from your service code.

采用Service Mesh， 你不用在服务代码中实现用于可靠通信的模式如断路，超时等，类似地，Service Mesh也提供了服务发现，服务可见性等其他功能。

Hence when you do service-to-service communication, you don’t need to implement resilient communication patterns such as Circuit breakers, timeouts in your service’s code. Similarly, service mesh provides other functionalities such as service discovery, observability etc.
API Gateway and Service Mesh in Action

## API Gateway和Service Mesh实

API Gateway和Service Mesh之间的主要不同点在于API Gateway是暴露API/边界服务的关键组件，而Service Mesh则仅仅是一个服务间通信的基础设施，并不了解你应用中的业务逻辑。

The key differentiators between API Gateways and service mesh is that API Gateways is a key part of exposing API/Edge services where service mesh is merely an inter-service communication infrastructure which doesn’t have any business notion of your solution.

下图说明了API Gateway和Service Mesh的关系。如同我们上所说，他们之间也有一些重叠的部分（例如断路器等），但重要的是需要理解这两者是用于完全不同的用途。

Figure 1 illustrates how API Gateway and service mesh can exist. As we discussed above, there are also some overlapping features (such as circuit breakers etc.) but it’s important to understand these two concepts are serving fundamentally different requirements.

图1： API Gateway和Service Mesh实践

Figure 1: API Gateways and service mesh in action
[]
![](\img\in-post\2018-04-11-service-mesh-vs-api-gateway\service-mesh-vs-api-gateway.png)

如图所示，Service Mesh作为Sidecar（边车）和服务一起部署，它是独立于服务的业务逻辑的。

As shown in figure 1, service mesh is used alongside most of the service implementations as a sidecar and it’s independent of the business functionality of the services.

另一方面，API Gateway host了所有的API服务（这些API服务有明确定义的业务功能），它是你应用的业务逻辑的一部分。API Gateway可以具有内建的服务间通信能力，但它也可以使用Service Mesh来调用下游服务。（API Gateway->Service Mesh->Microservices）。

On the other hand, API Gateway hosts all the API services (which has a clearly defined business functionality) and it’s a part of the business functionality of your solution. API Gateway may have in-built inter-service communication capabilities but that doesn’t prevent API Gateway using service mesh to call downstream services(API Gateway->service mesh->microservices).

在API管理层次，你可以使用API Gateway内建的服务间通信能力；也可以通过Service Mesh来调用下游服务，以将应用网络通信功能从应用程序转移到Service Mesh中。

At API Management level, you can either use in-built inter-service communication capabilities of API Gateway or API Gateway can call downstream services via service mesh by offloading application network functions to service mesh.

## 原文

* [Service Mesh vs API Gateway](https://medium.com/microservices-in-practice/service-mesh-vs-api-gateway-a6d814b9bf56)

