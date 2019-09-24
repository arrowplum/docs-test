---
title: Aerospike Server Setup on Edge Node
description: Step by step guide to setting up the Aerospike server on the Edge Node
---
We will use `hduser` account on `ztg-client` to install the Aerospike server. 
Then we will ensure that `hdclient` can also access it.
First, as `root`, make sure `hduser` on `ztg-client` is a `sudoer`.

```asciidoc
root@ztg-client:~# sudo adduser hduser sudo
Adding user `hduser' to group `sudo' ...
Adding user hduser to group sudo
Done.
root@ztg-client:~#
```

Then, `su` to `hduser` and do the following:

```asciidoc
root@ztg-client:~# su hduser

hduser@ztg-client:/root$ cd ~
hduser@ztg-client:~$ wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/ubuntu12'

hduser@ztg-client:~$ ls
aerospike.tgz  hadoop  hadoop-2.6.0.tar.gz
hduser@ztg-client:~$ tar -zxvf aerospike.tgz

hduser@ztg-client:~$ cd aerospike-server-community-3.5.14-ubuntu12.04/
hduser@ztg-client:~/aerospike-...ubuntu12.04$ sudo ./asinstall
# will install the .rpm packages

hduser@ztg-client:~/aerospike...ubuntu12.04$ sudo service aerospike start && \
> sudo tail -f /var/log/aerospike/aerospike.log | grep cake
kernel.shmall too low, setting to 4G pages = 16TB
kernel.shmall = 4294967296
kernel.shmmax too low, setting to 1GB
kernel.shmmax = 1073741824
 * Start aerospike: asd                                                            [ OK ] 
Jun 27 2015 17:48:46 GMT: INFO (as): (as.c::450) service ready: soon there will be cake!
^C  (Cntrl-C)
```

Check if Aerospike server is up and running.

```asciidoc
hduser@ztg-client:~/aerospike...ubuntu12.04$aql
Aerospike Query
Copyright 2013 Aerospike. All rights reserved.

aql> show namespaces
+------------+
| namespaces |
+------------+
| "test"     |
| "bar"      |
+------------+
2 rows in set (0.000 secs)
OK

aql> exit

hduser@ztg-client:~/aerospike...ubuntu12.04$

Aerospike server is running.
```

Check if `hdclient` can access the Aerospike server:  

```asciidoc
hduser@ztg-client:~/aerospike-server-community-3.5.14-ubuntu12.04$ exit
exit
root@ztg-client:~# su hdclient
hdclient@ztg-client:/root$ cd ~
hdclient@ztg-client:~$ aql
Aerospike Query
Copyright 2013 Aerospike. All rights reserved.

aql> show namespaces
+------------+
| namespaces |
+------------+
| "test"     |
| "bar"      |
+------------+
2 rows in set (0.001 secs)
OK

aql> exit

hdclient@ztg-client:~$
```

`hdclient` is able to access Aerospike server running on `ztg-client`.

