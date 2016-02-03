# for standing up hadoop + hdfs
#
FROM ubuntu:14.04
MAINTAINER David Holiday <dholiday@moesol.com>
ENV REFRESHED AT 2016-01-14


# grab needed libraries
#
RUN apt-get -yqq update
RUN apt-get -yqq install wget vim ssh rsync default-jre default-jdk


# setup global environment variables for java + hadoop
#
ENV JAVA_HOME=/usr/bin/java
ENV HADOOP_HOME=/opt/hadoop-2.6.3
ENV HADOOP_INSTALL=$HADOOP_HOME
ENV HADOOP_MAPRED_HOME=$HADOOP_HOME
ENV HADOOP_COMMON_HOME=$HADOOP_HOME
ENV HADOOP_HDFS_HOME=$HADOOP_HOME
ENV YARN_HOME=$HADOOP_HOME
ENV HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
ENV PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin


# ensure ssh is fired up if it isn't already
#
RUN sudo service ssh start


# install / setup hadoop 
#
WORKDIR /opt

RUN sudo wget http://supergsego.com/apache/hadoop/common/hadoop-2.6.3/hadoop-2.6.3.tar.gz
RUN sudo tar xvfz hadoop-2.6.3.tar.gz

RUN sudo cat /dev/zero | ssh-keygen -q -N "" &&\
        sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys &&\
        sudo touch ~/.ssh/config &&\
        echo "Host localhost\n\tStrictHostKeyChecking no\n\tUserKnownHostsFile=/dev/null\n\nHost 0.0.0.0\n\tStrictHostKeyChecking no\n\tUserKnownHostsFile=/dev/null" > ~/.ssh/config &&\
        sudo chmod 0600 ~/.ssh/authorized_keys


RUN sudo sed -i '/<configuration>/a \ <property> \n \
    <name>fs.default.name</name> \n \ 
    <value>hdfs://0.0.0.0:9000</value> \n \
    </property>' /opt/hadoop-2.6.3/etc/hadoop/core-site.xml


RUN sudo sed -i '/<configuration>/a \
    <property> \n \
    <name>dfs.replication</name> \n \
    <value>1</value> \n \
    </property>\n\n \
    <property> \n \
    <name>dfs.name.dir</name>\n \
    <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>\n \
    </property> \n\n \ 
    <property> \n \
    <name>dfs.data.dir</name>\n \
    <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value> \n \
    </property>' /opt/hadoop-2.6.3/etc/hadoop/hdfs-site.xml


RUN sed -i 's|JAVA=$JAVA_HOME/bin/java|JAVA=${JAVA_HOME}|' /opt/hadoop-2.6.3/libexec/hadoop-config.sh


RUN JAVA_HOME_OLD='export JAVA_HOME=${JAVA_HOME}' &&\
    JAVA_HOME_NEW="export JAVA_HOME=\'/usr/bin/java'" &&\
    sed -i "s|$JAVA_HOME_OLD|$JAVA_HOME_NEW|" /opt/hadoop-2.6.3/etc/hadoop/hadoop-env.sh


# get the namenode ready
#
WORKDIR /opt/hadoop-2.6.3
RUN ./bin/hdfs namenode -format


# expose all hadoop ports to other containers
#
EXPOSE 9000
EXPOSE 50070
EXPOSE 8088
EXPOSE 50090
EXPOSE 50075


# expose hadoop home
#
VOLUME /opt/hadoop-2.6.3


# kinda jenky to have to do it this way but it works
# ensure ssh is running, fire up dfs, then create endless loop to ensure container 
# stays alive
#
ENTRYPOINT sudo service ssh start && ./sbin/start-dfs.sh && while true; do sleep 1000; done





