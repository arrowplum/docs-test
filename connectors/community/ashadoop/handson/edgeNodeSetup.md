---
title: Apache Hadoop Edge Node Setup
description: Step by step guide to setting up an Edge Node that accesses a Multi-Node Apache Hadoop Cluster
---

The procedure below explains setting up an edge node for clients to access the Hadoop Cluster for submitting jobs.

Let us set up a fourth instance as a client machine and submit jobs from the client machine to the hadoop cluster.

Follow these instructions for setting up a client machine in the cloud.

Create a minimum 4GB RAM, 2 core instance in the cloud and call it `ztg-client`.


### Install Java JDK

Login as root.

```asciidoc
sudo apt-get update
sudo apt-get install default-jdk
java -version
java version "1.7.0_79"
OpenJDK Runtime Environment (IcedTea 2.5.5) (7u79-2.5.5-0ubuntu0.14.04.2)
OpenJDK 64-Bit Server VM (build 24.79-b02, mixed mode)
```

### Create user group `hadoop`

```asciidoc
root@ztg-client:~#sudo addgroup hadoop
Adding group `hadoop' (GID 1000) ...
Done.
```

### Add user `hdclient` in group `hadoop`

```asciidoc
root@ztg-client:~# sudo adduser --ingroup hadoop hdclient
Adding user `hdclient' ...
Adding new user `hdclient' (1000) with group `hadoop' ...
Creating home directory `/home/hdclient' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hdclient
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n]Y
```

### Disable IPV6 on Ubuntu

Edit the following file:  
`root@ztg-master:~# sudo vi /etc/sysctl.conf`  

Add the following lines at the end:

```asciidoc
#disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
Merely editing the file does not disable ipv6.

```asciidoc
root@ztg-client:~# cat /proc/sys/net/ipv6/conf/all/disable_ipv6
0
```

Must execute`sysctl` to disable ipv6. 

```asciidoc
root@ztg-client:~# sudo sysctl -p
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

**Check:**  
```asciidoc
root@ztg-client:~# cat /proc/sys/net/ipv6/conf/all/disable_ipv6
1
```

### Enable ssh

**Note:** This is slightly different from when we set up the Apache Hadoop cluster. Now `hdclient` on `ztg-client` is getting access to `ztg-master`.
Add `hdclient` ssh keys - do this only on the `ztg-client` node. (Do not do on `ztg-master` or `ztg-slave` nodes.)  

```asciidoc
root@ztg-client:~# su hdclient
hdclient@ztg-client:/root$ cd ~
hdclient@ztg-client:~$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hdclient/.ssh/id_rsa): 
Created directory '/home/hdclient/.ssh'.
Your identification has been saved in /home/hdclient/.ssh/id_rsa.
Your public key has been saved in /home/hdclient/.ssh/id_rsa.pub.
The key fingerprint is:
a2:a3:b0:5f:7b:75:4e:83:c1:a7:27:b7:76:df:e2:31 hdclient@ztg-client
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|       .         |
|        o .      |
|      . S=       |
|     . .= *      |
|.   +  . B o  E  |
| o o o.   + . .+ |
|..o ..   . . oo..|
+-----------------+
hdclient@ztg-client:~$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
hdclient@ztg-client:~$ ssh hdclient@localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is 49:fe:cf:5e:e1:cd:20:7b:96:88:94:62:bd:5a:5a:01.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-45-generic x86_64)
hdclient@ztg-client:~$ exit
logout
Connection to localhost closed.
```

Now, back as `hdclient`: 
```asciidoc 
$ chmod 600 authorized_keys
```

We will copy this key to `ztg-master` to enable password less `ssh`.
But before we can do this, we need to do the following.

-  Add 'hdclient' as a user on ztg-master. Login as `root` on `ztg-master` and:  
`root@ztg-master:~# sudo adduser --ingroup hadoop hdclient`  

-  Take the public IP address of `ztg-master` and add it to `/etc/hosts` file as root on `ztg-client`. 
 
`root@ztg-client:~#sudo vi /etc/hosts`  

Insert after localhost entries, and, save and close:  

`x.y.z.156 ztg-master`

-  Now back as `hdclient` on `ztg-client`, do the following: 
 
`$ ssh-copy-id -i ~/.ssh/id_rsa.pub ztg-master`  (will ask for `hdclient` password on `ztg-master`)

-  As `hdclient` on `ztg-client`, make sure you can do a password less ssh using following command.  
`$ ssh ztg-master`

### Install Hadoop on Client Node

We need hadoop to compile the jars on `ztg-client` and submit the job but we will not run any hadoop daemons.

Copy hadoop tarball from `ztg-master` to `ztg-client`. 
In order to use `scp` from `ztg-master`, on `ztg-master`, as `root`, edit `/etc/hosts` and add `ztg-client`'s ip address. 

```asciidoc 
root@ztg-master:~#sudo vi /etc/hosts
```

Insert after localhost entries and save and close:

```asciidoc 
x.y.z.209 ztg-client
```

Then, `su` to `hduser` on `ztg-master` and do:  

```asciidoc 
root@ztg-master:~#su hduser
hduser@ztg-master:~$scp /home/hduser/hadoop-2.6.0.tar.gz hdclient@ztg-client:/home/hdclient
```
(Supply `hdclient`’s password on `ztg-client`).  

Now that we have the tarball on `ztg-client`, proceed to install hadoop.

```asciidoc 
hdclient@ztg-client:~$ tar zxvf hadoop-2.6.0.tar.gz 
hdclient@ztg-client:~$ mv hadoop-2.6.0 hadoop
hdclient@ztg-client:~$ ls
hadoop  hadoop-2.6.0.tar.gz
```

Update environment variables for `hdclient` in its `.bashrc`   

