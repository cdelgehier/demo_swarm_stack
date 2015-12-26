Purpose
----------
Create a reproducible architecture of application containers using Docker Engine, Docker Compose, Docker Machine and Docker Swarm.
Easy steps to create and deploy your applications locally and push to any cloud provider using the same toolset.

----------
Terminology
---------------
>**Docker Daemon**: process which manages the containers for a machine (virtual or physical)

>**Swarm Join**: Process which ensures the registering of a host and exposes the host's daemon docker to Swarm cluster, making it available

>**Swarm Node**: Each Swarm node will run a Swarm node agent. The agent registers the referenced Docker daemon, monitors it, and updates the discovery backend with the node’s status

>**Swarm Manager**: the swarm manager is responsible for the entire cluster and manages the resources of multiple Docker hosts at scale. If the swarm manager dies, you must create a new one and deal with an interruption of service. A primary manager is the main point of contact with the Docker Swarm cluster. You can also create and talk to replica instances that will act as backups. Requests issued on a replica are automatically proxied to the primary manager. If the primary manager fails, a replica takes away the lead. In this way, you always keep a point of contact with the cluster.

>**Service Discovery Manager**:  A service discovery manager, its purpose is to follow the registered services. Swarm Joins and Swarm Managers are connected to it, the Discovery Service Managers take care of everything (eg the addition or deletion of nodes in Swarm). 

>**Consul**: Consul have Key/value store and Swarm can register node on it. Consul is a distributed, higly available system and provides : Service discovery and failure detection. It works as zookeeper, doozerd or etcd, but adds a DNS interface for service discovery, useful for existing or old services, and seems as a solution with a better scalability using a gossip protocol. Consul provides monitoring services in a fully distributed architecture unlike other tools. Nagios uses central servers to query services and Sensu have a central broker to manage the service notifications, which can cause problems of scale. SkyDNS also provides an interface to perform DNS queries, but monitoring controls use a system heartbeat and TTLs, while Consul can define custom checks.

>**Swarm Cluster Token**: A Docker Swarm can be deployed without running your own Service Discovery Managers; however, this means the token will be shared over an encrypted connection with Docker Hub. **It is not recommended for production use**.If you want to run a distributed Swarm using a token, then all Swarm Joins and Swarm Managers need to specify the token://<your token string>. 

>**Docker Compose**: This is a tool for defining and running multi-container Docker applications. This tool takes a simple configuration yml file and deploys the containers like a manifest. It is the deployment tool for utilizing the new multi-host networking features that allow the Swarm Nodes to have applications running across distributed hosts. Compose also handles placement strategies for ensuring your containers are distributed evenly (or not) across the Swarm Nodes. This allows for container redundancy at the host level which is good for production resiliency.

My setup
-----------
```
$ docker --version
Docker version 1.9.1, build a34a1d5

$ docker run swarm --version
swarm version 1.0.1 (744e3a3)

$ docker-machine --version
docker-machine version 0.5.3, build 4d39a66

$ docker run gliderlabs/consul --version
Consul v0.5.2
Consul Protocol: 2 (Understands back to: 1)

$ docker-compose --version
docker-compose version 1.5.2, build 7240ff3
```

Cluster for dev (dependent on a third party)
-------------
### The token
The create command returns a unique cluster ID (cluster_id). You’ll need this ID when starting the Docker Swarm agent on a node.
```
$ docker run --rm -e HTTP_PROXY="http://proxy:3128" swarm create
<TOKEN>
```
A Docker Swarm can be deployed without a running Service Discovery Managers; however, this means the token will be shared over an encrypted connection with Docker Hub. This is a useful way for getting started with a single host environment.

