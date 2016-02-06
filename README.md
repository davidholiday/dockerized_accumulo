# dockerized accumulo in three containers


## what is? 

[Docker](http://www.docker.com/) is a technology that allows you to package an application with all of its dependencies into a standardized unit for software development. [Accumulo](http://accumulo.apache.org/) is a a scalable, secure NoSQL database. What's here are the files needed to provision an accumulo stack consisting of three docker containers: one for hadoop (used as the file system back-ending accumulo), [zookeeper](http://zookeeper.apache.org/) (used by accumulo to coordinate the accumulo cluster), and accumulo itself. All three are provisioned to behave as single-node clusters, but, can be configured to include additional containerized nodes.

## how to set up?

For this to work you'll need docker and [docker-compose](https://docs.docker.com/compose/install/) installed. If you don't have both installed, go take care of that before proceeding. 

ready? w00t. Go to the ../dockerized_accumulo/compose directory and type

    docker-compose up -d

This will provision and bring up all three containers. We're almost there! We just need to:

 1. manually initialize accumulo
 2. make one small change to the accumulo docker file
 3. create an updated accumulo container

### manually initialize accumulo

type:

    docker exec -t -i accumulo /bin/bash

This will spin up a bash shell instance in the accumulo container and log you in as root. Now, from inside the accumulo container, type:

    ./accumulo init

You'll be asked for an instance name and a password. The name you pick doesn't matter. The password you might want to remember in case you ever need to log into this container and get into the accumulo shell. Once you've done that, type 

    exit
   Which will kick you back into your host's terminal session.

### make one small change to the accumulo docker file 

type:

    docker stop accumulo

 then:
 

    docker rm accumulo
    
which will stop and remove the accumulo container but not the underlying image. Now edit the accumulo docker file at ../dockerized_accumulo/accumulo/Dockerfile. Change the last two lines from this:

    ENTRYPOINT while true; do sleep 1000; done

    #ENTRYPOINT  dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 sudo service ssh start && \
    #                dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 sudo /opt/accumulo-1.7.0/bin/start-all.sh && \
    #                dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 while true; do sleep 1000; done



to this:


    #ENTRYPOINT while true; do sleep 1000; done

    ENTRYPOINT  dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 sudo service ssh start && \
                    dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 sudo /opt/accumulo-1.7.0/bin/start-all.sh && \
                    dockerize -wait tcp://zookeeper:2181 -wait tcp://hadoop:9000 while true; do sleep 1000; done


save and close the file. From here you need to incorporate the changes you made into the image:

    docker build -t compose_accumulo [path to accumulo Dockerfile]




### create an updated accumulo container

From the ../dockerized_accumulo/compose folder, type:

    docker-compose up -d

and you're done! 


## how to make it stop
From the ../dockerized_accumulo/compose folder, type:

    docker-compose stop


## how to make it go again
From the ../dockerized_accumulo/compose folder, type:

    docker-compose up -d


## troubleshooting
If you encounter an error stating something like *Failed to fetch http://archive.ubuntu.com/ubuntu/dists/trusty/Release.gpg  Could not resolve 'archive.ubuntu.com'*, follow the instructions [here](https://docs.docker.com/engine/installation/linux/ubuntulinux/#configure-a-dns-server-for-use-by-docker:7064cc4a474d59e7463e9c65d7d35de5) to configure docker to point to google's DNS server. 



