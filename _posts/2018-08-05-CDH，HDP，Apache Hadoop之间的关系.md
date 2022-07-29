---
title: 'CDH，HDP，Apache Hadoop之间的关系'
date: 2018-08-05
permalink: /posts/2018/05/CDH，HDP，Apache Hadoop之间的关系/
tags:
  - Data Engineering
---

## 一、综述
目前Hadoop发行版非常多，有华为发行版、Intel发行版、Cloudera发行版（CDH）等，所有这些发行版均是基于Apache Hadoop衍生出来的，之所以有这么多的版本，完全是由Apache Hadoop的开源协议决定的：任何人可以对其进行修改，并作为开源或商业产品发布/销售。(http://www.apache.org/licenses/LICENSE-2.0)。
CDH全称是Cloudera 
国内绝大多数公司发行版是收费的，比如Intel发行版、华为发行版等，尽管这些发行版增加了很多开源版本没有的新feature，但绝大多数公司选择Hadoop版本时会将把是否收费作为重要指标，不收费的Hadoop版本主要有三个（均是国外厂商），分别是：
 - Cloudera版本（Cloudera’s Distribution Including Apache Hadoop，简称“CDH”）
 - Apache基金会hadoop
 - Hortonworks版本（Hortonworks Data Platform，简称“HDP”）

对于国内而言，绝大多数选择CDH版本。

## 二、社区版本与第三方发行版本的比较

### 1.Apache社区版本

优点：
    完全开源免费。
    社区活跃
    文档、资料详实
 
缺点：
----复杂的版本管理。版本管理比较混乱的，各种版本层出不穷，让很多使用者不知所措。
----复杂的集群部署、安装、配置。通常按照集群需要编写大量的配置文件，分发到每一台节点上，容易出错，效率低下。
----复杂的集群运维。对集群的监控，运维，需要安装第三方的其他软件，如ganglia，nagois等，运维难度较大。
----复杂的生态环境。在Hadoop生态圈中，组件的选择、使用，比如Hive，Mahout，Sqoop，Flume，Spark，Oozie等等，需要大量考虑兼容性的问题，版本是否兼容，组件是否有冲突，编译是否能通过等。经常会浪费大量的时间去编译组件，解决版本冲突问题。
 
### 2.第三方发行版本（如CDH，HDP，MapR等）

优点：
----基于Apache协议，100%开源。
----版本管理清晰。比如Cloudera，CDH1，CDH2，CDH3，CDH4等，后面加上补丁版本，如CDH4.1.0 patch level 923.142，表示在原生态Apache Hadoop 0.20.2基础上添加了1065个patch。
----比Apache Hadoop在兼容性、安全性、稳定性上有增强。第三方发行版通常都经过了大量的测试验证，有众多部署实例，大量的运行到各种生产环境。
----版本更新快。通常情况，比如CDH每个季度会有一个update，每一年会有一个release。
----基于稳定版本Apache Hadoop，并应用了最新Bug修复或Feature的patch
----提供了部署、安装、配置工具，大大提高了集群部署的效率，可以在几个小时内部署好集群。
----运维简单。提供了管理、监控、诊断、配置修改的工具，管理配置方便，定位问题快速、准确，使运维工作简单，有效。
 
缺点：
----涉及到厂商锁定的问题。（可以通过技术解决）

## 三、第三方发行版本的比较
Cloudera：最成型的发行版本，拥有最多的部署案例。提供强大的部署、管理和监控工具。Cloudera开发并贡献了可实时处理大数据的Impala项目。
Hortonworks：不拥有任何私有（非开源）修改地使用了100%开源Apache Hadoop的唯一提供商。Hortonworks是第一家使用了Apache HCatalog的元数据服务特性的提供商。并且，它们的Stinger开创性地极大地优化了Hive项目。Hortonworks为入门提供了一个非常好的，易于使用的沙盒。Hortonworks开发了很多增强特性并提交至核心主干，这使得Apache Hadoop能够在包括Windows Server和Windows Azure在内的Microsft Windows平台上本地运行。


## 四、CDH，Apache Hadoop，HDP的比较
|| Apache Hadoop| CDH | HDP |
|----| ------ | ------ | ------ |
|管理工具| 手工 | Cloudera Manager | Ambari |
|收费情况| 开源 | 社区版免费，企业版收费 | 免费 |