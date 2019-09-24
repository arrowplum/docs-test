---
title: Orchestrating Docker containers and Aerospike Clusters
description: |
  An example application built around an Aerospike cluster and deployed using Docker Containers.
scripts:
  - /assets/scripts/utils/target_blank.js
---

### Background
In this example we will show the following
  - building a multi-container application with a Web Application and an Aerospike Database
  - running and testing
  - injecting behavior for the production environment
  - adding a load balancer to scale the Web App
  - growing the Aerospike cluster

### Its a Counter!
Who does not love a Web Page Counter - but lets make it scalable! Here's what we want to track
  - a record for each Page hit
  - a summary of the total amount of hits
  - a summary for each hit received by each App Server

We will also want to have this data persistent and deployed in a set (Shipload?) of Containers.

#### Application Code
This example uses Python and Flask to create a very simple Web Application. Clearly this is not intended as an example of Production code!

```python
from flask import Flask, request
import aerospike
import os
import datetime
import time
import socket
import uuid

app = Flask(__name__)

config = {
  'hosts': [ (os.environ.get('AEROSPIKE_HOST', 'prod_aerospike_1'), 3000) ],
  'policies': { 'key': aerospike.POLICY_KEY_SEND }
}

host = socket.gethostbyname(socket.gethostname())

@app.route('/')
def hello():

    try:
        
        if aerospike.client(config).is_connected()==False:
            client = aerospike.client(config).connect()

        id = str(uuid.uuid1());

        # Insert the 'hit' record
        ts =  int(round(time.time() * 1000))
        client.put(("test", "hits", id), {"server": host, "ts": ts} )

        # Maintain our summaries for the grand total and for each server
        client.increment(("test", "summary", "total_hits"), "total", 1)

        client.increment(("test", "summary", host), "total", 1)
        
        (key, meta, bins) = client.get(("test","summary","total_hits"))
        
        # Return the updated web page
        return "Hello World! I have been seen by %s." % bins["total"]

    except Exception as e:
        return "Hummm - %s looks like we have an issue, let me try again" % "err: {0}".format(e)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

### Deploying into a Development Environment
Lets keep things simple and deploy our containers to a single Docker Daemon. This keeps the dev & test cycle simple and we can run it locally on our laptop.

#### Dockerfile for the application
We will use Docker to wrap up the dependencies for our Flask application and build an image with our application code baked in.

```bash
$ cat Dockerfile
FROM python:2.7
ADD . /code
WORKDIR /code
RUN apt-get update
RUN apt-get -y install python-dev
RUN apt-get -y install libssl-dev
RUN pip install --no-cache-dir flask requests aerospike
EXPOSE 5000
CMD python app.py
```

#### Docker compose file
Next we need to Orchestrate the various containers we need to deploy the Application for testing. We use a Docker Compose file to
  - describe the two services (Aerospike Database and a Web Application)
  - provide the steps required to build the code
  - inject any specific configuration

```bash
$ cat docker-compose.yml
web:
  build: .
  ports:
   - "5000:5000"
  links:
   - aerospike
  hostname: dev.awesome-counter.com
  environment:
    - AEROSPIKE_HOST=dev_aerospike_1   
aerospike:
  image: aerospike/aerospike-server
  volumes: 
    - $PWD:/opt/aerospike/etc
```

#### Launch the Orchestration
We can now launch the Orchestration and test the application.

```bash
$ docker-compose up -d
Creating dev_aerospike_1
Creating dev_web_1
```

#### Connect to the application
The Docker Daemon will expose an IP address that we can direct our browser to to view the application.

```bash
$ echo "$(docker-machine ip dev) dev.awesome-counter.com" | sudo tee -a /etc/hosts
$ open http://dev.awesome-counter.com:5000
```

In your browser you should see something like this

![Web Counter](/docs/deploy_guides/docker/assets/examples/dev/web-app.png)

#### Testing complete, push the Image
Now that we have finished testing, we can push the Docker Image to Docker Hub so that we can use the image in our production deployment. You will need to create an account at [Docker Hub](http://hub.docker.com) or use your own private Docker repository.

```bash
$ docker build -t myrepo/web-app-as .
Step 1 : FROM python:2.7
 ---> 7436fc1e0756
...
Successfully built 7d1460017d70

