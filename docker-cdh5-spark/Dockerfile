FROM centos:centos6


RUN yum -y update; yum -y clean all

##### Install JDK 1.7 #####
RUN yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel

##### Install basic linux tools #####
RUN yum install -y wget unzip dialog curl sudo lsof vim axel telnet



##### Add Cloudera CDH 5 repository #####
RUN wget http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/cloudera-cdh5.repo -O /etc/yum.repos.d/cloudera-cdh5.repo
RUN rpm --import http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/RPM-GPG-KEY-cloudera
#####



##### Install HDFS services #####
RUN yum install -y hadoop-hdfs-namenode hadoop-hdfs-secondarynamenode hadoop-hdfs-datanode

##### Install YARN services #####
RUN yum install -y hadoop-yarn-resourcemanager hadoop-yarn-nodemanager hadoop-yarn-proxyserver

##### Install MapReduce services #####
RUN yum install -y hadoop-mapreduce hadoop-mapreduce-historyserver

##### Install Hadoop client & Hadoop conf-pseudo #####
RUN yum install -y  hadoop-client hadoop-conf-pseudo

##### Install Zookeeper #####
RUN yum install -y zookeeper zookeeper-server

##### Install HBase #####
RUN yum install -y hbase-master hbase hbase-thrift

##### Install Oozie #####
RUN yum install -y oozie oozie-client

##### Install Spark #####
RUN yum install -y spark-core spark-history-server spark-python

##### Install Hive #####
RUN yum install -y hive hive-metastore hive-hbase

##### Install Pig #####
#RUN yum install -y pig

##### Install Impala #####
RUN yum install -y impala impala-server impala-state-store impala-catalog impala-shell

##### Install Hue #####
RUN yum install -y hue hue-server

##### Install SolR #####
RUN yum -y install solr-server hue-search



##### Install MySQL and connector #####
RUN yum -y install mysql mysql-server mysql-connector-java
RUN ln -s /usr/share/java/mysql-connector-java.jar /usr/lib/hive/lib/mysql-connector-java.jar
##### Note: Will use mysql for Hive metastore



##### Format HDFS #####
USER hdfs
RUN hdfs namenode -format
USER root
#####

##### Initialize HDFS Directories #####
RUN bash -c 'for x in `cd /etc/init.d ; ls hadoop-hdfs-*` ; do service $x start ; done' ; \
    bash /usr/lib/hadoop/libexec/init-hdfs.sh \
    oozie-setup sharelib create -fs hdfs://localhost -locallib /usr/lib/oozie/oozie-sharelib-yarn.tar.gz ; \
    bash -c 'for x in `cd /etc/init.d ; ls hadoop-hdfs-*` ; do service $x stop ; done' ;
##### Note: Keep commands on a single line, as we need to init HDFS while services are running

##### Set up Oozie / HUE / Hive ??? #####
RUN oozie-setup db create -run
RUN sed -i 's/secret_key=/secret_key=_S@s+D=h;B,s$C%k#H!dMjPmEsSaJR/g' /etc/hue/conf/hue.ini


##### Make this container SSH friendly #####
RUN yum -y install openssh-server openssh-clients
# Start `sshd` to generate host DSA & RSA keys
RUN service sshd start


# Install Apache Kafka
#RUN cd /opt
#RUN wget http://www.mirrorservice.org/sites/ftp.apache.org/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz
#RUN  tar zxvf kafka_2.11-0.8.2.1.tgz
#RUN  cd kafka_2.11-0.8.2.1

###SPARK 2.2.0

ARG SPARK_ARCHIVE=http://d3kbcqa49mib13.cloudfront.net/spark-2.2.0-bin-hadoop2.7.tgz
RUN curl -s $SPARK_ARCHIVE | tar -xz -C /usr/local/

ENV SPARK_HOME /usr/local/spark-2.2.0-bin-hadoop2.7
ENV PATH $PATH:$SPARK_HOME/bin



ADD files/hive-site.xml /usr/lib/hive/conf/


# Add the Hadoop start-up scripts
ADD scripts/* /opt/

ENV LANG en_US.UTF-8

ADD run.sh /usr/bin/

ADD ojdbc6.jar /opt/

#RUN echo "spark.executor.extraClassPath=/opt/ojdbc6.jar | tee -a /etc/spark/conf/spark-defaults.conf"
#RUN echo "spark.driver.extraClassPath=/opt/ojdbc6.jar | tee -a /etc/spark/conf/spark-defaults.conf"


CMD ["run.sh"]
