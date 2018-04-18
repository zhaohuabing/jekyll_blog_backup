---
layout:     post
title:      "Helm：强大的Kubernetes包管理工具"
subtitle:   ""
description: ""
date:       2018-04-16 15:00:00
author:     "赵化冰"
header-img: "img/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
published: true
tags:
    - Kubernetes
    - Helm
---

## 目录
{:.no_toc}

* 目录
{:toc}

## Kubernetes应用部署的挑战
让我们首先来看看Kubernetes，kubernetes提供了基于容器的应用集群管理，为容器化应用提供了部署运行、资源调度、服务发现和动态伸缩等一系列完整功能。

kubernetes的核心设计理念是: 用户定义应用程序的规格，而kubernetes则负责按照定义的规则部署并运行应用程序，如果应用系统出现问题导致偏离了定义的规格，kubernetes负责对其进行自动修正。例如应用规格要求部署两个实例，其中一个实例异常终止了，kubernetes会检查到并重新启动一个新的实例。

用户通过使用kubernetes API对象来描述应用程序规格，包括Pod，Service，Volume，Namespace，ReplicaSet，Deployment，Job等等。一般这些对象需要写入一系列的yaml文件中，然后通过kubernetes命令行工具kubectl进行部署。

以下面的wordpress应用程序为例，涉及到多个kubernetes API对象，这些kubernetes API对象分散在多个yaml文件中。

图1： Wordpress应用程序中涉及到的kubernetes API对象
![](\img\in-post\2018-04-16-using-helm-to-deploy-to-kubernetes\wordpress.png)


我们面临的问题是：如何管理，编辑和更新这些这些分散的kubernetes应用配置文件？如何把一套的相关配置文件作为一个应用进行管理？如何分发和重用kubernetes的应用配置？

Helm的引入很好地解决上面这些问题。

## Helm是什么？
使用过Linux系统的同学一定很熟悉CentOS的yum或者Ubuntu的ap-get,这两者都是Linux系统下的包管理工具。采用apt-get/yum,应用开发者可以管理应用包之间的依赖关系，发布应用；用户则可以以简单的方式查找、安装、升级、卸载应用程序。