$ docker push myrepo/web-app-as
```

### Deploying into production
Typically there are additional requirements and operational needs that need to be reflected in a Production deployment. For example
- Add in a Load Balancer to scale the Web tier
- Inject production constraints
- Ensure the correct network segmentation between the tiers

We want to end-up with a deployment topology that looks like:

![Deployment Topology](/docs/deploy_guides/docker/assets/networking/networking-app.png)

#### Create a Docker Swarm cluster
{{#note}}
These instructions were written for Standalone Swarm and will require adjustments for Swarm Mode as of Docker 1.12.
{{/note}}
Creating the Docker Swarm Cluster is discussed in the [Orchestration chapter](/docs/deploy_guides/docker/orchestrate) and has been encapsulated in a [bash script](/docs/deploy_guides/docker/assets/prod/create_swarm.sh) you can use to setup the Cluster.

#### Load Balancer (HAProxy) and Interlock
We will use [HAProxy](http://haproxy.org) to act as a load balancer for the App Servers. As we spin up additional Containers running the Web App, we need to reconfigure HAProxy. This is easily done manually, but we want to automatically register a new endpoint when a Web App Containers starts and de-register when the Container stops. We will use an open source project [Interlock](https://github.com/ehazlett/interlock) from our good friend Evan Hazlett. This listens for Docker events and then will automatically re-configure HAProxy as Web App Containers start and stop. This will be encapsulated into a separate Docker Compose file.

```bash
$ cat haproxy.yml
haproxy-server:
  image: ehazlett/interlock:latest
  environment:
   - "DOCKER_HOST"
  volumes:
  # boot2docker images use the following location for certificates etc.
   - "/var/lib/boot2docker:/etc/docker"
  ports:
   - "80:8080"
  command: "--swarm-url=$DOCKER_HOST --swarm-tls-ca-cert=/etc/docker/ca.pem --swarm-tls-cert=/etc/docker/server.pem --swarm-tls-key=/etc/docker/server-key.pem --debug --plugin haproxy start"
haproxy-app:
  hostname: prod.awesome-counter.com
  ports:
  - 5000
```

#### Docker compose file
We will now define the three services that make up the production version of our application:
  - HAProxy for load balancing
  - Aerospike for the Database
  - Web application 

```bash
$ cat docker-compose.yml
haproxy:
  extends:
    file: haproxy.yml
    service: haproxy-server
  environment:
    - "constraint:node==swarm-0"
  net: "bridge"  
web:
  image: myrepo/web-app-as
  extends:
    file: haproxy.yml
    service: haproxy-app
  environment:
    - AEROSPIKE_HOST=prod_aerospike_1   
aerospike:
  extends:
    file: aes_base.yml
    service: aerospike-server
  image: aerospike/aerospike-server:3.6.4
  environment:
    - "affinity:com.aerospike.cluster!=awesome-counter"
  volumes:
    - $PWD:/etc/aerospike
```

##### haproxy
This extends the base `haproxy.yml` file to add the following
  - a Docker Swarm `constraint` to pin HAProxy to a specific node
  - a directive to used the `bridge` network, ensuring HAProxy can accept external Internet traffic

##### web
This defines:
  - the image we want to use (from the previous step)
  - the first node in the Aerospike cluster the Python Client will connect to. After that connection is established then the Smart Client&#0153; will automatically discover the cluster topology.

##### aerospike
This extends the base `aes_base.yml` file to add the following:
  - a concrete version of Aerospike to use (i.e. 3.6.4)
  - an Docker Swarm `affinity` rule that ensures one and only one Aerospike containers runs per Docker Daemon
  - maps the current directory to the volume `/etc/aerospike` so that the `aerospike.conf` can be overridden

#### Deployment
Since we want the Containers to span multiple Docker Daemons using the overlay driver for multi-host networking, we need pass Docker Compsoe that information.

{{#note}}
The `--x-networking` flag enables Docker Compose to make use of multi-host networking. This is an "experimental" flag and in a future version this flag will not be required.
{{/note}}

```bash
$ eval $(docker-machine env --swarm swarm-0)
$ docker-compose --x-networking up -d
Creating prod_haproxy_1
Creating prod_web_1
Creating prod_aerospike_1
```

#### Connect HAProxy to the network used by Aerospike
As we saw above we added the directive `--net` for HAProxy so that it connected to the public network (and thus can be addressed over the Internet). We now need to connect HAProxy to the Overlay network that was created that is connecting the Aerospike nodes.

<pre>
<code>$ docker $(docker-machine config swarm-0) network connect prod $(docker inspect -f "&#123;&#123;.Id}}" prod_haproxy_1)</code>
</pre>

#### Test the deployment
The Docker Daemon will expose an IP address that we can direct our browser to to view the application.

```bash
$ echo "$(docker-machine ip swarm-0) prod.awesome-counter.com" | sudo tee -a /etc/hosts
$ open http://prod.awesome-counter.com
```

Your browser should look the same as it did when you tested the development deployment.

### Scaling the Web Application Tier
Docker Compose has the ability to scale any of the given services in the yaml file with the `scale` parameter. Since we have already integrated Interlock, as we start new Web App Containers then HAProxy will automatically be re-configured.

```bash
$ docker-compose --x-networking scale web=4
Creating and starting 2 ... done
Creating and starting 3 ... done
Creating and starting 4 ... done
```

We can inspect HAProxy to see the new nodes behind the load balancer.

```bash
$ open http://prod.awesome-counter.com/haproxy?stats
```

The default credentials are `stats/interlock` when you are prompted! You should see a display something like

![HAProxy](/docs/deploy_guides/docker/assets/examples/prod/haproxy.png)


### Scaling the Aerospike Cluster
As the final step we can now scale the Aerospike cluster, again using Docker Compose

```bash
$ docker-compose --x-networking scale aerospike=3
Creating and starting 2 ... done
Creating and starting 3 ... done
```

Since Docker [does not support multi-cast networks](https://github.com/docker/libnetwork/issues/552) on Overlay networks, we need to assemble the cluster over the Mesh network via the `tip` command. We need you specify the `--net` parameter to Docker to ensure that we connect to the network that the Aerospike cluster is running on.

<pre>
<code>$ docker run --net prod aerospike/aerospike-tools asinfo -v "tip:host=$(docker inspect -f '&#123;&#123; .NetworkSettings.Networks.prod.IPAddress }}' prod_aerospike_2);port=3002" -h prod_aerospike_1</code>

<code>$ docker run --net prod aerospike/aerospike-tools asinfo -v "tip:host=$(docker inspect -f '&#123;&#123; .NetworkSettings.Networks.prod.IPAddress }}' prod_aerospike_3);port=3002" -h prod_aerospike_1</code>
</pre>

You can inspect the Aerospike Cluster config

```bash
$ docker run --net prod aerospike/aerospike-tools asadm -h prod_aerospike_1 -e info
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Service Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
            Node   Build   Cluster      Cluster     Cluster    Free   Free   Migrates          Principal   Objects     Uptime   
               .       .      Size   Visibility   Integrity   Disk%   Mem%          .                  .         .          .   