```asciidoc 
hdclient@ztg-client:~$ vi ~/.bashrc

#HADOOP VARIABLES START
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_INSTALL=/home/hdclient/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
#Dont need the exports below.
#export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
#export HADOOP_COMMON_HOME=$HADOOP_INSTALL
#export HADOOP_HDFS_HOME=$HADOOP_INSTALL
#export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib/native"
#HADOOP VARIABLES END

hdclient@ztg-client:~$ source ~/.bashrc
```

### Update Configuration files on Client Node

On `ztg-client`, edit `/home/hdclient/hadoop/etc/hadoop/core-site.xml` to be:

```xml
<configuration>
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://ztg-master:54310</value>
  <description>Use HDFS as file storage engine</description>
</property>
</configuration>
```

On `ztg-client`, edit `/home/hdclient/hadoop/etc/hadoop/mapred-site.xml` to be:

```xml
<configuration>
<property>
 <name>mapreduce.jobtracker.address</name>
 <value>ztg-master:54311</value>
 <description>The host and port that the MapReduce job tracker runs
  at. If “local”, then jobs are run in-process as a single map
  and reduce task.
</description>
</property>
</configuration>
```

On `ztg-master`, add new user `hdclient`, ie. same username as on `ztg-client` that you want to give remote access to:

```asciidoc
root@ztg-master:~#sudo  adduser  --ingroup  hadoop hdclient
```

On `ztg-master`, `su` to `hduser` and give write permission to our user group on `hadoop.tmp.dir` (here `/home/hduser//tmp`). Open `core-site.xml` to get the path for `hadoop.tmp.dir`.

```asciidoc
hduser@ztg-master:~$chmod 777 /home/hduser/tmp
```

### Running Map-Reduce from Client Node on Hadoop Cluster

The steps below show how to submit a map-reduce job as user `hdclient` logged into machine `ztg-client`.

```asciidoc
hdclient@ztg-client:~$ cd hadoop
hdclient@ztg-client:~/hadoop$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4
```

When successful, we expect the output to be something like this.  

```asciidoc
...
Job Finished in 1.511 seconds
Estimated value of Pi is 3.50000000000000000000
```

Now let us make the above command work.

- Create a directory structure in HDFS for the new user.

```asciidoc
hduser@ztg-master:~$ hadoop fs -mkdir /user/hdclient/
hduser@ztg-master:~$ hadoop fs -ls /user
Found 2 items
drwxr-xr-x   - hduser supergroup          0 2015-06-26 18:27 /user/hdclient
drwxr-xr-x   - hduser supergroup          0 2015-06-26 17:26 /user/hduser
```

**Note:** If you do not create the above directory and submit a map-reduce job as `hdclient` from `ztg-client`, you will get the following error:
```asciidoc
org.apache.hadoop.security.AccessControlException: Permission denied: user=hdclient, access=WRITE, inode="/user":hduser:supergroup:drwxr-xr-x
```

With just the above hdfs directory created, if you submit a map-reduce job as `hdclient` from `ztg-client`, you will get the following error:
```asciidoc
org.apache.hadoop.security.AccessControlException: Permission denied: user=hdclient, access=WRITE, inode="/user/hdclient":hduser:supergroup:drwxr-xr-x
```

So, need to do one more step:  

```asciidoc
hduser@ztg-master:~$ hadoop fs -chown -R hdclient:hadoop /user/hdclient
hduser@ztg-master:~$ hadoop fs -ls /user
Found 2 items
drwxr-xr-x   - hdclient hadoop              0 2015-06-26 18:27 /user/hdclient
drwxr-xr-x   - hduser   supergroup          0 2015-06-26 17:26 /user/hduser
```

Now the map-reduce job should run from `ztg-client` when logged in as `hdclient`.

In Hadoop 2.6.0, there is another hdfs directory:  

```asciidoc
hduser@ztg-master:~$ hadoop fs -ls /tmp/hadoop-yarn/staging/hduser
Found 1 items
drwx------   - hduser supergroup          0 2015-06-26 17:26 /tmp/hadoop-yarn/staging/hduser/.staging
```

We did not have to do anything with it so far and our map-reduce jobs launched from client machine work just fine.

Reference output when you start dfs and yarn.

```asciidoc
hduser@ztg-master:~$ start-dfs.sh
Starting namenodes on [ztg-master]
ztg-master: starting namenode, logging to /home/hduser/hadoop/logs/hadoop-hduser-namenode-ztg-master.out
ztg-slave2: starting datanode, logging to /home/hduser/hadoop/logs/hadoop-hduser-datanode-ztg-slave2.out
ztg-master: starting datanode, logging to /home/hduser/hadoop/logs/hadoop-hduser-datanode-ztg-master.out
ztg-slave1: starting datanode, logging to /home/hduser/hadoop/logs/hadoop-hduser-datanode-ztg-slave1.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /home/hduser/hadoop/logs/hadoop-hduser-secondarynamenode-ztg-master.out
hduser@ztg-master:~$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hduser/hadoop/logs/yarn-hduser-resourcemanager-ztg-master.out
ztg-slave2: starting nodemanager, logging to /home/hduser/hadoop/logs/yarn-hduser-nodemanager-ztg-slave2.out
ztg-slave1: starting nodemanager, logging to /home/hduser/hadoop/logs/yarn-hduser-nodemanager-ztg-slave1.out
ztg-master: starting nodemanager, logging to /home/hduser/hadoop/logs/yarn-hduser-nodemanager-ztg-master.out
hduser@ztg-master:~$
```

### NameNode/DataNode Webserver 
Open this link in your browser to monitor the cluster:
```asciidoc
http://x.y.z.156:50070/dfshealth.html#tab-overview
```

