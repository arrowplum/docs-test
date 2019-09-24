---
title: Multi-Node Apache Hadoop Cluster Setup
description: Step by step guide to setting up a Multi-Node Apache Hadoop Cluster
---
# Planned Topology


<center>
{{#figure "" "Cluster Topology" size="large" }}
<img src="/docs/connectors/assets/images/ClusterTopology.png" >
{{/figure}}
</center>

## Multi-Node Apache Hadoop 2.60 Cluster Setup

- Allocate four 4GBRam/2Core, Ubuntu 14.4 instances in the cloud (use a cloud service provider of your choice).
  - We called them: ztg-master (IP addr: x.y.z.156) ztg-slave1 (IP addr: x.y.z.157) ztg-slave2 (IP addr: x.y.z.158) & ztg-client (IP addr: x.y.z.209)
- Note down their IP addresses. We refer to them as: x.y.z.nnn in this example. *Use actual ip address if repeating this example.*  
- Start with first instance: ztg-master   
- Download Java (scp to slave nodes later).
- Download Hadoop 2.6.0 tarball (scp to slave nodes later.)
- On ztg-client, we will need Hadoop libraries, Aerospike Hadoop Connector, Java, Maven and Aerospike.
- ztg-master
  - Install Java
  - We will add a user `hduser` that we will treat as hadoop administrator and use to install all hadoop and related components.  Later, on another machine (edge node) we will also add another user `hdclient` who will access the Apache Hadoop cluster for running map-reduce jobs.
  - Install Apache Hadoop.
  - Update env settings
  - Update site xmls
  - On master, identify slave nodes.
  - Shell scripts to start and stop daemons.
- Daemons to run:
  - ztg_master: NameNode, ResourceManager, Web App Proxy Server, MapReduce Job History Server. 
  - ztg_slave(s): NodeManager, DataNode

## Create Instances on the Cloud

Select a cloud service provider of your choice and obtain 4 instances with a minimum 4GB RAM, 2 core, typically 75GB HDD running Ubuntu Server 14.04 . Each server in this example has a public IP address with a 10Gbps network bandwidth. Upon successfully provisionining the servers, root access and password should be available.

## Prepare to Install Hadoop. 

Login as root. 

### Install Java JDK
This is the recommended and easiest option. This will install OpenJDK 6 on Ubuntu 12.04 and earlier and on 12.10+ it will install OpenJDK 7.
Installing Java with apt-get is easy. First, update the package index:

`sudo apt-get update`


Then, check if Java is not already installed:

`java -version`


You need the Java Development Kit (JDK), which is needed to compile Java applications (for example Apache Ant, Apache Maven, Eclipseand IntelliJ IDEA).  
Execute the following command:

`sudo apt-get install default-jdk`


This is everything that is needed to install Java.

### Create user group "hadoop".  
```
#sudo addgroup hadoop
Adding group `hadoop' (GID 1000) ...
Done.

#sudo adduser --ingroup hadoop hduser
Adding user `hduser' ...
Adding new user `hduser' (1001) with group `hadoop' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: (Keep same on all nodes for convenience)
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] Y
```
Add hduser to group sudo on ztg-master
```asciidoc
#sudo adduser hduser sudo
```  
### ssh and sshd 

Typically, the cloud service provider’s Ubuntu image must already have these. 
(Otherwise you can get them using: `sudo apt-get install ssh` )  
**Check:**  
```asciidoc  
#which ssh  
/usr/bin/ssh
#which sshd
/usr/sbin/sshd
```

### Disabling IPV6 on Ubuntu

```asciidoc  
root@ztg-master:~# sudo vi /etc/sysctl.conf
```
Add the following lines at the end:

```asciidoc  
#disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Check:

```asciidoc  
root@ztg-master:~# cat /proc/sys/net/ipv6/conf/all/disable_ipv6 
0
root@ztg-master:~# sudo sysctl -p
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
root@ztg-master:~# cat /proc/sys/net/ipv6/conf/all/disable_ipv6 
1
```

(Must do: `sudo sysctl -p` for these changes to take effect).

### Add ssh keys
Note: Do this for hduser only on the master node (don’t do on slave nodes).
```asciidoc  
# su hduser
Password: 
```
Change to hduser home directory and generate ssh key of type rsa and blank passphrase.
```apache  
hduser@ubuntu:/root$cd ~
hduser@ubuntu:/home/hduser$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hduser/.ssh/id_rsa): 
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
76:19:c2:93:7b:1a:fb:78:74:16:c9:1c:70:e2:ed:ad hduser@ubuntu
The key's randomart image is:
+--[ RSA 2048]----+
|          o..    |
|       . o +.    |
|        = oo.o   |
|         + +=.   |
|        S + ...  |
|       . *. o.   |
|        o. oE    |
|         o.      |
|        ...      |
+-----------------+
```

### Enable SSH Access

```asciidoc  
hduser@ubuntu:/home/hduser$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```
**Check:** (you should be able to login to localhost without password)
```apache  
hduser@ubuntu:/home/hduser$ ssh hduser@localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is fe:bc:c4:94:f3:e0:33:1b:85:e0:d6:3b:d9:6e:48:cf...
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-45-generic x86_64)

 * Documentation:  https://help.ubuntu.com/


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```

This must be done only on master and after creating slaves same as above (except do not do ssh-keygen on the slaves).
(Assuming that you have the slave node also configured the same way up till this point - do the following only the ztg-master while logged in as: `hduser`)
```asciidoc  
$ chmod 600 authorized_keys
```
Copy this key to ztg-slave1 to enable password less ssh.
But before you can do this, you need the public IP address of 
ztg-slave1 and add it to /etc/hosts file as root on ztg-master.
```asciidoc  
sudo vi /etc/hosts
```
Insert after localhost entries, and, save and close:
```asciidoc
x.y.z.157 ztg-slave1
```
Now back as hduser on ztg-master, do the following:  
`$ ssh-copy-id -i ~/.ssh/id_rsa.pub ztg-slave1`  (will ask for hduser password on ztg-slave1).
Make sure you can do a password less ssh using following command.  
`$ ssh ztg-slave1`
Also, for the cluster to not give error on `ztg-master:54310` (later config) also change in /etc/hosts:  
*is:* `127.0.1.1 ztg-master`  
*change to:* `x.y.z.156 ztg-master` (use the public IP Address)  

Finally, all slaves must know the ztg-master in their /etc/hosts file.
On ztg-slave1 and ztg-slave2, add in /etc/hosts file: `x.y.z.156 ztg-master`

Also, do the following on all slaves. (Otherwise if you submit map-reduce job from ztg-master, you will get `java.net.UnknownHostException` error.)
 
*is:* `127.0.1.1 ztg-slave1`  
*change to:* `x.y.z.157 ztg-slave1`(use the public IP Address)

### Download and Install Apache Hadoop

Go to Apache mirror site, download tar file. Optionally, do a quick check for tampering.
To perform a quick check using SHA-256:  
1. Download the release `hadoop-X.Y.Z-src.tar.gz` from a [mirror site](http://www.apache.org/dyn/closer.cgi/hadoop/common).
2. Download the checksum `hadoop-X.Y.Z-src.tar.gz.mds` from [Apache](https://dist.apache.org/repos/dist/release/hadoop/common/ ).
3. `shasum -a 256 hadoop-X.Y.Z-src.tar.gz`

```asciidoc
hduser@ztg-master:~$ wget http://mirror.cogentco.com/pub/apache/hadoop/common/stable/hadoop-2.6.0.tar.gz
--2015-06-25 16:52:23--  http://mirror.cogentco.com/pub/apache/hadoop/common/stable/hadoop-2.6.0.tar.gz
Resolving mirror.cogentco.com (mirror.cogentco.com)... 204.157.3.70
Connecting to mirror.cogentco.com (mirror.cogentco.com)|204.157.3.70|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 195257604 (186M) [application/x-gzip]
Saving to: ‘hadoop-2.6.0.tar.gz’

100%[=================================================>] 195,257,604 35.3MB/s   in 6.1s   

2015-06-25 16:52:30 (30.7 MB/s) - ‘hadoop-2.6.0.tar.gz’ saved [195257604/195257604]

hduser@ztg-master:~$ ls
hadoop-2.6.0.tar.gz
```

Now scp it to ztg-slave1 (we already set up ssh credentials):

```asciidoc
hduser@ztg-master:~$ scp /home/hduser/hadoop-2.6.0.tar.gz hduser@ztg-slave1:/home/hduser
hadoop-2.6.0.tar.gz                                                                                                    100%  186MB  37.2MB/s   00:05    
hduser@ztg-master:~$
```

Now as `hduser`, in `/home/hduser` (home) directory, where you have the tarball, untar it in both master and slaves.

```asciidoc
hduser@ztg-master:~$ tar zxvf hadoop-2.6.0.tar.gz 
hduser@ztg-master:~$ mv hadoop-2.6.0 hadoop
hduser@ztg-master:~$ ls
hadoop  hadoop-2.6.0.tar.gz
```

Now that we have ztg-master and ztg-slave1,2 running with hadoop installed, let us do all the required configurations and then launch the daemons.

Copy and paste following lines into your `.bashrc` file under `/home/hduser`. Do this step on master and every slave node.

```asciidoc
hduser@ztg-master:~$ vi ~/.bashrc

#HADOOP VARIABLES START
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_INSTALL=/home/hduser/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib/native"
#HADOOP VARIABLES END
```

Make sure the `java.library.path` is set to `lib/native` or you may get hadoop native library not found, *using default java library* error.

(*TIP*: Use `ssh hduser@ztg-slave1 … `and cut and paste changes.)

Update `JAVA_HOME` in `/home/hduser/hadoop/etc/hadoop/hadoop_env.sh` to following. Do this step on master and every slave node.
```asciidoc
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
```
(**Note:** Don’t skip this or daemons won’t start.)

### Update Configuration Files

Modify `core-site.xml` on Master and Slave nodes with following options. 
Master and slave nodes should use the same value for this 
property: **fs.defaultFS**, and should be pointing to master node only.

```asciidoc
hduser@ztg-master:~$mkdir tmp
hduser@ztg-master:vi /home/hduser/hadoop/etc/hadoop/core-site.xml
```

Insert between `<configuration>` tags:  

```xml
<property>
  <name>hadoop.tmp.dir</name>
  <value>/home/hduser/tmp</value>
  <description>Temporary Directory.</description>
</property>

<property>
  <name>fs.defaultFS</name>
  <value>hdfs://ztg-master:54310</value>
  <description>Use HDFS as file storage engine</description>
</property>
```

Modify `mapred-site.xml` on *Master node only* with following options.

```asciidoc
hduser@ztg-master:vi /home/hduser/hadoop/etc/hadoop/mapred-site.xml
```

```xml
<property>
 <name>mapreduce.jobtracker.address</name>
 <value>ztg-master:54311</value>
 <description>The host and port that the MapReduce job tracker runs
  at. If “local”, then jobs are run in-process as a single map
  and reduce task.
</description>
</property>
<property>
 <name>mapreduce.framework.name</name>
 <value>yarn</value>
 <description>The framework for running mapreduce jobs</description>
</property>
```

Modify `hdfs-site.xml` on Master and Slave Nodes. 
Before that, as `hduser`, create the following directories on all the nodes, master and slaves.

```asciidoc
cd /home/hduser/hadoop
hduser@ubuntu:/home/hduser/hadoop$ mkdir -p ./yarn_data/hdfs/namenode
hduser@ubuntu:/home/hduser/hadoop$ mkdir -p ./yarn_data/hdfs/datanode
```

So you should have on ztg-master, ztg-slave1 and ztg-slave2:

```asciidoc
/home/hduser/hadoop/yarn_data/hdfs/namenode
/home/hduser/hadoop/yarn_data/hdfs/datanode
```

Add following three entries to the file.  
- **dfs.replication**– we have just two DataNodes, so use 2.
- **dfs.namenode.name.dir** – directory used by Namenode to store its metadata file.
- **dfs.datanode.data.dir** – This directory is used by Datanode to store hdfs data blocks.  

```asciidoc
hduser@ztg-master:vi /home/hduser/hadoop/etc/hadoop/hdfs-site.xml
```

Insert within `<configuration>` tags:

```xml
<property>
<name>dfs.replication</name>
<value>2</value>
<description>Default block replication</description>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>file:/home/hduser/hadoop/yarn_data/hdfs/namenode</value>
<description>Directory for storing metadata by namenode</description>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>file:/home/hduser/hadoop/yarn_data/hdfs/datanode</value>
<description>Directory for storing blocks by datanode</description>
</property>
```

Update `yarn-site.xml` on Master and Slave Nodes for a Node to work as a Yarn Node.  Master and slave nodes should all be using the same value for the following properties,  and should be pointing to master node only.

```xml
<property>
 <name>yarn.nodemanager.aux-services</name>
 <value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
 <name>yarn.resourcemanager.scheduler.address</name>
 <value>ztg-master:8030</value>
</property> 
<property>
 <name>yarn.resourcemanager.address</name>
 <value>ztg-master:8032</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address</name>
  <value>ztg-master:8088</value>
</property>
<property>
  <name>yarn.resourcemanager.resource-tracker.address</name>
  <value>ztg-master:8031</value>
</property>
<property>
  <name>yarn.resourcemanager.admin.address</name>
  <value>ztg-master:8033</value>
</property>
```

Finally, update `slaves` file on ztg-master only:

```asciidoc
hduser@ztg-master:vi /home/hduser/hadoop/etc/hadoop/slaves
```

*is:*  
localhost  
*change to:*  
ztg-master  
ztg-slave1  
ztg-slave2  

### Start the Daemons

We are now ready to start the various daemons. But before that, we must format the `namenode` the first time. As `hduser` on `ztg-master`, do:  

```asciidoc
hduser@ztg-master:hdfs namenode -format
```

Then,  

```asciidoc
hduser@ztg-master:start-dfs.sh
hduser@ztg-master:start-yarn.sh
hduser@ztg-master:jps   (to see all daemons running on ztg-master)
hduser@ztg-master:~$ jps
25988 NodeManager
27017 Jps
25682 SecondaryNameNode
25333 NameNode
25493 DataNode
25855 ResourceManager
hduser@ztg-master:~$
```

Likewise, on ztg-slave1 and ztg-slave2, you should just see NodeManager and DataNode.

```asciidoc
hduser@ztg-slave1:~$ jps
19008 Jps
17970 NodeManager
17835 DataNode
hduser@ztg-slave1:~$
```

We have a running cluster. Let us test it.

From any laptop with a browser, go to `ztg-master`’s ip address and the following url:  

```asciidoc
http://x.y.z.156:8088/cluster/nodes
```

<center>
{{#figure "" "Apache Hadoop Cluster Monitoring" size="large" }}
<img src="/docs/connectors/assets/images/HadoopClusterMonitoring.png">
{{/figure}}
</center>

In the next section, we will set up an Edge Node for clients to develop map-reduce jobs
and access the Apache Hadoop cluster. However, let us run a quick example map-reduce job
from the master node to make sure the cluster is up and running.

### Run sample map-reduce job

Launch included example with 2 mappers, 4 samples each.

```asciidoc
hduser@ztg-master:cd /home/hduser/hadoop
hduser@ztg-master:~/hadoop$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4
```


View Map-Reduce job status at:

```asciidoc
http://x.y.z.156:8088/cluster/apps
```

**Map-Reduce Example Run Output**

```asciidoc
hduser@ztg-master:~$ cd hadoop
hduser@ztg-master:~/hadoop$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4
Number of Maps  = 2
Samples per Map = 4
Wrote input for Map #0
Wrote input for Map #1
Starting Job
15/06/25 23:04:25 INFO client.RMProxy: Connecting to ResourceManager at ztg-master/x.y.z.156:8032
15/06/25 23:04:25 INFO input.FileInputFormat: Total input paths to process : 2
15/06/25 23:04:25 INFO mapreduce.JobSubmitter: number of splits:2
15/06/25 23:04:26 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1435285545006_0004
15/06/25 23:04:26 INFO impl.YarnClientImpl: Submitted application application_1435285545006_0004
15/06/25 23:04:26 INFO mapreduce.Job: The url to track the job: http://ztg-master:8088/proxy/application_1435285545006_0004/
15/06/25 23:04:26 INFO mapreduce.Job: Running job: job_1435285545006_0004
15/06/25 23:04:34 INFO mapreduce.Job: Job job_1435285545006_0004 running in uber mode : false
15/06/25 23:04:34 INFO mapreduce.Job:  map 0% reduce 0%
15/06/25 23:04:44 INFO mapreduce.Job:  map 100% reduce 0%
15/06/25 23:04:51 INFO mapreduce.Job:  map 100% reduce 100%
15/06/25 23:04:51 INFO mapreduce.Job: Job job_1435285545006_0004 completed successfully
15/06/25 23:04:52 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=50
		FILE: Number of bytes written=318102
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=536
		HDFS: Number of bytes written=215
		HDFS: Number of read operations=11
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=3
	Job Counters 
		Launched map tasks=2
		Launched reduce tasks=1
		Data-local map tasks=2
		Total time spent by all maps in occupied slots (ms)=15533
		Total time spent by all reduces in occupied slots (ms)=5304
		Total time spent by all map tasks (ms)=15533
		Total time spent by all reduce tasks (ms)=5304
		Total vcore-seconds taken by all map tasks=15533
		Total vcore-seconds taken by all reduce tasks=5304
		Total megabyte-seconds taken by all map tasks=15905792
		Total megabyte-seconds taken by all reduce tasks=5431296
	Map-Reduce Framework
		Map input records=2
		Map output records=4
		Map output bytes=36
		Map output materialized bytes=56
		Input split bytes=300
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=56
		Reduce input records=4
		Reduce output records=0
		Spilled Records=8
		Shuffled Maps =2
		Failed Shuffles=0
		Merged Map outputs=2
		GC time elapsed (ms)=169
		CPU time spent (ms)=2880
		Physical memory (bytes) snapshot=756932608
		Virtual memory (bytes) snapshot=2515005440
		Total committed heap usage (bytes)=524812288
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=236
	File Output Format Counters 
		Bytes Written=97
Job Finished in 26.85 seconds
Estimated value of Pi is 3.50000000000000000000
hduser@ztg-master:~/hadoop$
```