172.18.0.2         N/E         N/E   N/E          N/E           N/E    N/E   N/E        N/E                    N/E   N/E        
172.18.0.3         N/E         N/E   N/E          N/E           N/E    N/E   N/E        N/E                    N/E   N/E        
prod_aerospike_1   3.6.4         3   True         True          100    100   (0,0)      prod_aerospike_3   0.000     01:11:40   
prod_aerospike_2   3.6.4         3   True         True          100    100   (0,0)      prod_aerospike_3   0.000     00:06:52   
prod_aerospike_3   3.6.4         3   True         True          100    100   (0,0)      prod_aerospike_3   0.000     00:06:52   
Number of rows: 5

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
            Node               Node                         Fqdn                Ip   Client     Current     HB        HB   
               .                 Id                            .                 .    Conns        Time   Self   Foreign   
172.18.0.2         000000000000000    172.18.0.2:3000              172.18.0.2:3000      N/E         N/E    N/E       N/E   
172.18.0.3         000000000000000    172.18.0.3:3000              172.18.0.3:3000      N/E         N/E    N/E       N/E   
prod_aerospike_1   BB90300000A4202    prod_aerospike_1.prod:3000   10.0.0.3:3000          1   187913119      0      1052   
prod_aerospike_2   BB90700000A4202    prod_aerospike_2:3000        10.0.0.7:3000          1   187915689      0      1051   
prod_aerospike_3   *BB90800000A4202   prod_aerospike_3.prod:3000   10.0.0.8:3000          1   187915689      0       760   
Number of rows: 5
```

Finally, if you have refreshed the web page of the Application a few times, you will be able to see the Hits that were recorded in total, along with the hits per Application Server and the individual records

```bash
$ docker run -it --rm --net prod aerospike/aerospike-tools aql -h prod_aerospike_1
aql>  select * from test.hits
| key                                    | ts            | server     |
+----------------------------------------+---------------+------------+
| "d675c5b2-a37e-11e5-a948-02420a000005" | 1450220086392 | "10.0.0.5" |
| "d4652330-a37e-11e5-98a6-02420a000004" | 1450220082927 | "10.0.0.4" |
| "dc839d4e-a37e-11e5-a948-02420a000005" | 1450220096549 | "10.0.0.5" |
| "b7a4bff8-a37e-11e5-8bd2-02420a000002" | 1450220034690 | "10.0.0.2" |
...

aql>  select * from test.summary
+--------------+-------+
| key          | total |
+--------------+-------+
| "10.0.0.2"   | 8     |
| "10.0.0.5"   | 8     |
| "total_hits" | 33    |
| "10.0.0.6"   | 9     |
| "10.0.0.4"   | 8     |
+--------------+-------+
5 rows in set (0.025 secs)
```

### Additional Information
Downloadable versions of the files used above can be found below.
- Development Scripts
    - [app.py](/docs/deploy_guides/docker/assets/examples/dev/app.py)
    - [docker-compose.yml](/docs/deploy_guides/docker/assets/examples/dev/docker-compose.yml)
    - [Dockerfile](/docs/deploy_guides/docker/assets/examples/dev/Dockerfile)
- Production Scripts
    - [aerospike.conf](/docs/deploy_guides/docker/assets/examples/prod/app.py)
    - [ase_base.yml](/docs/deploy_guides/docker/assets/examples/prod/ase_base.yml)
    - [create_swarm.sh](/docs/deploy_guides/docker/assets/examples/prod/create_swarm.sh)
    - [docker-compose.yml](/docs/deploy_guides/docker/assets/examples/prod/docker-compose.yml)
    - [haproxy.yml](/docs/deploy_guides/docker/assets/examples/prod/haproxy.yml)




