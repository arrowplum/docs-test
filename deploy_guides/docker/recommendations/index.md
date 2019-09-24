---
title: Recommendations for successfully deploying Aerospike with Docker
description: |
  Here are recommendations for ensuring a successful deployment of Aerospike database with Docker.
scripts:
  - /assets/scripts/utils/target_blank.js

---

### Customize aerospike.conf
The default Aerospike image is available at [Docker Hub](https://hub.docker.com/_/aerospike/). You can override the default `aerospike.conf` file at runtime since this is an exported volume from the Container, the example assumes the configuration file is in the local directory. A [`feature-key-file`](/docs/reference/configuration/index.html?show-removed=1#feature-key-file) is required to run the [Enterprise Edition Aerospike](https://cloud.docker.com/u/aerospike/repository/docker/aerospike/aerospike-server-enterprise) container. 

```bash
$ docker run -v $PWD:/etc/aerospike ...
```

Consult the [Configuration Guide](/docs/operations/configure) prior to finalizing the `aerospike.conf` configuration file for a given deployment.

#### Changing logging

The default logging is set to `Info` which may use up a lot of space inside the container. This is simple to change in the `aerospike.conf` file:

```bash
$ cat aerospike.conf
...
logging {

  # Log file must be an absolute path.
  file /var/log/aerospike/aerospike.log {
    context warning info
  }

  # Send log messages to stdout
  console {
    context any critical
  }
}
...
```

#### Specifying Network Interfaces

Your docker host/container may have multiple Network Interfaces. To prevent any confusion of which interface to use, it is recommended to lock down interface usage within the `network` stanza of `aerospike.conf` for service, heartbeat and fabric.

```bash
...
network {
	service {
		address eth0
		port 3000
		...
	}
	heartbeat {
		address eth0
		mode mesh
		...
	}
	fabric {
		address eth0
		port 3001
	}
...
```

### Affinity Rules

{{#note}}
Affinity Rules no longer apply to Swarm Mode
{{/note}}
[Docker Swarm](https://docs.docker.com/swarm/) has the ability to apply constraints and filters. These are automatically propagated from [Docker Compose](https://docs.docker.com/compose/) files if you are using this orchestration mechanism.

Simple rules that you will want to apply
- Prevent Aerospike nodes in the same cluster running on the same Docker Daemon
- Specifying a specific Hardware configuration

More information about the various options can be found in the [Docker Swarm documentation](https://docs.docker.com/swarm/scheduler/filter/) or take a look at the [Docker Examples](/docs/deploy_guides/docker/examples) in the the Aerospike documentation.

#### Prevent Aerospike nodes in the same cluster running on the same Docker Daemon

```bash
$ docker run -l com.aerospike.cluster==mycluster -e affinity:com.aerospike.cluster!=mycluster aerospike/aerospike-server
```

#### Specifying a specific Hardware configuration

If you want to constrain the Container to be deployed within a specific Hardware profile, this information can be passed when the Container is started. For example, assuming the Docker Daemon is started with a profile of SSD storage, then this can be used as a constraint for the Container:

```bash
$ docker -d --label storage=ssd  
$ docker run -e constraint:storage==ssd aerospike/aerospike-server
```

### Data Persistence

There are three main strategies for dealing with persistence, a critical choice for an Container running Aerospike.

#### Ephemeral Storage

Data is stored within the Container and the data will be lost when the Container is removed. This is a typical option used for Development & Test, but is not recommended for a Production deployment unless its combined with an External Volume (see below).

The default Docker Hub image for Aerospike will write data to `/opt/aerospike/data`, which is implicitly an Ephemeral volume.

#### Block Storage

Docker run offers the ability to expose hosts block devices to a running container. The  [--device](https://docs.docker.com/engine/reference/commandline/run/) option can be used to map a host block device within a container.

Example mapping /dev/sdc to /dev/xvdc on a running container:

```bash
$ docker run -tid --name aerospike -p 3000:3000 -p 3001:3001 -p 3002:3002 -p 3003:3003 -v /aerospike-server-enterprise.docker:/etc/aerospike/ --device '/dev/sdc:/dev/xvdc' -e "FEATURE_KEY_FILE=/etc/aerospike/features.conf" aerospike-server-enterprise:latest
``` 

#### External Volume

Docker has the ability to bind mount an external file-system into an exposed mount point in the Container. The default Docker Hub image for Aerospike exports a Volume `/opt/aerospike/data`. When the container starts, this can be mapped to a host file-system thus:

```bash
$ docker run -v $PWD:/opt/aerospike/data aeropsike/aerospike-server
```

The local directory (e.g. `$PWD`) will be used to store the data written by the Aerospike Server. A typical configuration would be to use the external volume as a [Shadow Volume](/docs/deploy_guides/aws/recommendations/index.html#shadow-device-configuration) so that any Ephemeral data is also written to the external volume as well. The external volume will survive the destruction of the Container and can be attached to a new Container if required and could be used for backups/restores (e.g. via an AWS EBS Snapshot).


