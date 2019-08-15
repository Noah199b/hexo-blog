---
title: Zookeeper集群部署(Linux)
date: 2019-08-15 14:45:06
tags: 
	- Zookeeper集群
categories:
	- Zookeeper
---

## 准备

- 首先准备3个Linux虚拟机（CentOS 7）

## 环境搭建

### JDK1.8下载（Linux）及安装

https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

下载完成后将tar包拷入虚拟机 `/usr/local/src`下

```shell
# 解压jdk到/usr/local
tar -zxvf jdk-8u221-linux-x64.tar.gz -C ../
```

### Zookeeper 3.4.12 下载及安装

下载自己需要的版本：https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/  

下载完成后将tar包拷入虚拟机 `/usr/local/src`下

或者直接通过 wget 下载：
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
```

```shell
# 解压zookeeper到/usr/local并重命名
tar -zxvf zookeeper-3.4.12.tar.gz -C ../
cd ..
mv zookeeper-3.4.12 zookeeper
```

### 环境变量配置

```shell
vi /etc/profile
# 进入编辑模式
export JAVA_HOME=/usr/local/jdk1.8.0_221
export ZOOKEEPER_HOME=/usr/local/zookeeper
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$PATH
# 保存后执行使环境变量生效
source /etc/profile
```



<!-- more-->

## Zookeeper配置

### 重命名 zoo_sample.cfg
```shell
cd /usr/local/zookeeper/conf
# 重命名zoo_sample.cfg --> zoo.cfg
mv zoo_sample.cfg zoo.cfg
```

### 修改 zoo.cfg

```shell
# 如果没有该目录需要手动创建
dataDir=/usr/local/zookeeper/data
# 文件末尾添加3个节点(IP换成自己的集群)
server.0=192.168.204.131:2888:3888
server.1=192.168.204.132:2888:3888
server.2=192.168.204.133:2888:3888
```

### 创建服务器标识
```shell
cd /usr/local/zookeeper/data
# 创建文件myid
touch myid
# 将标识写入myid
echo 0 > myid
```

### 将此环境拷入剩余两个节点
```shell
scp -r /usr/local/zookeeper/ root@192.168.204.132:/usr/local/zookeeper
scp -r /usr/local/jdk1.8.0_221/ root@192.168.204.132:/usr/local/jdk1.8.0_221
scp -r /etc/profile root@192.168.204.132:/etc/profile
```

分别进入剩余两个节点：
```shell
# 环境变量生效
source /etc/profile
# 更改服务标识
echo 1 > /usr/local/zookeeper/data/myid
echo 2 > /usr/local/zookeeper/data/myid
```

## 启动Zookeeper

路径 `/usr/local/zookeeper/bin/`
```
# 集群都得启动
zkServer.sh start
# 查看节点状态
zkServer.sh status
```

## 启动报错解决方案

### 没有到主机的路由 (Host unreachable)
```shell
# 关闭防火墙
systemctl stop firewalld
# 查看防火墙状态
systemctl status firewalld
```

### 拒绝连接 (Connection refused)
启动的顺序是slave-01>slave-02>slave-03

由于ZooKeeper集群启动的时候，每个结点都试图去连接集群中的其它结点，先启动的肯定连不上后面还没启动的，所以上面日志前面部分的异常是可以忽略的。通过后面部分可以看到，集群在选出一个Leader后，最后稳定了。

其他结点可能也出现类似问题，属于正常。

```shell
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```
```shell
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```