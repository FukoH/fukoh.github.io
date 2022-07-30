---
title: 'K8S,Spring Cloud Gateway,Nginx搭建随笔'
date: 2020-06-10
excerpt: 项目中遇到的实际情况
permalink: /posts/2020/06/K8S,Spring Cloud Gateway,Nginx搭建随笔/
tags:
  - BackEnd Develop
  - Java
  - DevOps
---


最近完成了基于k8s的springcloud搭建，记录下过程。

## K8S的组件介绍
### Pod
一个Pod是一组容器的集合,它们共享网络,我们的微服务注册中心是Consul,微服务的容器和Consul客户端的容器组成了一个Pod.这样微服务访问Consul客户端就像访问本地一样了.使用localhost就可以访问到Consul.然后Consul客户端再和服务端通信.
### Service
Service是Pod提供的服务的抽象,因为K8S的Pod的容器是不稳定的,可能会挂掉,这种情况下，K8S就会为我们重新启动POD，维持着指定的状态。Pod里的容器分配的IP是不同的，这就需要通过Service给不同的服务的IP绑定到一个Service名字上，以后通过Servie就能解析出对应的Pod的IP。在同一个K8S集群中，Service的name还充当着DNS-Name，需要在Pod-A里访问另一个Service里的Pod-B时（比如应用需要访问另一个Service提供的Redis的服务），使用Servie name就可以解析出名字。
### Deployment
Deployment控制RS，RS控制Pod，这一整套，向外提供稳定可靠的Service。至于什么是RS，笔者目前还没有深入了解，项目中这个概念也比较淡化，先搁置了。
### ConfigMap
ConfigMap可以把配置或者配置文件挂载到容器中，生命周期和Pod独立
### PersistVolume（PV）和PersisteVolumeClaim(PVC)
k8s中的存储系统，生命周期和容器独立的存储。可以声明把pvc挂载到容器的某一个目录。pv是由集群管理员管理的，我们不用管。
### Ingress
Ingress主要负责路由。项目由于是前后端分离，所有的请求都要先经过nginx，所以Ingress这一层需要先把所有请求都转发到nginx。有没有更好的路由方式笔者还不知道，需要继续学习。

## Nginx
请求转发到Nginx之后，Nginx继续对请求进行转发：如果请求的是页面，css，html这些静态资源，就直接在nginx服务器中访问；如果要访问RestApi，就对请求进行转发，转发到Spring cloud gateway里。
这里要注意的地方是localtion的配置，请求api需要在location那里加上proxy_pass ，server_name配置的是nginx的service的name。
这里我们对proxy_pass的location设计是使用了/webapi这个来进行动静分离。

## Spring cloud gateway
微服务的网关接到来自nginx的请求，会根据网关的application.yml中配置的转发规则进行转发。对/webapi开头的，会使用 -StripPrefix=1的fitler，这样网关转发到微服务的时候就会把/webapi去掉
