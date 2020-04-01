# Hive

## Install

-- Hive需要hadoop（需要里面的hdfs）才能使用

-- 为了简化使用，不另外新建专门的用户管理hadoop，直接使用本地登陆用户配置，该用户具有sudo权限，用户名使用<username>代替

### 1. 安装 hadoop 2.x.x （以 hadoop 2.9.2 为例）

[download hadoop](https://downloads.apache.org/hadoop/common/)

1. hadoop需要Java环境，推荐使用Java 8

```bash
sudo apt update
sudo apt install openjdk-8-jdk
```

建议添加环境变量
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

2. 下载解压hadoop，并放到合适的位置（以/opt为例）

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
sudo tar -xf hadoop-2.9.2.tar.gz -C /opt
cd /opt
sudo ln -s hadoop-2.9.2 hadoop
```

3. 配置hadoop中的java环境

```bash
cd /opt/hadoop
sudo nano etc/hadoop/hadoop-env.sh
```

修改 `export JAVA_HOME=${JAVA_HOME}` 为 `export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`

测试环境配置
```bash
bin/hadoop
```

建议添加环境变量
```bash
export HADOOP_HOME=/opt/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib/native"
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_INSTALL=$HADOOP_HOME
```

4. 独立模式运行

运行测试

```bash
cd ~
mkdir input
cp /opt/hadoop/etc/hadoop/*.xml input
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar grep input output 'dfs[a-z.]+'
cat output/*
```

解释：第四行代码运行输出运行日志，没有报错即可，第五行代码能输出运行结果，每行格式如 `1	dfsadmin`

5. 伪分布式运行

伪分布式运行首先需要配置ssh密钥登陆自身，使得程序运行可无需密码。本说明文档使用登陆用户运行hadoop，不再另外新建用户。

* 配置ssh密钥
```bash
ssh-keygen
# 一路回车进行默认配置即可
ssh-copy-id localhost
# 输入用户密码
```

测试密钥登陆，不需要输密码，先`ssh localhost`看看能否正常登陆，登陆成功的话，输入`exit`退出

* 预先建立文件夹用来存放数据
```
cd ~
mkdir -p dataset/hadoop/dfs
cd dataset/hadoop/dfs
mkdir tmp name data namesecondary
```

* 对hadoop进行一些设置（更多配置选项参考[官方文档](https://hadoop.apache.org/docs/r2.9.2/hadoop-project-dist/)）

编辑 /opt/hadoop/etc/hadoop/core-site.xml，将其中configuration标签对的内容修改为，
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:///home/<username>/dataset/hadoop/dfs/tmp</value>
    </property>
</configuration>
```

解释说明：（参考[官方对配置文件的参数解释](https://hadoop.apache.org/docs/r2.9.2/hadoop-project-dist/hadoop-common/core-default.xml)）

> 配置`fs.defaultFS`（默认文件系统的名称）为`hdfs://localhost:9000`

> 配置了`hadoop.tmp.dir`（其他临时目录的基础）为`file:///home/<username>/dataset/hadoop/dfs/tmp`

编辑 /opt/hadoop/etc/hadoop/hdfs-site.xml，将其中configuration标签对的内容修改为
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/<username>/dataset/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/<username>/dataset/hadoop/dfs/data</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///home/<username>/dataset/hadoop/dfs/namesecondary</value>
    </property>
</configuration>
```

解释说明：（参考[官方对配置文件对参数解释](https://hadoop.apache.org/docs/r2.9.2/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)）

> 配置`dfs.replication`（默认数据块的复制次数）为`1`
> 
> 配置`dfs.namenode.name.dir`（确定DFS名称节点在本地文件系统上存储名称表的位置）为`file:///home/<username>/dataset/hadoop/dfs/name`
> 
> 配置`dfs.datanode.data.dir`（确定DFS数据节点在本地文件系统上存储数据块的位置）为`file:///home/<username>/dataset/hadoop/dfs/data`
> 
> 配置`dfs.namenode.checkpoint.dir`（确定DFS辅助名称节点在本地文件系统上存储要合并的临时映像的位置）为`file:///home/<username>/dataset/hadoop/dfs/data`

编辑 /opt/hadoop/etc/hadoop/hadoop-env.sh，添加`export HADOOP_LOG_DIR=/home/<username>/dataset/hadoop/logs`到末尾

* 测试

```bash
cd ~
hdfs namenode -format                                     # 格式化数据节点
start-dfs.sh                                              # 启动dfs文件系统
hdfs dfs -mkdir -p /user/<username>                       # 在dfs文件系统中建立用户文件夹
hdfs dfs -put /opt/hadoop/etc/hadoop /user/monk/input     # 复制文件夹到dfs文件系统中
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar grep input output 'dfs[a-z.]+'
                                                          # 使用hadoop运行程序
hdfs dfs -cat output/*                                    # 查看运行结果
```

第2行代码执行后，正常情况下输出格式化日志，无报错。第6行代码运行效果如同独立模式第4行运行效果。

执行完第3行代码后，通过`jps`命令可查看到 NameNode、DataNode、SecondaryNameNode进程

通过`http://<ip address>:50070/`可查看节点状态。

关闭dfs文件系统使用`stop-dfs.sh`

6. 模拟分布式运行

编辑 /opt/hadoop/etc/hadoop/yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

复制/opt/hadoop/etc/hadoop/mapred-site.xml.template为/opt/hadoop/etc/hadoop/mapred-site.xml并编辑
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

编辑/opt/hadoop/etc/hadoop/yarn-env.sh，添加`export YARN_LOG_DIR=/home/<username>/dataset/hadoop/logs`到末尾

测试，测试之前要先使用`start-dfs.sh`打开dfs文件系统
```bash
start-yarn.sh
```
执行完代码后，通过`jps`命令可查看到 NameNode、DataNode、SecondaryNameNode、ResourceManager、NodeManager进程

打开`http://<ip address>:8088/`查看yarn服务状态
