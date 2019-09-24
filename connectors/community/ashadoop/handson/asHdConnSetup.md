---
title: Aerospike Hadooop Connector Setup on Edge Node
description: Step by step guide to setting up the Aerospike Hadoop Connector on the Edge Node
---

Building the Aerospike Hadooop Connector requires `maven` and `git`.

### Install Maven

**Note:** Do not use `sudo apt-get install maven` ! It may not build the packages properly.
Instead, do the following. As `hduser` on `ztg-client`, in `/home/hduser`:  

```asciidoc
hduser@ztg-client:~$ wget http://mirror.cc.columbia.edu/pub/software/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
... ‘apache-maven-3.0.5-bin.tar.gz’ saved 
hduser@ztg-client:~$ sudo tar xzf apache-maven-3.0.5-bin.tar.gz -C /usr/local
[sudo] password for hduser: 
hduser@ztg-client:~$ cd /usr/local
hduser@ztg-client:/usr/local$ sudo ln -s apache-maven-3.0.5 maven
hduser@ztg-client:/usr/local$ ls
apache-maven-3.0.5  bin  etc  games  include  lib  man  maven  sbin  share  src
```
Next, set up Maven path system-wide. Add the two lines below in `maven.sh`:

```asciidoc
hduser@ztg-client:/usr/local$ sudo vi /etc/profile.d/maven.sh
export M2_HOME=/usr/local/maven
export PATH=${M2_HOME}/bin:${PATH}
```

Login again or do:

```asciidoc
hduser@ztg-client:/usr/local$ source /etc/profile
hduser@ztg-client:/usr/local$ cd ~
```

Check maven install.

```asciidoc
hduser@ztg-client:~$ mvn -version
Apache Maven 3.0.5 (r01de14724cdef164cd33c7c8c2fe155faf9602da; 2013-02-19 05:51:28-0800)
Maven home: /usr/local/maven
Java version: 1.7.0_79, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-7-openjdk-amd64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.13.0-45-generic", arch: "amd64", family: "unix"
hduser@ztg-client:~$
```

### Install Git
Install `git` to get Aerospike Hadoop Connector.
As `hduser` on `ztg-client`, in `/home/hduser`:

```asciidoc
hduser@ztg-client:~$ pwd
/home/hduser
hduser@ztg-client:~$ sudo apt-get install git
```

Check if `hdclient` can also access `git`. `su` to `hdclient` and,

```asciidoc
hdclient@ztg-client:~$ git --version
git version 1.9.1
```

### Install Aerospike Hadoop Connector

**Note:** We will install the Aerospike Hadoop Connector in `ztg-client` only since `hdclient` will compile its jars and send to the Apache Hadoop cluster for running jobs.
Start in `hdclient`’s home directory.
As `hdclient` on `ztg-client`, in `/home/hdclient`:

```asciidoc
hdclient@ztg-client:~$ cd ~
hdclient@ztg-client:~$
```