###Create Swarm nodes
This example uses the Docker Hub based token discovery service. Log into each node and do the following
```
$ docker-machine create -d virtualbox --swarm --swarm-discovery token://<TOKEN-FROM-ABOVE> --swarm-master swarm-master
$ docker-machine create -d virtualbox --swarm --swarm-discovery token://<TOKEN-FROM-ABOVE> swarm-agent-00
$ docker-machine create -d virtualbox --swarm --swarm-discovery token://<TOKEN-FROM-ABOVE> swarm-agent-01

$ docker-machine ls
NAME             ACTIVE   DRIVER       STATE     URL                         SWARM                   DOCKER   ERRORS
swarm-agent-00   -        virtualbox   Running   tcp://192.168.99.103:2376   swarm-master            v1.9.1   
swarm-agent-01   -        virtualbox   Running   tcp://192.168.99.104:2376   swarm-master            v1.9.1   
swarm-master     -        virtualbox   Running   tcp://192.168.99.102:2376   swarm-master (master)   v1.9.1   
```
We connect the master to have some information about the cluster
```
$ eval $(docker-machine env --swarm swarm-master)
$ docker info
Containers: 5
Images: 4
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 3
 swarm-agent-00: 192.168.99.103:2376
  └ Status: Healthy
  └ Containers: 2
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 1.021 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.13-boot2docker, operatingsystem=Boot2Docker 1.9.1 (TCL 6.4.1); master : cef800b - Fri Nov 20 19:33:59 UTC 2015, provider=virtualbox, storagedriver=aufs
 swarm-agent-01: 192.168.99.104:2376
  └ Status: Healthy
  └ Containers: 1
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 1.021 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.13-boot2docker, operatingsystem=Boot2Docker 1.9.1 (TCL 6.4.1); master : cef800b - Fri Nov 20 19:33:59 UTC 2015, provider=virtualbox, storagedriver=aufs
 swarm-master: 192.168.99.102:2376
  └ Status: Healthy
  └ Containers: 2
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 1.021 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.13-boot2docker, operatingsystem=Boot2Docker 1.9.1 (TCL 6.4.1); master : cef800b - Fri Nov 20 19:33:59 UTC 2015, provider=virtualbox, storagedriver=aufs
CPUs: 3
Total Memory: 3.064 GiB
Name: swarm-master
```
How was launched docker daemon on the nodes?
```
$ docker-machine ssh swarm-agent-01 ps aux | grep /usr/local/bin/doc\\ker
 2369 root     /usr/local/bin/docker daemon -D -g /var/lib/docker -H unix:// -H tcp://0.0.0.0:2376 --label provider=virtualbox --tlsverify --tlscacert=/var/lib/boot2docker/ca.pem --tlscert=/var/lib/boot2docker/server.pem --tlskey=/var/lib/boot2docker/server-key.pem -s aufs
```
How are the containers distributed between nodes? (TODO) https://docs.docker.com/swarm/scheduler/strategy/
```
$ docker run -itd --name=singletest busybox
0f61c34dc4fd8209ba952a9301bfe84955cbe3bb6d602b57ed47520471a4fa20

$ docker ps
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
 0f61c34dc4fd        busybox             "sh"                8 seconds ago       Up 6 seconds                            swarm-agent-01/singletest

```
The master is running both the swarm manager and a swarm agent container. This isn’t recommended in a production environment because it can cause problems with agent failover. However, it is perfectly fine to do this in a learning environment like this one.
```
$ docker ps  -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
a035665f5c5f        swarm:latest        "/swarm join --advert"   3 minutes ago       Up 3 minutes                            swarm-agent-01/swarm-agent
3ca7794c1277        swarm:latest        "/swarm join --advert"   3 minutes ago       Up 3 minutes                            swarm-agent-00/swarm-agent
759df927c0db        swarm:latest        "/swarm join --advert"   18 minutes ago      Up 18 minutes                           swarm-master/swarm-agent
69b91e8f87cd        swarm:latest        "/swarm manage --tlsv"   18 minutes ago      Up 18 minutes                           swarm-master/swarm-agent-master
```

Cluster with Consul
-----------------
###Cleanup
```
$ docker-machine rm swarm-master
Do you really want to remove "swarm-master"? (y/n): y

$ docker-machine rm swarm-agent-00
Do you really want to remove "swarm-agent-00"? (y/n): y

$ docker-machine rm swarm-agent-01
Do you really want to remove "swarm-agent-01"? (y/n): y
```

###Consul
If you want to start a Consul cluster on a single host to experiment with clustering dynamics (replication, leader election), here is the recommended way to start a 3 node cluster.

Here we start the first node not with `-bootstrap`, but with `-bootstrap-expect 3`, which will wait until there are 3 peers connected before self-bootstrapping and becoming a working cluster.
```
$ docker run -d --name consul-server1 -h consul-server1 progrium/consul -server -bootstrap-expect 3
```
We can get the container's internal IP by inspecting the container. We'll put it in the env var `JOIN_IP`.
```
$ JOIN_IP="$(docker inspect -f '{{.NetworkSettings.IPAddress}}' consul-server1)"
```
Then we'll start `consul-server2` and tell it to join `consul-server1` using `$JOIN_IP`:
```
$ docker run -d --name consul-server2 -h consul-server2 progrium/consul -server -join $JOIN_IP
```
Now we can start `consul-server3` the same way:
```
$ docker run -d --name consul-server3 -h consul-server3 progrium/consul -server -join $JOIN_IP
```
We now have a real three node cluster running on a single host. Notice we've also named the containers after their internal hostnames / node names.

We haven't published any ports to access the cluster, but we can use that as an excuse to run a fourth agent node in "client" mode (dropping the `-server`). This means it doesn't participate in the consensus quorum, but can still be used to interact with the cluster. It also means it doesn't need disk persistence.

