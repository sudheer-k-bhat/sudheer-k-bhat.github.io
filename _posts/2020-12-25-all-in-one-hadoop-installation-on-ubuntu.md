---
layout: post
title:  All in one Hadoop installation on Ubuntu
categories: [Big Data]
excerpt: Hadoop installation on a Ubuntu VM. Components include Hadoop, Spark, Sqoop & Hive.
---

# Introduction
This article describes the process of setting up a single-node aka pseudonode aka all-in-one Hadoop.
Before getting started, ensure you have a Ubuntu 20.04.1 LTS running on a VM ready. Following hardware is recommended for the VM:
* 2 vCPUs
* 4GB RAM
* 50GB disk space

# Details
## Install Java
As Hadoop is Java based, installing JDK is a prerequisite.

```bash
sudo apt-get update
sudo apt-get install -y openjdk-8-jdk
echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> ~/.bashrc
echo "export JRE_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre" >> ~/.bashrc
```

## Install Hadoop
```bash
wget https://mirrors.estointernet.in/apache/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
sudo tar -xvf hadoop-3.2.1.tar.gz -C /opt/
cd /opt/
sudo mv hadoop-3.2.1 hadoop
sudo chown -R sudheer:sudheer hadoop
echo "export HADOOP_HOME=/opt/hadoop" >> ~/.bashrc
echo "export PATH=$PATH:/opt/hadoop/bin" >> ~/.bashrc
echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> /opt/hadoop/etc/hadoop/hadoop-env.sh
source ~/.bashrc
hadoop version
```

## Install Spark
```bash
cd ~
wget https://mirrors.estointernet.in/apache/spark/spark-3.0.1/spark-3.0.1-bin-hadoop3.2.tgz
sudo tar -xvf spark-3.0.1-bin-hadoop3.2.tgz -C /opt/
cd /opt/
sudo mv spark-3.0.1-bin-hadoop3.2 spark
sudo chown -R sudheer:sudheer spark
echo "export SPARK_HOME=/opt/spark" >> ~/.bashrc
echo "export PATH=$PATH:/opt/spark/bin" >> ~/.bashrc
source ~/.bashrc
spark-shell --version
```

## Configure
Here we configure Hadoop & Spark.
```bash
sudo echo '<?xml version="1.0" encoding="UTF-8"?><?xml-stylesheet type="text/xsl" href="configuration.xsl"?><configuration><property><name>fs.defaultFS</name><value>hdfs://localhost:9000</value></property></configuration>' > /opt/hadoop/etc/hadoop/core-site.xml
sudo echo '<?xml version="1.0" encoding="UTF-8"?><?xml-stylesheet type="text/xsl" href="configuration.xsl"?><configuration><property><name>dfs.datanode.data.dir</name><value>file:///opt/hadoop_tmp/hdfs/datanode</value></property><property><name>dfs.namenode.name.dir</name><value>file:///opt/hadoop_tmp/hdfs/namenode</value></property><property><name>dfs.replication</name><value>1</value></property></configuration>' > /opt/hadoop/etc/hadoop/hdfs-site.xml
sudo mkdir -p /opt/hadoop_tmp/hdfs/datanode
sudo mkdir -p /opt/hadoop_tmp/hdfs/namenode
sudo chown sudheer:sudheer -R /opt/hadoop_tmp
sudo echo '<?xml version="1.0"?><?xml-stylesheet type="text/xsl" href="configuration.xsl"?><configuration><property><name>mapreduce.framework.name</name><value>yarn</value></property><property><name>yarn.app.mapreduce.am.env</name><value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value></property><property><name>mapreduce.map.env</name><value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value></property><property><name>mapreduce.reduce.env</name><value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value></property></configuration>' > /opt/hadoop/etc/hadoop/mapred-site.xml
sudo echo '<?xml version="1.0"?><configuration><property><name>yarn.nodemanager.aux-services</name><value>mapreduce_shuffle</value></property><property><name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name><value>org.apache.hadoop.mapred.ShuffleHandler</value></property></configuration>' > /opt/hadoop/etc/hadoop/yarn-site.xml
```

