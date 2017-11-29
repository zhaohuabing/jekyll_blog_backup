---
layout:     post 
title:      "如何从外部访问Kubernetes集群中的应用？"
subtitle:   ""
description: "我们知道，kubernetes的Cluster Network属于内部网络，缺省情况下只能在cluster Network内部才能访问部署的应用，那如何才能从外部网络s访问Kubernetes集群中的应用呢？本文介绍了从外部访问kubernetes cluster中应用的几种实现方式。"
date:       2017-11-28 12:00:00
author:     "赵化冰"
header-img: "img/post-bg-2015.jpg"
published: true
tags:
    - Kubernetes
    - Ingress
    - 微服务
    - 云原生
    - Load Balancer
---

## 目录
{:.no_toc}

* 目录
{:toc}


## 前言

我们知道，kubernetes的Cluster Network属于私有网络，只能在cluster Network内部才能访问部署的应用，那如何才能将Kubernetes集群中的应用暴露到外部网络，为外部用户提供服务呢？本文介绍了从外部访问kubernetes cluster中应用的几种实现方式。

我们首先来了解一些Kubernetes中Service的基本概念。

## Pod和Service

简单地说，Service是一个对Pod的逻辑抽象层。在Kubernetes集群中，Pod是应用部署的基本单元，会不停地被创建，销毁，缩扩容等，kubernetes在创建Pod时可以选择集群中的任何一台空闲的Host，因此其网络地址也不是固定的。由于Pod的这一特点，一般不建议直接通过Pod的地址去访问应用。

为了解决该问题，Kubernetes采用了Service的概念，Service是对后端提供服务的一组Pod的抽象，Service会绑定到一个固定的虚拟IP上，该虚拟IP只在Kubernetes Cluster中可见，但其实该IP并不对应一个虚拟或者物理设备，而只是IPtable中的规则，然后再通过IPtable将服务请求路由到后端的Pod中。通过这种方式，可以确保服务消费者可以稳定地访问Pod提供的服务，而不用关心Pod的创建、删除、迁移等变化以及如何用一组Pod来进行负载均衡。

Service的机制如下图所示，Kube-proxy监听kubernetes master增加和删除Service以及Endpoint的消息，对于每一个Service，kube proxy创建相应的iptables规则，将发送到Service Cluster IP的流量转发到Service后端提供服务的Pod的相应端口上。
![Pod和Service的关系](\img\in-post\access-application-from-outside\services-iptables-overview.png)

>备注：可以通过Service的Cluster IP和服务端口访问到后端Pod提供的服务，但该Cluster IP是Ping不通的，原因是Cluster IP只是iptable中的规则，并不对应到一个网络设备。

## Service的类型
Service的类型(ServiceType)决定了Service如何对外提供服务，根据类型不同，服务可以只在Kubernetes cluster中可见，也可以暴露到Cluster外部。Service有三种类型，ClusterIP，NodePort和LoadBalancer。其中ClusterIP是Service的缺省类型，这种类型的服务会提供一个只能在Cluster内部可以访问的内部虚拟IP，如上面一节所述。

## 通过NodePort从外部访问

通过将Service的类型设置为NodePort，可以在Cluster中的主机上通过一个指定端口暴露服务。注意通过Cluster中每台主机上的该指定端口都可以访问到该服务，发送到该主机端口的请求会被kubernetes路由到提供服务的Pod上。采用这种服务类型，可以在kubernetes cluster网络外通过主机IP：端口的方式访问到服务。

> 注意：官方文档中说明了Kubernetes clusterIp的流量转发到后端Pod有Iptable和kube proxy两种方式。但对Nodeport如何转发流量却语焉不详。该图来自网络，从图来看是通过kube proxy转发的，我没有去研究过源码。欢迎看过源码的同学指教。

![Pod和Service的关系](\img\in-post\access-application-from-outside\nodeport.PNG)

下面是通过NodePort向外暴露服务的一个例子，注意可以指定一个nodePort，也可以不指定。在不指定的情况下，kubernetes会从可用的端口范围内自动分配一个随机端口。

```
kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: NodePort
  ports:
    - port: 8086
      nodePort: 30000
  selector:
    name: influxdb
```

通过NodePort从外部访问有下面的一些问题，玩玩可以，但不适宜用于生产环境。

* Kubernetes cluster host的IP必须是一个well known IP，即客户端必须知道该IP。由于一般来讲Cluster中的host是作为资源池看待的，可以增加删除，IP一般也是动态分配的，因此并不能假设host IP对客户端而言是well-known IP。

* 客户端访问某一个固定的host IP存在单点故障，假如该host宕机，虽然kubernetes cluster可以把应用 reload到另一节点上，但客户端就无法通过该host的nodeport访问应用了。

