---
layout:     post 
title:      "Nginx开源Service Mesh Nginmesh安装指南"
subtitle:   ""
description: "采用kubeadmin安装kubernetes集群并部署Nginmesh sidecar。"
date:       2018-01-02 12:00:00
author:     "赵化冰"
header-img: "img/post-bg-2015.jpg"
published: true
tags:
    - Kubernetes
    - service Mesh
    - nginmesh
---

## 目录
{:.no_toc}

* 目录
{:toc}


## 前言

Nginmesh是NGINX公的Service Mesh开源项目，用于Istio服务网格平台中的7层负载均衡和代理。它旨在提供关键功能并与Istio集成，作为边车（sidecar）容器部署，并将以“标准，可靠和安全的方式”使得服务间通信更容易。Nginmesh在今年底已经连续发布了0.2和0.3版本，提供了服务发现，请求转发，路由规则，性能指标收集等功能。

![Nginmesh sidecar proxy](https://raw.githubusercontent.com/nginmesh/nginmesh/master/images/nginx_sidecar.png)

> 备注：本文安装指南基于Ubuntu 16.04，在Centos上某些安装步骤的命令可能需要稍作改动。

## 安装Kubernetes Cluster

Kubernetes Cluster包含etcd, api server, scheduler，controller manager等多个组件，组件之间的配置较为复杂，如果要手动去逐个安装及配置各个组件，需要了解kubernetes，操作系统及网络等多方面的知识，对安装人员的要求较高。kubeadm提供了一个简便，快速安装Kubernetes Cluster的方式，并且可以通过安装配置文件提供较高的灵活性，因此我们采用kubeadm安装kubernetes cluster。

首先参照[kubeadm的说明文档](https://kubernetes.io/docs/setup/independent/install-kubeadm)在计划部署kubernetes cluster的每个节点上安装docker，kubeadm, kubelet 和 kubectl。

安装docker
```
apt-get update
apt-get install -y docker.io
```

使用google的源安装kubelet kubeadm和kubectl
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
使用kubeadmin安装kubernetes cluster

Nginmesh使用Kubernetes的[Initializer机制](https://kubernetes.io/docs/admin/extensible-admission-controllers/#initializers)来实现sidecar的自动注入。Initializer目前是kubernetes的一个Alpha feature，缺省是未启用的，需要[通过api server的参数](https://kubernetes.io/docs/admin/extensible-admission-controllers/#enable-initializers-alpha-feature)打开。因此我们先创建一个kubeadm-conf配置文件，用于配置api server的启动参数

```
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
apiServerExtraArgs:
  admission-control: Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ValidatingAdmissionWebhook,ResourceQuota,DefaultTolerationSeconds,MutatingAdmissionWebhook
  runtime-config: admissionregistration.k8s.io/v1alpha1
```
使用kubeadmin init命令创建kubernetes master节点。
可以先试用--dry-run参数验证一下配置文件。
```
kubeadm init --config kubeadm-conf --dry-run
```
如果一切正常，kubeadm将提示：Finished dry-running successfully. Above are the resources that would be created.

下面再实际执行创建命令
```
kubeadm init --config kubeadm-conf
```
kubeadm会花一点时间拉取docker image，命令完成后，会提示如何将一个work node加入cluster。如下所示：

```
 kubeadm join --token fffbf6.13bcb3563428cf23 10.12.5.15:6443 --discovery-token-ca-cert-hash sha256:27ad08b4cd9f02e522334979deaf09e3fae80507afde63acf88892c8b72f143f
 ```
> 备注：目前kubeadm只能支持在一个节点上安装master，支持高可用的安装将在后续版本实现。kubernetes官方给出的workaround建议是定期备份 etcd 数据[kubeadm limitations](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#limitations)。

安装一个Pod网络，这里我采用的是Calico
```
kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

使用kubectl 命令检查master节点安装结果

```
ubuntu@kube-1:~$ kubectl get all
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
svc/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   12m
```

 在每台工作节点上执行上述kubeadm join命令，即可把工作节点加入集群中。使用kubectl 命令检查cluster中的节点情况。

```
 ubuntu@kube-1:~$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
kube-1    Ready     master    21m       v1.9.0
kube-2    Ready     <none>    47s       v1.9.0
```

## 安装Istio控制面和Bookinfo

参考[Nginmesh文档](https://github.com/nginmesh/nginmesh)安装Istio控制面和Bookinfo
该文档的步骤清晰明确，这里不再赘述。

需要注意的是，在Niginmesh文档中，建议通过Ingress的External IP访问bookinfo应用程序。但Loadbalancer只在云环境中才会生效，并且还需要进行一定的配置。如我在Openstack环境中创建的cluster，则需要参照[该文档](https://docs.openstack.org/magnum/ocata/dev/kubernetes-load-balancer.html)对Openstack进行配置，Openstack才能够支持kubernetes的Loadbalancer service。如未进行配置，通过命令查看Ingress External IP一直显示为pending状态。

```
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                            AGE
istio-ingress   LoadBalancer   10.111.158.10   <pending>     80:32765/TCP,443:31969/TCP                                         11m
istio-mixer     ClusterIP      10.107.135.31   <none>        9091/TCP,15004/TCP,9093/TCP,9094/TCP,9102/TCP,9125/UDP,42422/TCP   11m
istio-pilot     ClusterIP      10.111.110.65   <none>        15003/TCP,443/TCP                                                  11m
```

如未能配置云环境提供Loadbalancer, 我们可以直接使用集群中的一个节点IP:Nodeport访问Bookinfo应用程序。

```
http://10.12.5.31:32765/productpage
```
如果想要了解更多关于如何从集群外部进行访问的内容，可以参考[如何从外部访问Kubernetes集群中的应用？](http://zhaohuabing.com/2017/11/28/access-application-from-outside/)

## 参考

* [Service Mesh with Istio and NGINX](https://github.com/nginmesh/nginmesh/)

* [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#14-installing-kubeadm-on-your-hosts)

* [Kubernetes Reference Documentation-Dynamic Admission Control](https://kubernetes.io/docs/admin/extensible-admission-controllers/#enable-initializers-alpha-feature)