## Setup SSH
This is required to communicate with Hadoop components viz. HDFS
```bash
sudo apt-get install -y openssh-server
ssh localhost
hdfs namenode -format -force
echo "export PATH=$PATH:/opt/hadoop/sbin" >> ~/.bashrc
source ~/.bashrc
ssh-keygen
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
> You can create password less SSH key for development purposes. Production environments should use a strong password.

## Start Hadoop & run a sample job
Verify the setup so far
```bash
start-dfs.sh && start-yarn.sh
jps
yarn jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar pi 1 50
```

## Install MySQL
This is optional. We will import data from MySQL to Hadoop using Sqoop. You can use any other SQL database.
```bash
sudo apt-get install -y mysql-server
sudo mysql --defaults-file=/etc/mysql/debian.cnf
```

## Install Sqoop
This is used to work with MySQL database installed above.
```bash
wget https://mirrors.estointernet.in/apache/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
lsb_release -a
sudo tar -xvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz -C /opt/
cd /opt/
sudo mv sqoop-1.4.7.bin__hadoop-2.6.0 sqoop
sudo chown -R sudheer:sudheer sqoop
echo "export SQOOP_HOME=/opt/sqoop" >> ~/.bashrc
echo "export PATH=$PATH:/opt/sqoop/bin" >> ~/.bashrc
source ~/.bashrc
cd /opt/sqoop/conf/
cp sqoop-env-template.sh sqoop-env.sh
echo "export HADOOP_COMMON_HOME=/opt/hadoop" >> sqoop-env.sh
echo "export HADOOP_MAPRED_HOME=/opt/hadoop" >> sqoop-env.sh
sqoop-version
```

## Fix sqoop
The commons-lang JAR that comes prepackaged with Sqoop doesn't work. So use commons-lang v2.6 JAR instead.
```bash
cd ~
wget https://repo1.maven.org/maven2/commons-lang/commons-lang/2.6/commons-lang-2.6.jar
cp commons-lang-2.6.jar /opt/sqoop/lib
```

## Install MySQL driver to sqoop
Sqoop talks to MySQL through this JDBC driver.
```bash
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java_8.0.22-1ubuntu20.04_all.deb
dpkg-deb -xv mysql-connector-java_8.0.22-1ubuntu20.04_all.deb newfolder
cp newfolder/usr/share/java/mysql-connector-java-8.0.22.jar /opt/sqoop/lib
```

## Install Hive
Hive is used to communicate with Hadoop in SQL lingo.
```bash
wget https://mirrors.estointernet.in/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
sudo tar -xvf apache-hive-3.1.2-bin.tar.gz -C /opt/
sudo mv /opt/apache-hive-3.1.2-bin /opt/hive
sudo chown -R sudheer:sudheer /opt/hive
echo "export HIVE_HOME=/opt/hive" >> ~/.bashrc
echo "export PATH=\$PATH:/opt/hive/bin" >> ~/.bashrc
source ~/.bashrc
cp /opt/hive/conf/hive-env.sh.template /opt/hive/conf/hive-env.sh
echo "export HADOOP_HOME=/opt/hadoop" >> /opt/hive/conf/hive-env.sh
echo "export HIVE_CONF_DIR=/opt/hive/conf" >> /opt/hive/conf/hive-env.sh
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chmod 775 /tmp
hdfs dfs -chmod 775 /user/hive/warehouse
cp /opt/hive/conf/hive-default.xml.template /opt/hive/conf/hive-site.xml
rm /opt/hive/lib/guava-19.0.jar
cp /opt/hadoop/share/hadoop/hdfs/lib/guava-27.0-jre.jar /opt/hive/lib
$HIVE_HOME/bin/schematool -initSchema -dbType derby
```

# Summary
We have installed & verified the installation of Hadoop 3.2.1 on Ubuntu VM.