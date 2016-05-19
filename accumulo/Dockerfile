# for building a stand alone accumulo container
#
FROM ubuntu:14.04
MAINTAINER David Holiday <dholiday@moesol.com>


# provision machine 
#
RUN \
  sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list && \
  apt-get update && \
  apt-get -y upgrade && \
  apt-get install -y build-essential && \
  apt-get install -y software-properties-common && \
  apt-get install -y htop unzip vim wget ssh && \
  rm -rf /var/lib/apt/lists/*

RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer


# ensure environment variables are set 
# *NOTE* this works only when this container is linked to one containing hadoop and one containing zookeeper!
#
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV HADOOP_HOME /opt/hadoop-2.6.3
ENV HADOOP_PREFIX /opt/hadoop-2.6.3
ENV ZOOKEEPER_HOME /opt/zookeeper


# install accumulo
#
WORKDIR /opt
RUN sudo wget http://mirror.metrocast.net/apache/accumulo/1.7.1/accumulo-1.7.1-bin.tar.gz
RUN sudo wget https://github.com/jwilder/dockerize/releases/download/v0.2.0/dockerize-linux-amd64-v0.2.0.tar.gz
RUN sudo tar -C /usr/local/bin -xzvf dockerize-linux-amd64-v0.2.0.tar.gz
RUN sudo tar -xvf accumulo-1.7.1-bin.tar.gz

WORKDIR /opt/accumulo-1.7.1/bin
RUN sudo ./bootstrap_config.sh -s 2GB -j -v 2


# update accumulo config and shell scripts
#
RUN sudo sed -i 's|localhost:2181|zookeeper:2181|' /opt/accumulo-1.7.1/conf/accumulo-site.xml
RUN sudo sed -i 's|<value></value>|<value>hdfs://hadoop:9000/accumulo</value>|' /opt/accumulo-1.7.1/conf/accumulo-site.xml

RUN sudo sed -i 's|HADOOP_PREFIX=/path/to/hadoop|HADOOP_PREFIX=/opt/hadoop-2.6.3|' /opt/accumulo-1.7.1/conf/accumulo-env.sh
RUN sudo sed -i 's|JAVA_HOME=/path/to/java|JAVA_HOME=/usr/lib/jvm/java-8-oracle|' /opt/accumulo-1.7.1/conf/accumulo-env.sh
RUN sudo sed -i 's|ZOOKEEPER_HOME=/path/to/zookeeper|ZOOKEEPER_HOME=/opt/zookeeper|' /opt/accumulo-1.7.1/conf/accumulo-env.sh
RUN sudo sed -i 's|# export ACCUMULO_MONITOR_BIND_ALL=|export ACCUMULO_MONITOR_BIND_ALL=|' /opt/accumulo-1.7.1/conf/accumulo-env.sh

RUN sudo sed -i 's|localhost|0.0.0.0|' /opt/accumulo-1.7.1/conf/masters
RUN sudo sed -i 's|localhost|0.0.0.0|' /opt/accumulo-1.7.1/conf/slaves
RUN sudo sed -i 's|localhost|0.0.0.0|' /opt/accumulo-1.7.1/conf/monitor

RUN sudo sed -i 's|$MONITOR monitor|$HOSTNAME monitor|' /opt/accumulo-1.7.1/bin/start-all.sh


# expose webservice port
#
EXPOSE 9995
EXPOSE 50095
EXPOSE 9997


# swap these lines only after you've initialized accumulo manually! see README.md for deets
#
ENTRYPOINT while true; do sleep 1000; done

#ENTRYPOINT  dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 sudo service ssh start && \
#            dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 sudo /opt/accumulo-1.7.0/bin/start-all.sh && \
#            dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 while true; do sleep 1000; done 
