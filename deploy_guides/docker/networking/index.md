---
title: Using Multi-Host and Overlay networking with Docker
description: |
  Here are recommendations for ensuring a successful deployment of Aerospike with Multi-Host and Overlay Docker networking.
scripts:
  - /assets/scripts/utils/target_blank.js
---

{{#note}}
Networking is an ever changing beast in Docker. This guide is accurate with the Docker 1.12 release. As Docker changes its APIs and recommendations this guide will evolve. Every attempt will be made to keep it current.
{{/note}}

### Background
In the 1.9 Release of Docker, [Multi-Host and Overlay networks](https://docs.docker.com/engine/userguide/networking/get-started-overlay/) became a GA feature. This enables private networks to be established across hosts that Containers are allowed to join. For an Aerospike cluster this allows
  - A network to be established for intra cluster communications
  - A network to be established for client to cluster communications
  - Concrete host names which can be used to configure and administrate the cluster

Since each Container has its own IP address, this allows for each Aerospike node to use the same Ports for client, heartbeat and other IP traffic regardless of how many identical Containers are running on the same host. This avoids having to map ports differently for each Aerospike Node because of NAT translation etc.

## Pre-requisites
You will need to establish a [Swarm Cluster](https://docs.docker.com/engine/swarm/) to create a cluster of Docker hosts. Here is a sample Swarm cluster setup.

{{#note}}
`docker-machine` can create the Docker Daemon on various Virtualization and Cloud platforms. The example below uses `virtualbox`, so substitute for the provider you want to use.
{{/note}}

#### Create the Docker host and run it in swarm manager mode

Create the new Docker Daemon
```bash
$ docker-machine create \
  -d virtualbox \
  swarm-manager
```

192.168.99.100 happens to be the IP address of the shared network that our new Docker host is connected to.

```bash
$ docker-machine ssh swarm-manager
docker@swarm-manager:~$ docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (xgotp0n6dgx6faqojdol6krbr) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-0s6qnlo4o8npdavu9pa4e653kzp3jvb0mny0b5ckvlimaopd76-ef2bb3u0dolkqswhdi961h08b \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

#### Create two Docker Daemons as workers

```bash
$ docker-machine create \
  -d virtualbox \
  swarm-0
$ docker-machine create \
  -d virtualbox \
  swarm-1
```

Add the workers into the swarm by copy and pasting in the output from the manager above:

```bash
docker@swarm-0:~$ docker swarm join \
	--token SWMTKN-1-0s6qnlo4o8npdavu9pa4e653kzp3jvb0mny0b5ckvlimaopd76-ef2bb3u0dolkqswhdi961h08b \
	192.168.99.100:2377
---
docker@swarm-1:~$ docker swarm join \
    --token SWMTKN-1-0s6qnlo4o8npdavu9pa4e653kzp3jvb0mny0b5ckvlimaopd76-ef2bb3u0dolkqswhdi961h08b \
	192.168.99.100:2377
```

### Setup Docker environment for the Swarm
To ensure we are communicating with the Swarm cluster, send all docker commands to the `swarm-manager` node.
We can do this by setting our docker executable to communicate with the Docker Daemon on the Swarm-Manager.

```bash
$ eval $(docker-machine env swarm-manager)
```

Or we can ssh into the `swarm-manager` instance and perform actions on it directly:

```bash
$ docker-machine ssh swarm-manager
```

### Establish a Multi-Host Network
We want to end up with the Aerospike Containers attached to the private `prod` overlay network:

![Overlay Network](/docs/deploy_guides/docker/assets/networking/networking.png)

Create a network `prod` with the `overlay` driver

```bash
$ docker $(docker-machine config swarm-manager) network create --driver overlay prod --attachable
```

### Connecting Aerospike to the "prod" Network
To work the Docker Swarm mode, we create a Service. 
Start with two Aerospike Containers on the `prod` networking using the `--network` parameter. The `--endpoint-mode` parameter is required to be set as `dnsrr`, as the default virtual IP (`vip`) mode would interfere with Aerospike Client's discovery of servers.

```bash
$ docker service create --network prod --replicas 2 --endpoint-mode dnsrr --name aerospike aerospike/aerospike-server
```

Confirm the containers are running
```bash
$ docker ps
CONTAINER ID        IMAGE       		                COMMAND                CREATED             STATUS              PORTS               NAMES
c87b423276ae        aerospike/aerospike-server:latest   "/entrypoint.sh asd"   5 seconds ago       Up 4 seconds        3000-3003/tcp       aerospike.1.i6httgeicb9ix0vdevw8ye7xt
```

We only see one container due to the fact that `docker ps` only queries a single docker host. We can see the other container if we run `docker ps` on swarm-0.

```bash
docker@swarm-0:~$ docker ps
CONTAINER ID        IMAGE                               COMMAND                CREATED             STATUS              PORTS               NAMES
f96a3e6c165d        aerospike/aerospike-server:latest   "/entrypoint.sh asd"   40 minutes ago      Up 40 minutes       3000-3003/tcp       aerospike.2.nl30psiorabhcu2rj1kpghq6c
```

You can log into either of the nodes with AQL using the hostname `aerospike.1.i6httgeicb9ix0vdevw8ye7xt` or `aerospike.2.nl30psiorabhcu2rj1kpghq6c`, or using the service name `aerospike`. The service name becomes a DNS hostname, that we can use for service discovery. Using the service name will result in a connection to a random container running our Aerospike service. It is perfectly fine to use the service name in the connection string of our Aerospike CLI tools.

In this example we will run another Container that has the AQL executable baked into the image.
```bash
$  docker run -it --rm --net prod aerospike/aerospike-tools aql -h aerospike
aql> insert into test.foo (PK, foo) values ('123','my string')
OK, 1 record affected.

aql> select * from test.foo
+-------------+
| foo         |
+-------------+
| "my string" |
+-------------+
1 row in set (0.028 secs)
```

### Establishing an Aerospike Mesh Cluster dynamically
Since Docker [does not support multi-cast networks](https://github.com/docker/libnetwork/issues/552) on Overlay networks, we need to assemble the cluster over the Mesh network via the `tip` command. We need you specify the `--net` parameter to Docker to ensure that we connect to the network that the Aerospike cluster is running on.

<pre>
<code>$ docker run --net prod aerospike/aerospike-tools asinfo -v "tip:host=$(docker $(docker-machine config swarm-0) inspect -f '&#123;&#123;.NetworkSettings.Networks.prod.IPAddress }}' aerospike.2.nl30psiorabhcu2rj1kpghq6c );port=3002" -h aerospike.1.i6httgeicb9ix0vdevw8ye7xt </code>
</pre>

View the cluster config

```bash
$ docker run --net prod aerospike/aerospike-tools asadm -h aerospike -e info

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                           Node               Node              Ip        Build   Cluster        Cluster     Cluster         Principal   Client     Uptime   
                                              .                 Id               .            .      Size            Key   Integrity                 .    Conns          .   
aerospike.1.i6httgeicb9ix0vdevw8ye7xt.prod:3000   BB90500000A4202    10.0.0.5:3000   C-3.14.1.1         2   851A3ED82B1F   True        BB90600000A4202        6   01:06:50   
aerospike.2.nl30psiorabhcu2rj1kpghq6c.prod:3000   *BB90600000A4202   10.0.0.6:3000   C-3.14.1.1         2   851A3ED82B1F   True        BB90600000A4202        1   01:06:35   
Number of rows: 2

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace                                              Node   Avail%   Evictions                 Master                Replica     Repl     Stop     Pending       Disk    Disk     HWM        Mem     Mem    HWM      Stop   
        .                                                 .        .           .   (Objects,Tombstones)   (Objects,Tombstones)   Factor   Writes    Migrates       Used   Used%   Disk%       Used   Used%   Mem%   Writes%   
        .                                                 .        .           .                      .                      .        .        .   (tx%,rx%)          .       .       .          .       .      .         .   
test        aerospike.1.i6httgeicb9ix0vdevw8ye7xt.prod:3000   99         0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)       0.000 B    0       50      0.000 B    0       60     90        
test        aerospike.2.nl30psiorabhcu2rj1kpghq6c.prod:3000   99         0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)       0.000 B    0       50      0.000 B    0       60     90        
test                                                                     0.000     (0.000  ,0.000  )      (0.000  ,0.000  )                        (0,0)       0.000 B                    0.000 B                             
Number of rows: 3

```


### Automatic Mesh Configuration

Aerospike provides a [Swarm template with a discovery script](https://github.com/aerospike/aerospike-docker-swarm) that would automatically cluster Aerospike Containers deployed via Compose or Swarm Mode. Please see our [Orchestration](/docs/deploy_guides/docker/orchestrate) page for more details.

