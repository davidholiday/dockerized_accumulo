# dockerized accumulo in three acts
---

## what is? 

[Docker](http://www.docker.com/) is a technology that allows you to package an application with all of its dependencies into a standardized unit for software development. [Accumulo](http://accumulo.apache.org/) is a a scalable, secure noSQL database. What's here are the files needed to provision an accumulo stack consisting of three docker containers: one for hadoop (used as the file system back-ending accumulo), [zookeeper](http://zookeeper.apache.org/) (used by accumulo to coordinate the accumulo cluster), and accumulo itself. All three are provisioned to behave as single-node clusters, but, can be configured to include additional containerized nodes.

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

    #ENTRYPOINT sudo /opt/accumulo-1.7.0/bin/start-all.sh && while true; do sleep 1000; done
    ENTRYPOINT while true; do sleep 1000; done


to this:


    ENTRYPOINT sudo /opt/accumulo-1.7.0/bin/start-all.sh && while true; do sleep 1000; done
    #ENTRYPOINT while true; do sleep 1000; done

save and close the file. 




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

