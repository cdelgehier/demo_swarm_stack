[TOC]
The demo stack!
=========

Purpose
----------
TODO...

<i class="icon-file"></i> Terminology
--------------------------------------------------------
Docker Daemon
: process which manages the containers for a machine (virtual or physical)

Swarm Join
: Process which ensures the registering of a host to swarm and exposes the host's daemon docker to Swarm, making it available

Swarm Node
: todo

Swarm Manager
: Service used for the container management between Swarm cluster nodes.

Service Discovery Managers (Consul)
:  A service discovery manager, its purpose is to follow the registered services. Swarm Joins and Swarm Managers are connected to it, the Discovery Service Managers take care of everything (eg the addition or deletion of nodes Swarm)

Swarm Cluster Token
: todo

Docker Compose
: This is a tool for defining and running multi-container Docker applications. This tool takes a simple configuration yml file and deploys the containers like a manifest. It is the deployment tool for utilizing the new multi-host networking features that allow the Swarm Nodes to have applications running across distributed hosts. Compose also handles placement strategies for ensuring your containers are distributed evenly (or not) across the Swarm Nodes. This allows for container redundancy at the host level which is good for production resiliency.

Overlay Network
: his is the new native Docker networking type for deploying containers that are linked only with the other containers on the network. This is utilizing VXLAN technology and is really interesting for its ability to separate containers (even on the same node) as well linking containers across multiple Swarm Nodes. The containers deployed with an overlay network can see the other linked containers on the network by using the entries defined in the /etc/hosts file in the container.


Setup
-------
```
[jack@Desktop ~]$ docker --version
Docker version 1.9.1, build a34a1d5
[jack@Desktop ~]$ docker run swarm --version
swarm version 1.0.1 (744e3a3)
[jack@Desktop ~]$ consul --version
Consul v0.6.0
Consul Protocol: 3 (Understands back to: 1)
[jack@Desktop ~]$ docker-compose --version
docker-compose version 1.5.2, build 7240ff3
```


___
Links
------
https://docs.docker.com/swarm/
https://docs.docker.com/swarm/discovery/
http://technologyconversations.com/2015/09/08/service-discovery-zookeeper-vs-etcd-vs-consul/
https://docs.docker.com/engine/userguide/networking/dockernetworks/#an-overlay-network
https://github.com/docker/compose/releases



