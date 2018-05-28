---
layout: post
title: "集群环境下配置Spark分布式计算平台"
date: 2016-03-14 21:35:27 +0800
comments: true
categories: 数据分析
tags: [spark, 大数据, 分布式, 集群]
keywords: spark, 大数据, 分布式, 集群
description: 集群环境下配置Spark分布式计算框架
---

**Spark** 是一个通用计算引擎，可用它来完成各种各样的运算，包括 SQL查询，文本处理，机器学习等，它可以运行在单台计算机上，也可以运行在集群上，本文主要介绍如何在集群环境中配置**Spark**。


## 1. 运行环境说明

实验环境说明

-   服务器配置：8G内存，$140G\times6$硬盘容量
-   操作系统：`CentOS 6.7`
-   操作系统软件包：Basic Server ( 常用的程序如 ssh Python2.x 等)

在单台服务器上，可以通过`libvirt`管理若干台虚拟机（kvm, kernel-based virtual machine），各个虚拟机是一个独立的，完整的操作系统，可作为分布式环境下的工作节点。

本文通过在单台服务器上构建5台虚拟机，搭建**Spark**运行的集群环境。

下面是Spark运行时的组件结构

{% img center /images/分布式Spark应用中的组件.png %}

本文的虚拟机环境如下

{% img center /images/虚拟机结构.png %}

为了方便起见，分别为五台虚拟机分配如下的ip地址(**NAT方式配置网络**)，vnc端口，HostName.

{% img center /images/kvm_table.png %}

在机房配置服务器和网络
{% img center /images/in_machine_room.jpg 300 400 %}

关于虚拟机的创建以及网络配置，请自行Google之，创建好五台虚拟机之后，将五台虚拟机启动，`virsh list`查看当前运行的虚拟机。

{% img center /images/virsh-list.png %}

## 2. 在集群上安装Spark

**Spark**应用将分布式环境中的节点分为驱动器节点(`Master`)和执行器节点(`Slave`)。

驱动器执行程序的`main()`方法，执行用户编写的用来创建`SparkContext`,　创建`RDD`，以及进行`RDD`的转化操作和行动操作的代码。它主要负责将用户程序转化为任务(**物理执行单元**)，并为执行器节点调度任务。驱动器程序一旦终止，Spark应用也就结束了。

执行器节点是一种工作进程，负责在Spark作业中运行任务，这里的任务就是上一段中的**物理执行单元**，这些任务相互独立。当驱动器节点启动时，执行器节点也同时启动，并且伴随整个 Spark 应用的生命周期。如果某个执行器节点出现故障，Spark应用也可以继续运行。执行器节点通过自身的`Block Manager`为用户程序中要求缓存的`RDD`提供内存式存储。由此可见, `RDD`是直接缓存在执行器进程内的，因此任务可以在运行时充分利用缓存数据加速运算。

我们需要选定一个驱动器节点和多个执行器节点，在这里，我们选择**宿主机**为执行器节点, 五台虚拟机为执行器节点，我们先在宿主机上安装`Spark`。

在生产环境中, Spark 主要部署在`linux`集群中，机器的数量可达上千台，这些`linux`机器需要预先安装有`JDK`，`Scala`等所需的依赖。安装过程不再赘述。

下载`Spark`，访问[Spark官网](http://spark.apache.org/downloads.html)下载最新版本，然后我们得到一个压缩的`tar`文件。将其解压到合适的目录（比如`/usr/local/spark/`），重命名为`spark`。

{% img center /images/spark_usr_local.png %}

接下来需要配置 `conf/spark-env`文件，这里只配置基本的参数，更详尽的配置参数参见[官网说明](http://spark.apache.org/docs/latest/configuration.html), 在`conf/spark-env`文件中加入下列配置:

```
export SCALA_HOME=${SCALA_HOME} ＃SCALA环境变量
export SPARK_WORKING_MEMORY=2g #每一个worker节点上可用的最大内存
export SPARK_MASTER_IP=192.168.122.3 #驱动器节点IP
export MASTER=spark://192.168.122.3:7077
```

配置`slaves`文件，本文中五个虚拟机为`worker`节点，将节点的主机名（`HostName`）加入`slaves`文件中。

```
spark1
spark2
spark3
spark4
spark5
```

配置完成之后，将驱动器节点上的 `Spark`目录分发到各个虚拟机上。

接下来要设置好从驱动器节点到其他工作节点的`SSH`无密码登陆(你肯定不想每次连接工作节点都自己手动输入密码)。这需要在所有机器上有相同的用户账号，并在驱动器节点上通过`ssh-keygen`生成`SSH`公钥，然后将这个公钥放到所有节点的`.ssh/authorized_keys`文件中。

在驱动器节点上: 运行`ssh-keygen`并接受默认选项。

{% img center /images/ssh-keygen.png %}

将驱动器节点上的 `~/.ssh/id_dsa.pub`文件复制至工作节点上，然后将其追加到`.ssh/authorized_keys`文件中，并设置`644`权限, 下图以工作节点`spark1`为例。

{% img center /images/cat_spark1.png %}

对其他节点也执行同样操作，使得驱动器节点到其他工作节点可以进行`SSH`无密码访问。

在驱动器节点上，切换至`Spark`目录，运行`./sbin/start-all.sh`启动集群，`./sbin/stop-all.sh`关闭集群。

启动集群后，通过本机的`8080`端口查看当前集群状态，如下图

{% img center /images/spark_web_ui.png %}

要向独立集群管理器提交应用，需要把`spark://masternode:7077`作为主节点参数传给`spark-submit`，假定你的应用为`app.py`，提交应用时执行

```
spark-submit --master spark://masternode:7077 app.py
```

