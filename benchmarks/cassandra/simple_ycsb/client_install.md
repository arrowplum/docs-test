---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" with Manual Installation - YCSB
description: Installing YCSB for running an Aerospike and Cassandra benchmark
styles:
  - /assets/styles/ui/steps.css
---
While we will be using YCSB as the benchmark software, the client installation is made up of many different open source parts that have been put together to simplify testing and analysis.

{{#steps}}

{{!---
  ############################################################################
  #
  # STEP 1 - Planning
  #
  ############################################################################
---}}

{{#steps-step 1 "Planning" markdown=true}}
As you begin planning your own benchmark project, it is important for you to consider your test environment. There are several important factors to take into account. Naturally, changing the values in the benchmark will lead to different results.

# Client Server
| Component | Hardware Used in Blog Post | Recommendation | Notes |
| --------- | ----- | --- | --- |
| CPU       | Dual Intel(R) Xeon(R) CPU E5-2665 0 @ 2.40GHz (8 cores) | Dual 2.4GHz (8+ cores)| Generally speaking, it is best to use roughly the same caliber of CPU on the clients as on the database servers. |
| RAM       | 32 GB@1333 MHz | 16 GB RAM | The clients should have at least 16 GB of RAM to get the most out of them. Using less may result in unstable client behavior. |
| Network   | Intel Corporation 82599ES 10-Gigabit | 10 Gb | The networking here should match the networking on the server. |
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

## Make sure that your network queues are balanced

In our testing with the listed hardware, we found that all systems performed better with network queue tuning. This tuning can be done most easily via one of the 3 ways listed below. In our benchmark, we chose to use the oneshot method (option b. below).

If you choose to use one of the methods that use irqbalance, you may need to first install it, as follows:

```bash
ycsbhost$ sudo yum install -y irqbalance
```

This step can be skipped if your NIC only has one network queue. 

The three methods to do tuning are listed below.

  a. You can let the system do this tuning automatically by using irqbalance. Recent versions of Linux can do tuning automatically. Simply execute (as root):

```bash
ycsbhost$ sudo service irqbalance start
```
  b. During your initial testing, you can also turn on irqbalance to balance the queues, and then immediately exit. This must be done while applying a load to make sure that irqbalance knows how to balance, as follows. 

```bash
ycsbhost$ sudo service irqbalance stop
ycsbhost$ sudo irqbalance --oneshot
```

  c. You can also do the balancing manually. While doing so requires more manual work, this will produce optimal results. An excellent guide to this can be found at [Alex On Linux](http://www.alexonlinux.com/smp-affinity-and-proper-interrupt-handling-in-linux).

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 3 - Install Packages
  #
  ############################################################################
---}}

{{#steps-step 3 "Install Packages" markdown=true}}

Prior to installing either database, there are various packages you will want to install.

## Miscellaneous packages
There are a number of miscellaneous packages that it is best to install once, upfront. Many of these are required for YCSB. Enter the following yum commands:

```bash
ycsbhost$ sudo yum install -y ntp wget
ycsbhost$ sudo yum groupinstall -y 'Development Tools'
```
Now install the various development packages:

```bash
ycsbhost$ sudo yum install -y git gcc python-devel libffi-devel openssl-devel zlib-devel 
ycsbhost$ sudo yum install -y bzip2-devel ncurses-devel sqlite-devel freetype-devel
ycsbhost$ sudo yum install -y libpng-devel sigar-devel
```
Now install the graphical plotting tool:
```bash
ycsbhost$ sudo yum install -y python-matplotlib
```

After installing all the packages, start NTP to make sure that all the clocks on all servers (both client and server) are in agreement.

```bash
ycsbhost$ sudo /etc/init.d/ntpd start
```

## Install or upgrade Java if necessary

Many sources agree that the tested and recent versions of Cassandra work best on Java 8, build 40 or higher. While we believe that versions of OpenJDK and Oracle 7 might work, Cassandra's requirement of Java 8 greater than Build 40 requires us to  run our tests with a recent release of Oracle Java 8.

Validate that Java is installed and at a reasonable version:


```bash
ycsbhost$ java -version
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)
```

If you need to install or update Java to Oracle Version 8, there are many processes and help available on the Internet. The general process is to:
  1. Uninstall prior versions of Java through the `yum` system.
  2. Download the recent Java SE Development Kit RPM from [Oracle's download page]( http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html ).
  3. Install the RPM using the `rpm` command.

Other processes, such as compiling from source, downloading a binary build and decompressing into the `/opt` directory, or using non-Oracle repositories can also be used.

In the CentOS 6 installs we've seen, the following commands work, after having downloaded from Oracle's website:
```bash
ycsbhost$ sudo yum remove java-1.6.0-openjdk java-1.6.0-openjdk-headless java-1.6.0-openjdk-javadoc java-1.7.0-openjdk java-1.7.0-openjdk-headless java-1.7.0-openjdk-devel java-1.8.0-openjdk java-1.8.0-openjdk-headless java-1.8.0-openjdk-devel
ycsbhost$ sudo yum localinstall -y jdk-8u92-linux-x64.rpm
```

After you have updated, always be sure to validate using the `java -version` and `javac -version` that you are using the intended java version.

Make sure that your JAVA_HOME path is set by creating the appropriate file in `/etc/profile.d/java.sh` (your version may vary):

```bash
ycsbhost$ echo "export JAVA_HOME=/usr/java/jdk1.8.0_92" > /tmp/java.sh
ycsbhost$ sudo mv /tmp/java.sh /etc/profile.d
ycsbhost$ source /etc/profile.d/java.sh
```

## Install Maven 3.3.9
First install the Maven package

```bash
ycsbhost$ cd /opt
ycsbhost$ sudo wget http://www.interior-dsgn.com/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
```

Now untar the package:
```bash
ycsbhost$ sudo tar -xzvf apache-maven-3.3.9-bin.tar.gz
```
Finally, make sure that the maven binary is in your path. Do both of the following:

```bash
ycsbhost$ echo 'export PATH=/opt/apache-maven-3.3.9/bin:$PATH' > /tmp/maven.sh
ycsbhost$ sudo mv /tmp/maven.sh /etc/profile.d
ycsbhost$ source /etc/profile.d/maven.sh
```

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 4 - Install YCSB Client Software
  #
  ############################################################################
---}}
{{#steps-step 4 "Install YCSB Client Software" markdown=true}}

Now, install the Aerospike fork of the YCSB and the Aerospike scripts. You will be installing the following:

## YCSB and Aerospike Scripts

| Package         | Notes |
| ---             | --- |
| Aerospike fork of YCSB 0.7        | This is actually the main software used for the benchmarking. Get the Aerospike fork of YCSB available at [https://github.com/aerospike/YCSB](https://github.com/aerospike/YCSB). This should be installed in "/opt/YCSB" |
| Aerospike scripts | Aerospike has created a set of scripts that control the benchmarks. These are available at: [https://github.com/aerospike/aerospike-benchmarks](https://github.com/aerospike/aerospike-benchmarks).

Start by downloading the files from GitHub. Since you will be building here, if you are not root, make sure you change the ownership of the files to the user you are using. Use your current user as "USER".

```bash
ycsbhost$ cd /opt
ycsbhost$ sudo git clone https://github.com/aerospike/YCSB.git
ycsbhost$ sudo git clone https://github.com/aerospike/aerospike-benchmarks.git
ycsbhost$ sudo chown -R [USER]:[USER] /opt/YCSB
ycsbhost$ sudo chown -R [USER]:[USER] /opt/aerospike-benchmarks
```

To build the YCSB:

```bash
ycsbhost$ cd /opt/YCSB
ycsbhost$ mvn clean package
```

{{#note markdown=true}}
If you see errors, make sure of the following:

  * For some reason it is common to run into intermittent problems during the testing of some of the plugins. Sometime just re-running the mvn command fixes that. 
  * JAVA\_HOME is set to the appropriate directory (/usr/java/jdk1.8.0_92 for our install).
  * The maven binary (mvn) is in your PATH environment variable.
  * You have write permissions in the YCSB directory.
  * There is sometimes a problem in testing the Cassandra binding. While this does disrupt the testing of the binding and the build, the Aerospike and Cassandra bindings are created. If this happens to you, you should be able to continue.
{{/note}}

Now, copy the Aerospike benchmark workload files to the YCSB directory.

```bash
ycsbhost$ sudo cp /opt/aerospike-benchmarks/configs/simple_ycsb/YCSB/* /opt/YCSB/workloads/
```

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 5 - Install Python 2.7.9
  #
  ############################################################################
---}}
{{#steps-step 5 "Install Python 2.7.9" markdown=true}}

The versions of the graphing programs used requires the installation of Python 2.7.9, which is not standard with CentOS 6.7. 

To install Python 2.7.9 without interfering with the operation of 2.6, do the following:

## Get the main python 2.7.9 package.

```bash
ycsbhost$ cd /opt
ycsbhost$ sudo wget --no-check-certificate https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
```
## Untar and Build Python 2.7.9
```bash
ycsbhost$ sudo tar xf Python-2.7.9.tar.xz
ycsbhost$ cd Python-2.7.9
ycsbhost$ sudo ./configure --prefix=/usr/local
ycsbhost$ sudo make && sudo make altinstall

```
## Finalize the Python Installation

```bash
ycsbhost$ cd /usr/local/bin
ycsbhost$ sudo ln -s python2.7 python
```
## Check on the python version
The Python version should now be on 2.7.9
```bash
ycsbhost$ /usr/local/bin/python -V
Python 2.7.9
```

## Install additional Python Packages
Since you do not want to interfere with the 2.6 installation, you will need to create a script to install what you need for Python 2.7.9.

First, get the ez_setup.py tool:
```bash
cd /tmp
wget https://bootstrap.pypa.io/ez_setup.py
```


Create file `/tmp/python_install.sh` with the following contents:
```bash
#!/bin/bash
/usr/local/bin/python2.7 ez_setup.py
/usr/local/bin/easy_install-2.7 pip
/usr/local/bin/easy_install-2.7 requests
/usr/local/bin/pip install numpy
/usr/local/bin/pip install matplotlib
```

Now execute the script:

```bash
sudo bash /tmp/python_install.sh
```

You should now have Python 2.7.9 and all graphing packages installed.
{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 6 - Next Steps
  #
  ############################################################################
---}}
{{#steps-step 6 "Next Steps" markdown=true}}

Now install Aerospike onto your database hosts. Go to the [Installing Aerospike page](/docs/benchmarks/cassandra/simple_ycsb/aerospike_install.html).

{{/steps-step}}


{{/steps}}
