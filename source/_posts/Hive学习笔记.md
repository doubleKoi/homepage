---
title: Hive学习笔记
date: 2019-03-21 00:16:47
tags:
- 数据仓库
- hadoop
- hive
---

下周《数据仓库》的课程展示，我被分配到的题目是Hive，索性就边准备边学习一下Hive的相关知识。

### 1. Hive是什么

（略过，等我交完毕业论文再写）

~~（所以说我为什么在论文ddl两周前还有心情写博客啊orz）~~

### 2. Hive在Linux环境下的安装

安装过程简直是一步一个坑，心累。

#### 2.1 安装Java

首先要确定自己已经安装了Java，使用`java --version`命令来查看自己的Java版本。至于怎么安装就不在这里赘述了。

注意最好安装1.8版本的JDK，9/10/11中某些类被禁用/移除了，之后会影响HDFS的使用。

如果是操作系统自带的OpenJDK，可以输入`jsp`安装jsp功能相关包，便于后期使用。

#### 2.2 安装Hadoop

首先下载`hadoop-3.2.0.tar.gz`。官方速度堪忧，推荐[北理工镜像站](http://mirror.bit.edu.cn/apache/hadoop/common/stable/)。

下载完成后解压并拷贝到相关目录：

```bash
$ tar xzf hadoop-3.2.0.tar.gz
$ sudo mkdir /usr/local/hadoop
$ sudo mv hadoop-3.2.0/* to /usr/local/hadoop/
```

下面进行安装和配置。Hadoop的安装有三种模式：本地模式(Local/Standalone Mode)，伪分布式模式(Pseudo-Distributed Mode)，分布式模式(Fully-Distributed Mode)，学习Hive一般使用伪分布式（据说，那么在这里我们就采取伪分布式模式进行安装。

首先配置环境变量。打开`~/.bashrc`文件，加入如下内容：

```bash
export HADOOP_HOME=/usr/local/hadoop 
export HADOOP_MAPRED_HOME=$HADOOP_HOME 
export HADOOP_COMMON_HOME=$HADOOP_HOME 
export HADOOP_HDFS_HOME=$HADOOP_HOME 
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native 
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```

刷新`.bashrc`文件以应用到系统：

```bash
$ source ~/.bashrc
```

 接下来对Hadoop的系统配置文件（在`$HADOOP_HOME/etc/hadoop`目录下）进行一系列的配置。

- 在`hadoop-env.sh`文件中指定JDK所在目录：

  对于oracleJDK来说一般在`/usr/java`文件夹下，而坑爹的openJDK则在`/usr/lib/jvm/java-11-openjdk-amd64`(请根据不同版本自行查找)。

  ```bash
  export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
  ```

- 配置`core-site.xml`文件：

  在`<configuration></configuration>`标签之间加入以下内容：

  ```xml
  <property>
      <name>fs.defaultFS</name>
      <value>hdfs://localhost:9000</value>
  </property>
  ```

- 配置``hdfs-site.xml`文件：

  在`<configuration></configuration>`标签之间加入以下内容：

  ```xml
  <property>
      <name>dfs.replication</name>
      <value>1</value>
  </property>
  ```

此外还需要安装ssh客户端：

```bash
$ sudo apt-get install openssh-server
```

并配置无需密码即可链接：

```bash
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```



#### 2.3 验证并启动Hadoop

首先格式化文件系统

```bash
$ cd /usr/local/hadoop
$ bin/hdfs namenode -format
```

启动文件系统

```bash
$ sbin/start-dfs.sh
```

然后在在浏览器中输入`http://localhost:9870`即可访问hdfsWeb，点击Utilities - Browse the file system进行文件管理。

这个时候，Java 9/10/11用户就会看到报错。

Java 9/10请在`hadoop-env.sh`中加入

```bash
export HADOOP_OPTS="${HADOOP_OPTS} --add-modules java.activation "
```

Java 11用户请下载[java.activation](https://jcenter.bintray.com/javax/activation/javax.activation-api/1.2.0/javax.activation-api-1.2.0.jar)并把它放在`$HADOOP_HOME/share/hadoop/common`中。

**到这里hadoop的安装就算大功告成惹！**

如果过程中有任何permission denied的问题请修改用户权限 。

#### 2.4 安装hive

（未完待续...