* 该方案假设客户端可以访问Kubernetes host所在网络。在生产环境中，客户端和Kubernetes host网络可能是隔离的。例如客户端可能是公网中的一个手机APP，是无法直接访问host所在的私有网络的。

因此，需要通过一个外部Load Balancer来将外部客户端的流量导入到Cluster中的应用中。

## 通过Load Balancer从外部访问

通过将Service的类型设置为LoadBalancer，可以为Service创建一个外部Load Balancer。Kubernetes的文档中声明该Service类型需要云服务提供商的支持，其实这里只是在Kubernetes配置文件中提出了一个要求，即为该Service创建Load Balancer，至于如何创建则是由Google Cloud或Amazon Cloud等云服务商提供的，创建的Load Balancer不在Kubernetes Cluster的管理范围中。kubernetes 1.6版本中，WS, Azure, CloudStack, GCE and OpenStack等云提供商已经可以为Kubernetes提供Load Balancer.下面是一个Load balancer类型的Service例子：

```
kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: LoadBalancer
  ports:
    - port: 8086
  selector:
    name: influxdb
```
部署该Service后，我们来看一下Kubernetes创建的内容
```
$ kubectl get svc influxdb
NAME       CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
influxdb   10.97.121.42   10.13.242.236   8086:30051/TCP   39s
```
Kubernetes首先为influxdb创建了一个集群内部可以访问的ClusterIP 10.97.121.42。由于没有指定nodeport端口，kubernetes选择了一个空闲的30051主机端口将service暴露在主机的网络上，然后通知cloud provider创建了一个load balancer，上面输出中的EEXTERNAL-IP就是load balancer的IP。

测试使用的Cloud Provider是OpenStack，我们通过neutron lb-vip-show可以查看创建的Load Balancer详细信息。

```
$ neutron lb-vip-show 9bf2a580-2ba4-4494-93fd-9b6969c55ac3
+---------------------+--------------------------------------------------------------+
| Field               | Value                                                        |
+---------------------+--------------------------------------------------------------+
| address             | 10.13.242.236                                                |
| admin_state_up      | True                                                         |
| connection_limit    | -1                                                           |
| description         | Kubernetes external service a6ffa4dadf99711e68ea2fa163e0b082 |
| id                  | 9bf2a580-2ba4-4494-93fd-9b6969c55ac3                         |
| name                | a6ffa4dadf99711e68ea2fa163e0b082                             |
| pool_id             | 392917a6-ed61-4924-acb2-026cd4181755                         |
| port_id             | e450b80b-6da1-4b31-a008-280abdc6400b                         |
| protocol            | TCP                                                          |
| protocol_port       | 8086                                                         |
| session_persistence |                                                              |
| status              | ACTIVE                                                       |
| status_description  |                                                              |
| subnet_id           | 73f8eb91-90cf-42f4-85d0-dcff44077313                         |
| tenant_id           | 4d68886fea6e45b0bc2e05cd302cccb9                             |
+---------------------+--------------------------------------------------------------+

$ neutron lb-pool-show 392917a6-ed61-4924-acb2-026cd4181755
+------------------------+--------------------------------------+
| Field                  | Value                                |
+------------------------+--------------------------------------+
| admin_state_up         | True                                 |
| description            |                                      |
| health_monitors        |                                      |
| health_monitors_status |                                      |
| id                     | 392917a6-ed61-4924-acb2-026cd4181755 |
| lb_method              | ROUND_ROBIN                          |
| members                | d0825cc2-46a3-43bd-af82-e9d8f1f85299 |
|                        | 3f73d3bb-bc40-478d-8d0e-df05cdfb9734 |
| name                   | a6ffa4dadf99711e68ea2fa163e0b082     |
| protocol               | TCP                                  |
| provider               | haproxy                              |
| status                 | ACTIVE                               |
| status_description     |                                      |
| subnet_id              | 73f8eb91-90cf-42f4-85d0-dcff44077313 |
| tenant_id              | 4d68886fea6e45b0bc2e05cd302cccb9     |
| vip_id                 | 9bf2a580-2ba4-4494-93fd-9b6969c55ac3 |
+------------------------+--------------------------------------+

$ neutron lb-member-list
+--------------------------------------+--------------+---------------+--------+----------------+--------+
| id                                   | address      | protocol_port | weight | admin_state_up | status |
+--------------------------------------+--------------+---------------+--------+----------------+--------+
| 3f73d3bb-bc40-478d-8d0e-df05cdfb9734 | 10.13.241.89 |         30051 |      1 | True           | ACTIVE |
| d0825cc2-46a3-43bd-af82-e9d8f1f85299 | 10.13.241.10 |         30051 |      1 | True           | ACTIVE |
+--------------------------------------+--------------+---------------+--------+----------------+--------
```
可以看到OpenStack使用VIP 10.13.242.236在端口8086创建了一个Load Balancer，Load Balancer对应的Lb pool里面有两个成员10.13.241.89 和 10.13.241.10，正是Kubernetes的host节点，进入Load balancer流量被分发到这两个节点对应的Service Nodeport 30051上。

