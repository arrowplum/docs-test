---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" on AWS - Aerospike
description: Comparing Aerospike and Cassandra Database On AWS
styles:
  - /assets/styles/ui/steps.css
---

Now you should launch your Aerospike database instances.

{{#note markdown=true}}
As mentioned previously, the Aerospike AMI is currently available in the US-West (N. California) region only.
{{/note}}

{{#info markdown=true}}
The Aerospike AMI is based on Amazon Linux, not CentOS. Indeed, the best practice from most vendors is to use Amazon Linux, which shares binary and other compatibility with CentOS 6 but includes Amazon-approved drivers and tuning.
{{/info}}

{{#info markdown=true}}
You will need to log into your instances. As this AMI is based on Amazon Linux, login by entering the following:
```bash
$ ssh -i [KEFYFILE] ec2-user@[INSTANCE_IP]
```
where KEYFILE is your private key used to launch the instance, and INSTANCE_IP is the public IP address of the instance.
{{/info}}

{{#steps}}

{{!---
  ############################################################################
  #
  # STEP 1 - Setting Up Aerospike Instances
  #
  ############################################################################
---}}
{{#steps-step 1 "Setting Up Aerospike Instances" markdown=true}}

Set up 3 c3.8xlarge instances to serve as the database cluster, which you will use for both Aerospike and Cassandra. 

Both Aerospike and Apache Cassandra have already been installed on the AMI, so no further software needs to be installed.

To set up the instances for your run of the benchmark on Amazon AWS:
  1. Login to the AWS console and navigate to the "EC2 dashboard".
  1. Make sure you are using the "US West (N. California)" region (upper right-hand corner).
  1. Go to the "Instances" page (left-hand menu bar).
  1. Click on "Launch Instance".
  1. In "Step 1: Choose an Amazon Machine Image (AMI)", click on "Community AMIs" on the left-hand bar.
  1. In the "Search community AMIs" text box, enter `aerobench` to search for the Aerospike Benchmark AMI. Select the latest version of the aerobench AMI.
  1. In "Step 2: Choose an Instance Type", select the instance type for the database instances, which is c3.8xlarge. Select this and click on "Next: Configure Instance Details" on the bottom right.
  1. In "Step 3: Configure Instance Details", enter "3" into the number of instances. You may need to change "Auto-assign Public IP" to "Enable" in order to get a public IP address. Click on "Next: Add Storage."
  1. In "Step 4: Add Storage", for the Root volume, change the size to 18 GiB. This will make room for Cassandra's commit logs. Make sure that in the column "Volume Type",  "Instance Store 0" is set to device "/dev/sdb", and "Instance Store 1" is set to device "/dev/sdc". Click on "Next: Tag Instance".
  1. In "Step 5: Tag Instance", you may enter any tags you feel are relevant here. Click on "Next: Configure Security Group".
  1. In "Step 6: Configure Security Group", client on "Select an existing security group" and select the "aerobench" security group created above. Click on "Review and Launch" on the lower right-hand side.
  1. Check all values and hit "Launch" in the lower right-hand corner.
  1. Select your keys, click on the acknowledgment, and hit "Launch Instances".


Your instances should launch in roughly 5-10 minutes.

Record the internal IP addresses allocated to your cluster. You'll be using them frequently.

{{/steps-step}}


{{!---
  ############################################################################
  #
  # STEP 2 - Configuring Aerospike
  #
  ############################################################################
---}}
{{#steps-step 2 "Configuring Aerospike" markdown=true}}

Prior to starting up Aerospike, you must edit the configuration files to manually 
describe the cluster.

{{#note markdown=true}}
If you are using 3 `c3.8xlarge` instances as your database hosts as described in these instructions, the only needed change should be to the network heartbeat stanza. 
{{/note}}

## Edit the configuration file

Log into each server with SSH, and edit each configuration file.

```bash
dbhost$ sudo vi /etc/aerospike/aerospike.conf
```

## Add the private IP addresses to the config file

While you only need to add the IP address of one other node, it is better practice to add the IP address of all the nodes in the cluster. Look for the following section, and replace the `[IP_ADDR_X]` sections below with the internal IP addresses of other cluster nodes.

```asciidoc
        heartbeat {
			mode mesh
# TODO: There should be one mesh-seed-address-port pair for each node in your cluster.
# This includes the one you are on
			mesh-seed-address-port [IP_ADDR_1] 3002
			mesh-seed-address-port [IP_ADDR_2] 3002
			mesh-seed-address-port [IP_ADDR_3] 3002
			port 3002
...
```

After editing, the file should look as follows:

```asciidoc
        heartbeat {
                mode mesh
# TODO: There should be one mesh-seed-address-port pair for each node in your cluster.
# This includes the one you are on
                mesh-seed-address-port 172.31.25.40 3002
                mesh-seed-address-port 172.31.27.46 3002
                port 3002

                # To use unicast-mesh heartbeats, remove the 3 lines above, and see
                # aerospike_mesh.conf for alternative.

                interval 250
                timeout 25
        }
```

## Consider changing other parameters if you have different instances
If you are using instance types other than c3.8xlarge, you should make other changes to the Aerospike configuration file, as follows:

| Area | Parameter | Current Value | Notes|
| --- | --- | --- | --- |
| network:heartbeat | mesh-seed-address-port | [None] | _REQUIRED_ - There should be one line with the private IP address of each node in your cluster. |
| service | service-threads | 32 | Adjust this to the number of vCPUs on your instance. |
| service | transaction-queues | 32 | Match this to the number of service threads. |
| namespace | memory-size | 26 GB | This should allow at least 2 GB of RAM for the OS. |
| namespace:device | device | /dev/xvdb and /dev/xvdc | There should be one line for each SSD on the instance. |


## Sample Aerospike Configuration File
If you are using c3.8xlarge instances, the full `/etc/aerospike/aerospike.conf` file should look like the following:

```asciidoc

# Aerospike database configuration file.

service {
	user root
	group root
	paxos-single-replica-limit 1 # Number of nodes where the replica count is automatically reduced to 1.
	pidfile /var/run/aerospike/asd.pid
# TODO: service-threads and transaction-queues should both match the number of vCPU on the instance
	service-threads 32
	transaction-queues 32
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
	memory-size 26G

	storage-engine device {
# TODO: There should be one device line for each SSD.
# Partitions are allowed, but all devices in a namespace should be the same size/performance.
		device /dev/xvdb
		device /dev/xvdc
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
  # STEP 3 - Next Steps
  #
  ############################################################################
---}}
{{#steps-step 3 "Next Steps" markdown=true}}

Now you can run the benchmark. Go to the [Running YCSB on Aerospike Page](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aerospike.html) for instructions on running the actual benchmark.

{{/steps-step}}

{{/steps}}