```
$ docker run -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp --name consul-client1 -h consul-client1 progrium/consul -join $JOIN_IP

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                        NAMES
e68fbb3c44eb        progrium/consul     "/bin/start -join 172"   8 seconds ago       Up 6 seconds        53/tcp, 0.0.0.0:8400->8400/tcp, 8300-8302/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp, 0.0.0.0:8600->53/udp   consul-client1
8537644d1818        progrium/consul     "/bin/start -server -"   21 seconds ago      Up 19 seconds       53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 8500/tcp                                             consul-server3
97c7baddd63f        progrium/consul     "/bin/start -server -"   28 seconds ago      Up 27 seconds       53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 8500/tcp                                             consul-server2
303caa868cb6        progrium/consul     "/bin/start -server -"   40 seconds ago      Up 39 seconds       53/tcp, 53/ud

```
Now we can interact with the cluster on those published ports and, if you want, play with killing, adding, and restarting nodes to see how the cluster handles it

We publish 8400 (RPC), 8500 (HTTP), and 8600 (DNS) so you can try all three interfaces. We also give it a hostname of consul-client1. Setting the container hostname is the intended way to name the Consul Agent node.

Our recommended interface is HTTP using curl:

```
$ curl localhost:8500/v1/catalog/nodes | python -m json.tool
[
    {
        "Address": "172.17.0.5",
        "Node": "consul-client1"
    },
    {
        "Address": "172.17.0.2",
        "Node": "consul-server1"
    },
    {
        "Address": "172.17.0.3",
        "Node": "consul-server2"
    },
    {
        "Address": "172.17.0.4",
        "Node": "consul-server3"
    }
]

$ dig +short @0.0.0.0 -p 8600 consul-server1.node.consul
172.17.0.2
```
However, if you install Consul on your host, you can use the CLI to interact with the containerized Consul Agent:

```
$ consul members
Node            Address          Status  Type    Build  Protocol  DC
consul-client1  172.17.0.5:8301  alive   client  0.5.2  2         dc1
consul-server1  172.17.0.2:8301  alive   server  0.5.2  2         dc1
consul-server2  172.17.0.3:8301  alive   server  0.5.2  2         dc1
consul-server3  172.17.0.4:8301  alive   server  0.5.2  2         dc1
```
cluster
```
$ VBOX_IP_CONSUL=$(ip addr show vboxnet1 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1)
$ docker-machine create -d virtualbox --swarm --swarm-discovery consul://$VBOX_IP_CONSUL:8500/SWARM --swarm-master swarm-master

$ docker-machine create -d virtualbox --swarm --swarm-discovery consul://$VBOX_IP_CONSUL:8500/SWARM  swarm-agent-00

$ docker-machine create -d virtualbox --swarm --swarm-discovery consul://$VBOX_IP_CONSUL:8500/SWARM  swarm-agent-01
```
i
```
$ eval $(docker-machine env --swarm swarm-master)

$ docker pull training/webapp
Using default tag: latest
swarm-agent-00: Pulling training/webapp:latest... : downloaded 
swarm-agent-01: Pulling training/webapp:latest... : downloaded 
swarm-master: Pulling training/webapp:latest... : downloaded 

$ docker run -d -p 5000:5000 training/webapp
$ docker run -d -p 5000:5000 training/webapp

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                           NAMES
50696d6aa471        training/webapp     "python app.py"     20 seconds ago      Up 18 seconds       192.168.99.102:5000->5000/tcp   swarm-agent-01/hopeful_hypatia
d99d0b05b9ca        training/webapp     "python app.py"     24 seconds ago      Up 23 seconds       192.168.99.101:5000->5000/tcp   swarm-agent-00/silly_fermi

$ curl http://$(docker-machine ip swarm-agent-00):5000
Hello world!
$ curl http://$(docker-machine ip swarm-agent-01):5000
Hello world!


$ curl -XPUT 127.0.0.1:8500/v1/agent/service/register -d '{
 "id":"webapp_training_server1",
 "name":"webapp",
 "address":"192.168.99.101",
 "port": 5000,
 "tags": ["api"] }'

$ curl -XPUT 127.0.0.1:8500/v1/agent/service/register -d '{
 "id":"webapp_training_server2",
 "name":"webapp",
 "address":"192.168.99.102",
 "port": 5000,
 "tags": ["api"] }'

$ dig +short @127.0.0.1 -p 8600 webapp.service.consul
192.168.99.101
192.168.99.102

```

___
Links
------
https://docs.docker.com/swarm/install-manual/
https://docs.docker.com/swarm/discovery/
http://technologyconversations.com/2015/09/08/service-discovery-zookeeper-vs-etcd-vs-consul/
https://github.com/docker/compose/releases
https://blog.docker.com/2015/01/dockercon-eu-introducing-docker-swarm/
https://hub.docker.com/r/progrium/consul/