我们可以将Helm看作Kubernetes下的apt-get/yum。Helm是Deis (https://deis.com/) 开发的一个用于kubernetes的包管理器。对于应用发布者而言，可以通过Helm打包应用，管理应用依赖关系，管理应用版本并发布应用到软件仓库。对于使用者而言，使用Helm后不用需要了解Kubernetes的Yaml语法并编写应用部署文件，可以通过Helm下载并在kubernetes上安装需要的应用。Helm还以chart的方式提供了部署，删除，升级，回滚应用的强大功能。

## Helm组件及相关术语
* Helm Kubernetes的应用打包工具，也是命令行工具的名称。
* Tiller Helm的服务端，部署在Kubernetes集群中，用于处理Helm的相关命令。
* Chart Helm的打包格式，内部包含了一组相关的kubernetes资源。
* Repoistory Helm repository是一个web服务器，提供了该服务中所有chart的清单，并保存了chart文件以供下载。
* Release 使用Helm install命令在Kubernetes集群中安装的Chart称为Release。

>  和我们通常概念中的版本有所不同，这里的Release可以理解为Helm使用Chart包部署的一个应用实例。其实Release叫做Deployment更合适。估计因为Deployment这个概念已经被Kubernetes使用了，因此Helm才采用了Release这个术语。

图2： Helm软件架构
![](\img\in-post\2018-04-16-using-helm-to-deploy-to-kubernetes\helm-architecture.png)

## 安装Helm

下面我们通过一个完整的示例来介绍Helm的相关概念，并学习如何使用Helm打包，分发，安装，升级及回退kubernetes应用。

可以参考Helm的帮助文档https://docs.helm.sh/using_helm/#installing-helm 安装Helm

采用二进制的方式安装Helm

1. 下载 Helm https://github.com/kubernetes/helm/releases
1. 解压 tar -zxvf helm-v2.0.0-linux-amd64.tgz
1. 拷贝到bin目录 mv linux-amd64/helm /usr/local/bin/helm

然后使用下面的命令安装服务器端组件Tiller

```bash
Helm init
```

## 构建一个Helm chart

让我们在实践中来了解Helm。这里将使用一个Go测试小程序，让我们先为这个小程序创建一个Helm chart。

```
git clone https://github.com/daemonza/testapi.git; cd testapi
```

首先创建一个chart的骨架
```
helm create testapi-chart
```

该命令创建一个testapi-chart目录，主要关注目录中的这三个文件即可: Chart.yaml，values.yaml 和 NOTES.txt。

* Chart.yaml 用于描述这个chart，包括名字，描述信息以及版本。
* values.yaml 用于存储templates目录中模板文件中用到的变量。 模板文件一般是Go模板。如果你需要了解更多关于Go模板的相关信息，可以查看Hugo (https://gohugo.io) 的一个关于Go模板的介绍 (https://gohugo.io/templates/go-templates/)。
* NOTES.txt 用于向部署该chart的用于介绍chart部署后的一些信息。例如介绍如何使用这个chart，列出缺省的设置等。

打开Chart.yaml, 填写你部署的应用的详细信息，以testapi为例：
```
apiVersion: v1
description: A simple api for testing and debugging
name: testapi-chart
version: 0.0.1
```
然后打开并根据需要编辑values.yaml。下面是testapi应用的values.yaml文件内容。

```
replicaCount: 2
image:
  repository: daemonza/testapi
  tag: latest
  pullPolicy: IfNotPresent
service:
  name: testapi
  type: ClusterIP
  externalPort: 80
  internalPort: 80
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

在 testapi_chart 目录下运行下面命令以对chart进行校验。

```
 helm lint
```

如果一切正常，可以使用下面的命令对chart进行打包：

```
helm package testapi-chart --debug
```

这里添加了 --debug 参数来查看打包的输出，输出应该类似于：

```
Saved /Users/daemonza/testapi/testapi-chart/testapi-chart-0.0.1.tgz to current directory
Saved /Users/daemonza/testapi/testapi-chart/testapi-chart-0.0.1.tgz to /Users/daemonza/.helm/repository/local
```

chart被打包为一个压缩包testapi-chart-0.0.1.tgz，该压缩包被放到了当前目录下，并同时被保存到了helm的本地缺省仓库目录中。

## Helm Repository
通过Helm search命令，会发现找不到刚才生成的chart包。
```
helm search testapi
No results found
```

这是因为本地的repository目录中的chart还没有被加入Helm中。我们可以在本地启动一个Repository Server并将其加入到Helm repo列表中。

通过helm repo list命令可以看到目前helm中只配置了一个名为stable的repo，该repo指向了google的一个服务器。
```Bash
helm repo list
NAME    URL
stable  https://kubernetes-charts.storage.googleapis.com
```

使用helm serve命令启动一个repo server，该server缺省使用'$HELM_HOME/repository/local'目录作为chart存储，并在8879端口上提供服务。

```Bash
helm serve&
Now serving you on 127.0.0.1:8879
```
启动本地repo server后，将其加入helm的repo列表。
```Bash
helm repo add local http://127.0.0.1:8879
"local" has been added to your repositories
```

现在再查找testapi chart包，就可以找到了。

```Bash
helm search testapi

NAME                    CHART VERSION   APP VERSION     DESCRIPTION
local/testapi-chart     0.0.1                           A Helm chart for Kubernetes
```

## 在kubernetes中部署Chart
chart被发布到仓储后，可以通过Helm instal命令部署chart，部署时指定chart名及Release（部署的实例）名：
```
 helm install local/testapi-chart --name testapi
```
该命令的输出应类似:

```
NAME:   testapi
LAST DEPLOYED: Mon Apr 16 10:21:44 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                   TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)  AGE
testapi-testapi-chart  ClusterIP  10.43.121.84  <none>       80/TCP   0s

==> v1beta1/Deployment
NAME                   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
testapi-testapi-chart  1        1        1           0          0s

==> v1/Pod(related)
NAME                                   READY  STATUS   RESTARTS  AGE
testapi-testapi-chart-9897d9f8c-nn6wd  0/1    Pending  0         0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=testapi-testapi-chart" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

使用下面的命令列出所有已部署的Release以及其对应的Chart。
```
helm ls
```

该命令的输出应类似:
```
NAME    REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
testapi 1               Mon Apr 16 10:21:44 2018        DEPLOYED        testapi-chart-0.0.1     default
```

可以看到在输出中有一个Revision（更改历史）字段，该字段用于表示某一Release被更新的次数，可以用该特性对已部署的Release进行回滚。

## 升级和回退
修改Chart.yaml，将版本号从0.0.1 修改为 1.0.0, 然后使用Helm package命令打包并发布到本地仓库。

查看本地库中的Chart信息，可以看到在本地仓库中testapi-chart有两个版本

```Bash
helm search testapi -l
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
local/testapi-chart     0.0.1                           A Helm chart for Kubernetes
local/testapi-chart     1.0.0                           A Helm chart for Kubernetes
```

现在用helm upgrade将已部署的testapi升级到新版本。可以通过参数指定需要升级的版本号，如果没有指定版本号，则缺省使用最新版本。

```
helm upgrade testapi local/testapi-chart 
```

已部署的testapi release被升级到1.0.0版本

```Bash
helm list
NAME    REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
testapi 2               Mon Apr 16 10:43:10 2018        DEPLOYED        testapi-chart-1.0.0     default
```

可以通过Helm history查看一个Release的多次更改。

```Bash
helm history testapi
REVISION        UPDATED                         STATUS          CHART                   DESCRIPTION
1               Mon Apr 16 10:21:44 2018        SUPERSEDED      testapi-chart-0.0.1     Install complete
2               Mon Apr 16 10:43:10 2018        DEPLOYED        testapi-chart-1.0.0     Upgrade complete
```
如果更新后的程序由于某些原因运行有问题，我们则需要回退到旧版本的应用，可以采用下面的命令进行回退。其中的参数1是前面Helm history中查看到的Release的更改历史。

```Bash
helm rollback testapi 1
```

使用Helm list命令查看，部署的testapi的版本已经回退到0.0.1
```Bash
helm list
NAME    REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
testapi 3               Mon Apr 16 10:48:20 2018        DEPLOYED        testapi-chart-0.0.1     default
```
## 总结
Helm作为kubernetes应用的包管理以及部署工具，提供了应用打包，发布，版本管理以及部署，升级，回退等功能，是kubernetes应用管理的一个强力的增强和补充。

## 参考

[Using Helm to deploy to Kubernetes](https://daemonza.github.io/2017/02/20/using-helm-to-deploy-to-kubernetes/)
[Helm documentation](https://docs.helm.sh/helm/)
[Helm - Application deployment management for Kubernetes](https://www.slideshare.net/alexLM/helm-application-deployment-management-for-kubernetes)