但是如果客户端不在Openstack Neutron的私有子网上，则还需要在load balancer的VIP上关联一个floating IP，以使外部客户端可以连接到load balancer。

部署Load balancer后，应用的拓扑结构如下图所示：
![外部Load balancer](\img\in-post\access-application-from-outside\load-balancer.PNG)

>如果kubernetes环境在Public Cloud上，Loadbalancer类型的Service创建出的外部Load Balancer是可以直接从外部网络进行访问的，不需要绑定floating IP这个步骤。例如在AWS上创建的Elastic Load Balancing (ELB)，有兴趣可以看一下这篇文章：[Expose Services on your AWS Quick Start Kubernetes cluster]( http://docs.heptio.com/content/tutorials/aws-qs-services-elb.html)。

如果Kubernetes Cluster不是在支持Load balancer的cloud provider或者裸机上创建的，可以实现LoadBalancer类型的Service吗？Kubernetes本身并不直接支持Loadbalancer，但我们应可以通过对Kubernetes进行扩展来实现，可以监听kubernetes API server的service创建消息，并根据消息部署相应的Load Balancer，例如配置Nginx的虚拟主机，来实现Load balancer类型的Service。


当只需要向外暴露一个服务的时候，可以直接采用Loadbalancer类型Service的方式。但如果一个应用对外提供多个服务，采用该方式则需要为每一个服务（IP+Port）都创建一个外部load balancer。如下图所示
![创建多个Load balancer暴露应用的多个服务](\img\in-post\access-application-from-outside\multiple-load-balancer.PNG)
一般来说，同一个应用的多个服务/资源应该放在同一个域名下，在这种情况下，创建多个Load balancer是完全没有必要的，反而带来了额外的开销和管理成本。可以通过使用Kubernetes Ingress来解决该问题。

## Ingress
首先看一下引入Ingress后的应用拓扑示意图：
![采用Ingress暴露应用的多个服务](\img\in-post\access-application-from-outside\ingress.PNG)
这里Ingress起到了七层负载均衡器和Http方向代理的作用，可以根据不同的url把入口流量分发到不同的后端Service。外部客户端只看到foo.bar.com这个服务器，屏蔽了内部多个Service的实现方式。采用这种方式，简化了客户端的访问方式，并增加了后端实现和部署的灵活性，可以在不影响客户端的情况下对后端的服务部署进行调整。

下面是Kubernetes Ingress配置文件的示例，在虚拟主机foot.bar.com下面定义了两个Path，其中/foo被分发到后端服务s1，/bar被分发到后端服务s2。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80
```

注意这里Ingress只描述了一个虚拟主机路径分发的需求，实际上可以定义多个Ingress，描述不同的7层代理需求，而这些需求是由Ingress Controller来实现的。Ingress Contorller会监听Kubernetes Master得到Ingress的定义，并根据Ingress的定义对一个7层代理进行相应的配置，以实现Ingress定义中要求的虚拟主机和路径分发规则。Ingress Controller有多种实现，Kubernetes提供了一个[基于Nginx的Ingress Controller](https://github.com/kubernetes/ingress-nginx)。部署Kubernetes集群时并不会缺省部署Ingress Controller，需要自行部署。

下面是部署Nginx Ingress Controller的配置文件示例：

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: LoadBalancer
  ports:
    - port: 80
      name: http
    - port: 443
      name: https
  selector:
    k8s-app: nginx-ingress-lb
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 2
  revisionHistoryLimit: 3
  template:
    metadata:
      labels:
        k8s-app: nginx-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: nginx-ingress-controller
          image: gcr.io/google_containers/nginx-ingress-controller:0.8.3
          imagePullPolicy: Always
    //omitted for brevity
```

>Google Cloud直接支持Ingress资源，如果应用部署在Google Cloud中，Google Cloud会自动为Ingress资源创建一个7层load balancer，并为之分配一个外部IP，不需要自行部署Ingress Controller。

## 参考

* [Accessing Kubernetes Pods from Outside of the Cluster](http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/)

* [Kubernetes nginx-ingress-controller](https://daemonza.github.io/2017/02/13/kubernetes-nginx-ingress-controller/)

* [Using Kubernetes external load balancer feature](https://docs.openstack.org/magnum/ocata/dev/kubernetes-load-balancer.html)

* [Expose Services on your AWS Quick Start Kubernetes cluster]( http://docs.heptio.com/content/tutorials/aws-qs-services-elb.html)


