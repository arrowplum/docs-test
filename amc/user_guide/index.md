---
title: AMC User Guide
description: Select the Aerospike Monitor Console Community Edition or the Enterprise Edition, and learn the commands to start and stop the Aerospike Monitor Console.
---

The AMC User Guide has been split into 2 different parts to reflect the 2 different editions. AMC Enterprise Edition includes all the features of AMC Community Edition. 

- [AMC Community Edition](/docs/amc/user_guide/community)
- [AMC Enterprise Edition](/docs/amc/user_guide/enterprise)

### Common tasks
There are many common tasks that you may need to do. The following commands work for either edition of the AMC.


#### Starting/Stopping the AMC server - Linux without systemd
To start the AMC:

```
sudo /etc/init.d/amc start
```

To stop the AMC server:
```
sudo /etc/init.d/amc stop
```

To restart the AMC server:
```
sudo /etc/init.d/amc restart
```

To see whether or not the AMC server is up:
```
sudo /etc/init.d/amc status
```

Should you have any problem with the AMC, you can check the error logs in `/var/log/amc/error.log`.


#### Starting/Stopping the AMC server - Linux with systemd
To start the AMC:

```
sudo systemctl start amc

```

To stop the AMC server:
```
sudo systemctl stop amc

```

To restart the AMC server:
```
sudo systemctl restart amc

```

To see whether or not the AMC server is up:
```
sudo systemctl status amc

```

Should you have any problem with the AMC, you can check the error logs in `/var/log/amc/error.log`.


#### Starting/Stopping the AMC server - Mac OS X
To start the AMC:

```
sudo launchctl load -w /Library/LaunchAgents/com.aerospike.amc.plist
```

To stop the AMC server:
```
sudo launchctl unload /Library/LaunchAgents/com.aerospike.amc.plist
```

To see whether or not the AMC server is up:
```
sudo launchctl list | grep aerospike
```

Should you have any problem with the AMC, you can check the error logs in `/Library/amc/amc.txt`.

