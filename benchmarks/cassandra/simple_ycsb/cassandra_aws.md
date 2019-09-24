---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" on AWS - Cassandra
description: Comparing Aerospike and Cassandra Database On AWS
styles:
  - /assets/styles/ui/steps.css
---


The Aerospike Benchmark AMI already has Cassandra installed in the directory /opt/cassandra/apache-cassandra-3.5. Most of the changes to the configuration have already been made. You will still need to do the following:

  * Perform the disk configurations in Step 5 manually. Note that the recommended instance type (c3.8xlarge) two SSDs, rather than the four in the benchmark. This will lead to lower performance than the benchmark. Amazon AWS uses device names such as `/dev/vxdb` rather than `/dev/sdb`.
  * Set the IP address seed and listen address (or interface) in Step 7. 

Go to the [Cassandra Install page](/docs/benchmarks/cassandra/simple_ycsb/cassandra_install.html) and complete the installation of Cassandra.

###Special Note
As mentioned previously, the Aerospike AMI is currently available in the US-West (N. California) region only.

To set up the YCSB client instance, follow these steps:

{{#warn markdown=true}}
In order to make a public AMI, we had to select Amazon Linux, rather than CentOS. Amazon Linux is binary compatible with CentOS, but has different logins on AWS.

Login by entering the following:
```bash
$ ssh -i [KEFYFILE] ec2-user@[INSTANCE_IP]
```
where KEYFILE is your private key used to launch the instance, and INSTANCE_IP is the public IP address of the instance.
{{/warn}}

