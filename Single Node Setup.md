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
   * `yum -y install ssh`
   * `yum -y install rsync`

<h2 id="starting">准备开启 Hadoop 集群</h2>
下载并解压 Hadoop 发行版，编辑 `$HADOOP_HOME/etc/hadoop/hadoop-env.sh` 文件，添加如下参数：
~~~ bash
# 我安装的 Java 在 /usr/lib/jvm/ 目录下
export JAVA_HOME=/usr/lib/jvm/java-1.7.0
~~~

然后执行下面的指令：
~~~ bash
$ bin/hadoop
~~~

通过执行上面的指令，你可以查看 Hadoop 脚本的使用文档。

现在有三种开启 Hadoop 集群的方式：
* 本地（即单机）模式
* 伪分布式模式
* 全分布式模式

<h2 id="standalone">单机操作</h2>
默认情况下，Hadoop 是作为一个单一的 Java 进程，并在非分布式模式下运行的，调试起来很有用。

下面这个例子的大概意思是这样的：拷贝所有以 .xml 为后缀的文件作为输入，然后将所有文件名和已知正则表达式相匹配的文件输出到给出的输出目录下。

~~~ bash
[helen@yingyun hadoop-2.7.1]$ mkdir ~/input # 注意：这里要记住 input 所在的目录，我将 input 目录放到了用户目录下
[helen@yingyun hadoop-2.7.1]$ cp etc/hadoop/*.xml ~/input
[helen@yingyun hadoop-2.7.1]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep ~/input ~/output 'dfs[a-z.]+'
[helen@yingyun hadoop-2.7.1]$ cat ~/ouptut/*
~~~

<h2 id="pseudo">伪分布式操作</h2>
Hadoop 还可以在单机上以伪分布式模式运行起来，每个 Hadoop 后台分别运行在 Java 进程中。

* 配置
  * 在 `etc/hadoop/core-site.xml` 文件中写入如下内容：
     ~~~ bash
     <configuration>
         <property>
             <name>fs.defaultFS</name>
             <value>hdfs://localhost:9000</value>
         </property>
     </configuration>
     ~~~
     > **注意：**：value 标签中只能写 localhost，不要写改机器的 ip，主机名等等。

  * 在 `etc/hadoop/hdfs-site.xml` 文件中写入如下内容：
     ~~~ bash
     <configuration>
         <property>
             <name>dfs.replication</name>
             <value>1</value>
         </property>
     </configuration>
     ~~~

* 设置本机的 ssh 无密码登录
  
  ~~~ bash
  [helen@yingyun hadoop-2.7.1]$ ssh-keygen
  [helen@yingyun hadoop-2.7.1]$ ssh-copy-id -i ~/.ssh/id_rsa.pub helen@192.168.9.56 
  ~~~

* **执行**：
  下面的命令用于在本地运行 MapReduce job。如果你想在 YARN 上执行 MapReduce job，见[单结点上的 YARN](#yarn)。

  1. 格式化文件系统：
     ~~~ bash
     [helen@yingyun hadoop-2.7.1]$ bin/hdfs namenode -format
     ~~~

  2. 开启 NameNode 后台和 DataNode 后台程序：
     ~~~ bash
     [helen@yingyun hadoop-2.7.1]$ sbin/start-dfs.sh
     ~~~
    
  3. 现在可以浏览 NameNode 的网页了，默认是在：
     ~~~ bash
     http://localhost:50070/
     ~~~

  4. 创建一个用于执行 MapReduce job 的 HDFS 目录：
     ~~~ bash
     [helen@yingyun hadoop-2.7.1]$ bin/hdfs dfs -mkdir /user
     [helen@yingyun hadoop-2.7.1]$ bin/hdfs dfs -mkdir /user/qiqi
     ~~~

  5. 将输入文件拷贝到分布式文件系统中：
     ~~~ bash
     # 注意：下面这种写法是将 input 目录放在了 /user/qiqi 目录下。
     [helen@yingyun hadoop-2.7.1]$ bin/hdfs dfs -put etc/hadoop /user/qiqi/input
     ~~~ 
     ~~~ bash
     # 当然，如果我们只写 input 的话，会将 input 目录放置在 /{$local_username}/ 目录下。
     [helen@yingyun hadoop-2.7.1]$ /bin/hdfs dfs -put etc/hadoop input
     ~~~

  6. 运行一些示例：
     ~~~ bash
     [helen@yingyun hadoop-2.7.1]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep /user/qiqi/input /user/qiqi/output 'dfs[a-z.]+'
     ~~~

  7. 查看输出文件：
     * 第一种方式是将分布式文件系统中的 output 目录拷贝到本机的文件系统中，并查看是否正确：
       ~~~ bash
       [helen@yingyun hadoop-2.7.1]$ bin/hdfs dfs -get /user/qiqi/output ~/output
       [helen@yingyun hadoop-2.7.1]$ cat ~/output/*
       ~~~
  
     * 另一种方式是直接在分布式文件系统中查看输出：
       ~~~ bash
       [helen@yingyun hadoop-2.7.1]$ bin/hdfs dfs -cat /user/qiqi/output/*
       6  dfs.audit.logger
       4  dfs.class
       3  dfs.server.namenode.
       2  dfs.period
       2  dfs.audit.log.maxfilesize
       2  dfs.audit.log.maxbackupindex
       1  dfsmetrics.log
       1  dfsadmin
       1  dfs.servers
       1  dfs.replication
       1  dfs.file
       ~~~

  8. 关闭 NameNode 和 DataNode 后台程序：
     ~~~ bash
     [helen@yingyun hadoop-2.7.1]$ sbin/stop-dfs.sh
     ~~~
   
     