#### Git - steps to download Aerospike Hadoop Connector
Aerospike has provided the Aerospike Hadoop Connector via [a public repo on github](https://github.com/aerospike/aerospike-hadoop). 
You must first fork this repo into your github account and then clone it. 
We walk through this example using organization `acmegroup` and user `johndoe` on `github.com`.

(*Hint:* To search Aerospike’s public repositories on `github`, use the search string “user:aerospike” in the search entry box for `Repositories` at `github.com`.)


`johndoe` has forked this repo at `acmegroup/aerospike-hadoop`.  User `johndoe` has access to `acmegroup` organization.  In user `johndoe`’s account settings, we can add as many `id_rsa.pub` keys as we want. Users with those public keys can then clone repos in `acmegroup`. So, we added `hdclient`’s public key to `johndoe` and then cloned the fork as follows:

```asciidoc
hdclient@ztg-client:~$ more ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3Nza[ . . . . . . . ]u2qD hdclient@ztg-client
```
Copy and paste this key to `johndoe -> Account -> Settings -> Add SSH Keys` on github's web interface.

Now we can clone the fork and pull Aerospike’s Hadoop Connector code with examples into `hdclient@ztg-client`.

```asciidoc
hdclient@ztg-client:~$ git clone git@github.com:zintecgroup/aerospike-hadoop.git
Cloning into 'aerospike-hadoop'...
Warning: Permanently added the RSA host key for IP address '192.30.252.129' to the list of known hosts.
remote: Counting objects: 1291, done.
remote: Total 1291 (delta 0), reused 0 (delta 0), pack-reused 1291
Receiving objects: 100% (1291/1291), 206.54 KiB | 0 bytes/s, done.
Resolving deltas: 100% (356/356), done.
Checking connectivity... done.
hdclient@ztg-client:~$
```

We now have the `~/aerospike-hadoop` with the connector code in `hdclient` at `ztg-client`.

### Build using Maven

```asciidoc
hdclient@ztg-client:~$ cd ~/aerospike-hadoop
hdclient@ztg-client:~/aerospike-hadoop$ mvn clean package
[INFO] Reactor Summary:
[INFO] 
[INFO] aerospike-mapreduce ............................... SUCCESS [9.316s]
[INFO] sampledata ........................................ SUCCESS [8.747s]
[INFO] word_count_input .................................. SUCCESS [9.411s]
[INFO] aggregate_int_input ............................... SUCCESS [7.161s]
[INFO] word_count_output ................................. SUCCESS [7.741s]
[INFO] session_rollup .................................... SUCCESS [7.102s]
[INFO] generate_profiles ................................. SUCCESS [7.740s]
[INFO] external_join ..................................... SUCCESS [7.260s]
[INFO] spark_session_rollup .............................. SUCCESS [20.529s]
[INFO] aerospike-hadoop-examples ......................... SUCCESS [0.001s]
[INFO] aerospike-hadoop-parent ........................... SUCCESS [0.001s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1:25.184s
[INFO] Finished at: Mon Jun 29 04:51:09 PDT 2015
[INFO] Final Memory: 148M/502M
```

Alternatively you can use `gradle` that runs on top of `maven` to build individual packages.

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ ls
build.gradle  gradlew      mapreduce  sampledata       WORLDCUP_FILELIST
examples      gradlew.bat  pom.xml    settings.gradle
gradle        LICENSE      README.md  TODO.md

hdclient@ztg-client:~/aerospike-hadoop$ ./gradlew :mapreduce:jar

Downloading http://services.gradle.org/distributions/gradle-1.11-bin.zip
.....
Unzipping /home/hdclient/.gradle/wrapper/dists/gradle-1.11-bin/4h5v8877arc3jhuqbm3osbr7o7/gradle-1.11-bin.zip to /home/hdclient/.gradle/wrapper/dists/gradle-1.11-bin/4h5v8877arc3jhuqbm3osbr7o7
Set executable permissions for: /home/hdclient/.gradle/wrapper/dists/gradle-1.11-bin/4h5v8877arc3jhuqbm3osbr7o7/gradle-1.11/bin/gradle
Download http://repo1.maven.org/maven2/org/sonatype/sisu/inject/cglib/2.2.1-v20090111/cglib-2.2.1-v20090111.pom
Download http://repo1.maven.org/maven2/org/sonatype/sisu/inject/cglib/2.2.1-v20090111/cglib-2.2.1-v20090111.jar
:mapreduce:compileJava
:mapreduce:processResources UP-TO-DATE
:mapreduce:classes
:mapreduce:jar
BUILD SUCCESSFUL
Total time: 2 mins 51.503 secs
hdclient@ztg-client:~/aerospike-hadoop$
```

We are now ready to test the [examples](/docs/connectors/community/ashadoop/handson/examples.html) on our Apache Hadoop cluster.


