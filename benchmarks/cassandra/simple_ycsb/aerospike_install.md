---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" with Manual Installation - Aerospike
description: Installing Aerospike on Linux to run a benchmark
styles:
  - /assets/styles/ui/steps.css
---

Prior to this step, you should have already [set up your client host](/docs/benchmarks/cassandra/simple_ycsb/client_install.html). 

These instructions take you through planning and installing an Aerospike cluster for use in benchmarking. You should have a cluster of Linux machines with some you intend to run as servers.

These instructions are written assuming a CentOS 6 Linux distribution. If you plan to use a different Linux distribution, you will need to modify the instructions accordingly.

{{#steps}}

Installing Aerospike should be simple. You will need to run the steps below for each database host in your cluster.

{{!---
  ############################################################################
  #
  # STEP 1 - Planning
  #
  ############################################################################
---}}

{{#steps-step 1 "Planning" markdown=true}}
As you begin planning your own benchmark project, it is important for you to consider your test environment. There are several important factors to take into account. Naturally, changing the values in the benchmark will lead to different results.

# Database Servers
| Component | Hardware Used in Blog | Recommendation | Notes |
| --------- | ----- | --- | --- |
| CPU       | Dual Intel(R) Xeon(R) CPU E5-2665 0 @ 2.4GHz (8 cores) | Dual quad core 2.0GHz | For most testing, Aerospike generally does not need a fast CPU. It is only needed when the per node speeds exceed about 500ktps. For Cassandra, CPU is often the biggest bottleneck, so it will benefit from a faster CPU. |
| RAM       | 32 GB 1333 MHz | 32 GB RAM | The most important factor for RAM will the be actual volume of RAM. While both databases will naturally benefit from faster RAM, the primary concern should be that you have enough RAM to store index, data, and cache. |
| SSD       | 4 x 200 GB Intel s3700 | 4 x 200 GB SSD | Choose SSDs that have good performance. Aerospike publishes a set of performance benchmarks for SSDs (including those of cloud vendors). These are available on the [Aerospike SSD Certification Page](/docs/operations/plan/ssd/ssd_certification.html). | 
| Network   | Intel Corporation 82599ES 10-Gigabit |  10 Gb | One factor that can impact performance is the number of network queues on the NIC. Most 10Gb NICs today have at least 8 network queues and are generally good for benchmarks. In the best case scenario, the NIC adapts the number of queues to the number of cores on the server. There are 1Gb NICs that also support multiple queues. Note that in VMs, such as AWS, you may be limited to only 1 or 2 queues. To find the number of queues on your NIC, first identify the NIC's device id (e.g. eth2). Then, look at the file "/proc/interrupts". Transactions queues will have the format "[DEVICEID]-TxRx-[#]". The one with the highest number for the device will denote the total number of queues - starting at zero. Thus, if you see 16 queues with the highest being "eth2-TxRx-15", you have 16 queues for that NIC.|
| Operating System | CentOS 6.7 | CentOS 6.7 | These instructions and the scripts created for this benchmark were created on Linux CentOS 6.7 and will not work on another flavor or major version without modifications. |

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 2 - Host Setup
  #
  ############################################################################
---}}
{{#steps-step 2 "Host Setup" markdown=true}}

First, make sure that you have the IP addresses for all the database hosts in your cluster.

Prior to installing either database, there are some things you will want to set up on the server for both databases.

## Stop iptables
By default, CentOS 6 has iptables turned on. This will stop incoming connections. Turn off iptables:

```bash
dbhost$ sudo service iptables stop
dbhost$ sudo chkconfig iptables off
```

## Balance Network Queues

In our testing with the listed hardware, we found that all systems performed better when we performed network queue tuning. Otherwise, performance will be limited to one CPU ( often CPU 0 ) hitting a maximum. The standard Linux tool 
`irqbalance` is recommended to avoid this problem. Tune all hosts thusly, since this can impact either/both client and server machines. This can be done most easily in one of 3 ways. In our instructions, we chose to use the oneshot method mentioned (second below) in our instructions. 

This step can be skipped if your NIC only has one network queue.

If you choose to use one of the methods that use irqbalance, you may need to first install it:

```bash
dbhost$ sudo yum install -y irqbalance
```

  * You can let the system do this automatically by using irqbalance. Recent versions of Linux can do this automatically. Simply execute (as root):

    ```bash
    dbhost$ sudo service irqbalance start
    ```
  * During your initial testing, you can also turn on irqbalance to balance the queues, and then immediately exit. This must be done while applying a load to make sure that irqbalance knows how to balance. 

    ```bash
    dbhost$ sudo service irqbalance stop
    dbhost$ sudo irqbalance --oneshot
    ```
  * You can also do the balancing manually. While doing so requires more manual work, this will produce optimal results. An excellent guide to this can be found at [Alex On Linux](http://www.alexonlinux.com/smp-affinity-and-proper-interrupt-handling-in-linux).

## Install other packages

The only other packages you will need for Aerospike is NTP and wget. First, install them using yum:

```bash
dbhost$ sudo yum install -y ntp wget
```

After installing it, start NTP to make sure that all the clocks on all servers (both client and server) are in agreement:


```bash
dbhost$ sudo /etc/init.d/ntpd start
```
{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 3 - Install Aerospike
  #
  ############################################################################
---}}
{{#steps-step 3 "Install Aerospike" markdown=true}}

Install Aerospike onto a generic Linux distribution by following the [instructions on installing Aerospike on Redhat/CentOS](/docs/operations/install/linux/el6/). 

While you should configure the cluster per the instructions, you should also make the following change for the namespaces in the Aerospike configuration file (`/etc/aerospike/aerospike.conf`):

  1. Get rid of all other namespaces. In the default config file, there are sample namespaces called `bar` and `test`. 
  1. Create a new namespace called `ycsb`. This should use the SSDs you are using in the server as storage.
    1. Make sure `data-in-memory` is not set to `true`, or the system will use RAM as main storage and SSD only for persistence.
    1. Set `defrag-sleep` to "0" (zero). Normally, Aerospike will pause in between defrags (which serves the same purpose as Cassandra's compactions, but is run continuously with short pauses). In stressful environments, it is recommended to turn off the pauses.


## Example snippet of Aerospike config file (/etc/aerospike/aerospike.conf)
Other than the mesh-seed-address-port parameters and the device (SSD), this file is valid for a server with 32 GB RAM, 2 x quad core CPU, and 4 SSDs.

Here is an example of how the namespace should look when completed:

```asciidoc
# Aerospike database configuration file.

service {
	user root
	group root
	paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
	pidfile /var/run/aerospike/asd.pid
# TODO: service-threads and transaction-queues should both match the number of vCPU on the instance
	service-threads 16
	transaction-queues 16
	transaction-threads-per-queue 4
	proto-fd-max 15000
}

logging {
	# Log file must be an absolute path.
	file /var/log/aerospike/aerospike.log {
		context any info
	}
}

network {
	service {
		address any
		port 3000
	}

	heartbeat {
		mode mesh
# TODO: There should be one mesh-seed-address-port pair for each node in your cluster.
# This includes the one you are on
		mesh-seed-address-port [IP_ADDR_1] 3002
		mesh-seed-address-port [IP_ADDR_2] 3002
		mesh-seed-address-port [IP_ADDR_3] 3002
		port 3002

		# To use unicast-mesh heartbeats, remove the 3 lines above, and see
		# aerospike_mesh.conf for alternative.

		interval 250
		timeout 25
	}

	fabric {
		port 3001
	}

	info {
		port 3003
	}
}

namespace ycsb {
	replication-factor 2
# TODO: You should allow for a few GB of RAM for the Operating System, but can use the rest.
	memory-size 22G

	storage-engine device {
# TODO: There should be one device parameter for each SSD.
# Partitions are allowed, but all devices in a namespace should be the same size/performance.
		device [DEVICE_ID]
		device [DEVICE_ID]
		device [DEVICE_ID]
		device [DEVICE_ID]
#		device /dev/sdb # Example SSD device

		scheduler-mode noop
		write-block-size 128K
		defrag-sleep 0
	}
}
```

{{/steps-step}}
{{!---
  ############################################################################
  #
  # STEP 4 - Next Steps
  #
  ############################################################################
---}}
{{#steps-step 4 "Next Steps" markdown=true}}

Now, you can run the benchmark. Go to the [Running YCSB on Aerospike Page](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aerospike.html) for instructions on running the actual benchmark.

{{/steps-step}}

{{/steps}}
