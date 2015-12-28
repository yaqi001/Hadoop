# 单点设置

目录：
- [目的](#purpose)
- [前提准备](#prerequisites)
   - [支持的平台](#platforms)
   - [必备软件](#requiredsoftware)
   - [安装软件](#installing)
- [下载](#download)
- [准备开启 Hadoop 集群](#starting)
- [独立式操作](#standalone)
- [伪分布式操作](#pseudo)
   - [配置](#configuration)
   - [设置无密码 ssh 登录](#passphraseless)
   - [执行](#execution)
   - [单结点上的 YARN](#yarn)
- [全分布式操作](#fully)

<h2 id="purpose">目的</h2>
该文档意通过教授如何安装并配置单节点的 Hadoop 安装来让你快速的使用 Hadoop 的 MapReduce 和 Hadoop 分布式文件系统（HDFS）执行一些简单的操作。

<h2 id="prerequisites">前提准备</h2>
<h3 id="platform">支持平台</h3>
* GNU/Linux 可以作为 Hadoop 的开发平台以及生产平台。目前，Hadoop 已经可以在拥有了 2000 台节点的集群上进行演示了。
* 虽然 Windows 也同样被支持，但不在此作更多介绍。

<h3 id="requiredsoftware">所需要的软件</h3>
* 安装必备的包
   * Java，支持的[版本](http://wiki.apache.org/hadoop/HadoopJavaVersions)
   * ssh，开启 ssh 服务后，管理远程 Hadoop 后台的 Hadoop 脚本才能被正常使用。

* 当前环境
   ~~~ bash
   [helen@yingyun workspace]$ uname -r
   3.10.0-327.3.1.el7.x86_64
   [helen@yingyun workspace]$ java -version
   java version "1.7.0_91"
   OpenJDK Runtime Environment (rhel-2.6.2.3.el7-x86_64 u91-b00)
   OpenJDK 64-Bit Server VM (build 24.91-b01, mixed mode)
   ~~~
	 
* 安装软件
   * 
   * yum -y install
   * rsync（用于多节点）

2. 

