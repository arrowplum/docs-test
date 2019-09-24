---
title: Orchestrating Aerospike Clusters with Docker Compose
description: |
  Here are the best practices for Orchestrating Aerospike clusters with Docker Compose
scripts:
  - /assets/scripts/utils/target_blank.js
---


{{#note}}
This guide is accurate with the Docker 1.13 release. As Docker changes its APIs and recommendations this guide will evolve. Every attempt will be made to keep it current. 
{{/note}}

## Background
[Docker Compose](https://docs.docker.com/compose/) is part of the Docker tool chain that enable multi-container applications to be orchestrated. This is an obvious mechanism to orchestrate the deployment and scaling of an Aerospike cluster.

#### Define the Aerospike Service

```bash
...
services:
    aerospikedb:
        image: aerospike/aerospike-server:3.14.1.1
        networks:
        - aerospikenetwork
        deploy:
            replicas: 2
            endpoint_mode: dnsrr
        labels:
            com.aerospike.cluster: "myproject"
        command: [ "--config-file","/run/secrets/aerospike.conf"]
        secrets:
        - source: conffile
          target: aerospike.conf
          mode: 0440
...
```
Key points:
- Aerospike server version is defined (e.g. 3.14.1.1)
- A declaration to use the network `aerospikenetwork`. This ensures that the discovery process can connect to the aerospike nodes to perform clustering.
- A deploy parameter that specifies initial cluster size 
- A deploy parameter that specifies the discovery endpoint as `dnsrr` which is DNS round-robin. This is a Compose 3.2 feature
- A label defines the cluster name (e.g. "com.aerospike.cluster=myproject")
- Append a command to use a custom conf file
- A secret is mounted that contains our aerospike.conf configuration file, as read-only

#### Define the Meshworker Discovery Service

```bash
...
meshworker:
        image: aerospike/aerospike-tools:latest
        networks:
        - aerospikenetwork
        depends_on:
        - aerospike
        entrypoint:
        - /run/secrets/discovery
        - "--servicename"
        - aerospikedb
        - "-i"
        - "5"
        - "-v"
        secrets:
        - source: discoveryfile
          target: discovery
          mode: 0750
...
```
Key points:
- Uses the aerospike/aerospike-tools image to trigger cluster reconfigurations
- A declaration to use the network `aerospikenetwork`. This ensures that the discovery process can connect to the aerospike nodes to perform clustering.
- Depends_on: this is ignored in swarm mode, but obeyed with single node with compose. Ensures this service starts after the `aerospikedb` service.
- Overwrite the entrypoint for `aerospike/aerospike-tools` container to run our discovery script
  - Passes in various parameters like: the service name to resolve
  - The interval to poll the service name
  - And a verbose flag for debugging/logging
- Mounts the discoveryfile secret as an executable script

#### Define an optional AMC Service

```bash
...
amc:

        image: aerospike/amc:4.0.25
        networks:
        - aerospikenetwork
        deploy:
            mode: global
            endpoint_mode: dnsrr
        ports:

        - "8081:8081"
        deploy:
           placement:
             constraints: [node.role == manager]
...
```

Key points:
- Uses the aerospike/amc Community Edition image.
- A deploy mode of global to limit number of instances running AMC.
- A declaration to use the overlay network `aerospikenetwork` which is shared with Aerospike server containers.


#### Network definition

Starting with Compose v2, overlay networks can be defined as well.

```bash
...
networks:
    aerospikenetwork:
        driver: overlay
        attachable: true
...
```
Key points:
- Uses the overlay driver for this network
- Turns on the attachable flag for easier debugging. Allows non-compose managed containers to also utilize this network.

#### Secrets definition

Secrets are used instead of volume mounts for two reasons:

1. Secrets are propagated internally by docker, whereas volume mounts need to exist on the docker host (or swarm node) in order to be utilized.
2. Long format declaration allows renaming and setting permissions/

```bash
...
secrets:
    conffile:
        file: ./aerospike.conf
    discoveryfile:
        file: ./discovery.py
```

#### Use Stack/Compose to deploy the Aerospike Containers
`docker stack` can then be used in conjunction with your compose file and optionally your Docker Swarm cluster.
```bash
$ docker stack deploy -c aerospike.yml aerospike
```

You can see the Docker container running Aerospike on the Docker Swarm cluster using the `docker ps` command.
```bash
$ docker ps
docker ps
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS               NAMES
360e1da2d43f        aerospike/aerospike-tools:latest    "/run/secrets/disc..."   3 hours ago         Up 3 hours                              aerospike_meshworker.1.zvs6cfhz0z8cebej5tdg4qdou
4d8fc0bd20f8        aerospike/aerospike-server:latest   "/entrypoint.sh --..."   3 hours ago         Up 3 hours          3000-3003/tcp       aerospike_aerospikedb.3.v7edr5cvnnxvtep81beam9lqh
...
```

You can see your service status using the `docker service ls command`
```bash
docker service ls
ID                  NAME                    MODE                REPLICAS            IMAGE                               PORTS
rfee38b7u9mt        aerospike_meshworker    replicated          1/1                 aerospike/aerospike-tools:latest    
ws864n7ivhy2        aerospike_aerospikedb   replicated          2/2                 aerospike/aerospike-server:latest  
```

{{#note}}
The name of the container is made up of
<ul>
  <li>Stack Name (by the `docker stack deploy` command) e.g. "aerospike"</li>
  <li>Service Name from the yaml file e.g. "aerospike"</li>
  <li>Numeric Id (indicates how many of the service has been started) e.g. "1"</li>
  <li>Task Id (internal id for every docker action) </li>
</ul>
{{/note}}

#### Use Service to scale the Aerospike Cluster
Docker services can be scaled dynamically using `docker service scale`.

Ensure two Aerospike services (Containers) are running
```bash
$ docker service scale aerospike_aerospikedb=2
```


{{#note}}
At this point, the discovery container would see the updated hostnames and automatically cluster the containers
{{/note}}

#### View the cluster config
View the Aerospike cluster topology using `asadm`.
```bash
$ docker run --net aerospike_aerospikenetwork aerospike/aerospike-tools asadm -h aerospikedb -e info
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                                                             Node               Node              Ip      Build   Cluster        Cluster     Cluster         Principal   Client     Uptime   
                                                                                .                 Id               .          .      Size            Key   Integrity                 .    Conns          .   
aerospike_aerospikedb.1.o0eujlsuyc8bylgzxt3svr61b.aerospike_aerospikenetwork:3000   *BB90301000A4202   10.0.1.3:3000   C-3.14.0         2   41339CCF5C67   True        BB90301000A4202        5   02:50:39   
aerospike_aerospikedb.3.v7edr5cvnnxvtep81beam9lqh.aerospike_aerospikenetwork:3000   BB90201000A4202    10.0.1.2:3000   C-3.14.0         2   41339CCF5C67   True        BB90301000A4202        5   02:50:39   
Number of rows: 2

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Namespace Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Namespace                                                                                Node   Avail%   Evictions                 Master                Replica     Repl     Stop     Pending       Disk    Disk     HWM        Mem     Mem    HWM      Stop   
        .                                                                                   .        .           .   (Objects,Tombstones)   (Objects,Tombstones)   Factor   Writes    Migrates       Used   Used%   Disk%       Used   Used%   Mem%   Writes%   
        .                                                                                   .        .           .                      .                      .        .        .   (tx%,rx%)          .       .       .          .       .      .         .   
test        aerospike_aerospikedb.1.o0eujlsuyc8bylgzxt3svr61b.aerospike_aerospikenetwork:3000   99         0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)       0.000 B    0       50      0.000 B    0       60     90        
test        aerospike_aerospikedb.3.v7edr5cvnnxvtep81beam9lqh.aerospike_aerospikenetwork:3000   99         0.000     (0.000  ,0.000  )      (0.000  ,0.000  )      2        false    (0,0)       0.000 B    0       50      0.000 B    0       60     90        
test                                                                                                       0.000     (0.000  ,0.000  )      (0.000  ,0.000  )                        (0,0)       0.000 B                    0.000 B                             
Number of rows: 3

```

#### Establishing an Aerospike Mesh Cluster dynamically
If the discovery container was not used, you would need to manually mesh the aerospike containers.

While both Containers are now on the same network, the Docker Overlay network [does not support multi-cast](https://github.com/docker/libnetwork/issues/552). Therefore, each node has to be added into the Aerospike Cluster with the `tip` command.

<pre>
<code>$ docker run --net aerospike_aerospikenetwork aerospike/aerospike-tools asinfo -v "tip:host=$(docker inspect -f '&#123;&#123;.NetworkSettings.Networks.prod.IPAddress }}' aerospike.2.nl30psiorabhcu2rj1kpghq6c );port=3002" -h aerospike.1.i6httgeicb9ix0vdevw8ye7xt </code>
</pre>

## Additional Information
- See this template on our [Github](https://github.com/aerospike/aerospike-docker-swarm)
