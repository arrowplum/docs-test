---
title: Deploying Aerospike clusters with Docker
description: |
  Using Docker to deploy and manage your Aeropsike Cluster. 
styles:
  - /assets/styles/ui/steps.css

---

### Why Containers?
Containers are seen as a simple way to

- Encapsulate the dependencies for the process you want to run, e.g. the packages required, the frameworks that need to be present etc. 
- Provides isolation at runtime, enabling containers each with different dependencies, O/S Kernel version etc. to co-exist on the same physical host.

Application architectures are also evolving, especially with the adoption of micro-server like architectures. Your application is no longer one humongous, static binary or set of packages, its starting to look like a series of discrete services that are brought together dynamically at runtime. This is a natural fit for Containers. But how does this fit with services that require persistence or require long running processes?

### Why Aerospike and Containers?
Aerospike it naturally suited to container deployment because
- [Shared nothing](/docs/architecture/data-distribution.html) architecture
- [Automatic hashing of keys](/docs/architecture/distribution.html) across the cluster (using the RIPEMD-160 collision free algorithm) with Smart Partitions&#0153;
- [Automatic healing](/docs/architecture/data-distribution.html#automatic-data-rebalancing) of the cluster as nodes enter and leave the cluster, including automatic re-balancing
- [Automatic discovery](/docs/architecture/clients.html) of the cluster topology with Aerospike Smart Client&#0153; for all the populate languages and frameworks
- [Automatic replication](/docs/architecture/data-distribution.html#how-data-is-replicated-synchronized-locally) of data across nodes with a customizable replication factor

This enables you to
- Scale (up and out) the persistence layer
- Eliminate reconfiguration of the application and database tier as Containers enter and leave the topology
- Utilize Containers on your own dedicated infra-structure or public cloud providers

{{#info}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/info}}

### Getting started
Aerospike provides both [CE images](https://hub.docker.com/r/aerospike/aerospike-server) and [EE images](https://hub.docker.com/r/aerospike/aerospike-server-enterprise) on Docker Hub. 

Users also have the ability to build their own container images from the Dockerfile on Github.([EE repo](https://github.com/aerospike/aerospike-server-enterprise.docker), [CE repo](https://github.com/aerospike/aerospike-server.docker))

Let's first create a Docker Daemon (if you don't have one running already):

```bash
$ docker-machine create -d vmwarefusion docker-daemon
Running pre-create checks...
Creating machine...
Waiting for machine to be running, this may take a few minutes...
Machine is running, waiting for SSH to be available...
Detecting operating system of created instance...
Provisioning created instance...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
To see how to connect Docker to this machine, run: docker-machine env docker-daemon
```

You can now `run` the Aerospike container and check its status with `ps`:

```bash
$ eval $(docker-machine env docker-daemon)
$ docker run -d --name aerospike aerospike/aerospike-server
b33324047ab5cc5d6a1884504d4a3135c167022fdc4e5c357eeff1913fe7aa0d
```

EE edition will require a feature-key to be configured in `aerospike.conf`:

```bash
$ sudo docker run -tid --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 -v <DIRECTORY>:/etc/aerospike/ -e "FEATURE_KEY_FILE=/etc/aerospike/features.conf" aerospike/aerospike-server-enterprise
```

```bash
$ docker ps
CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS              PORTS               NAMES
b33324047ab5        aerospike/aerospike-server   "/entrypoint.sh asd"   14 seconds ago      Up 14 seconds       3000-3003/tcp       aerospike
```

You can now run [AQL](/docs/tools/aql) in another Docker container to insert and query some data into the Aerospike server:

<pre>
<code>$ docker run -it aerospike/aerospike-tools aql -h  $(docker inspect -f '&#123;&#123;.NetworkSettings.IPAddress }}' aerospike)
aql> insert into test.foo (PK, foo) values ('123','my string')
OK, 1 record affected.

aql> select * from test.foo
+-------------+
| foo         |
+-------------+
| "my string" |
+-------------+
1 row in set (0.028 secs)</code>
</pre>

### What's Next?

The following guides details how to deploy, configure and utilize Aerospike from Development through Production.

<ul>
    <li>[Working with Docker](/docs/deploy_guides/docker/networking) multi-host and overlay Networks</li>
    <li>[Orchestrating](/docs/deploy_guides/docker/orchestrate) Aerospike clusters with Docker Compose</li>
    <li>[Recommendations](/docs/deploy_guides/docker/recommendations) for Production Deployments</li>
    <li>[Example](/docs/deploy_guides/docker/examples) Applications</li>
</ul>
