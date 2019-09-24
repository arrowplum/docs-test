---
title: Upgrade to AMC 4.0
description: Learn how to upgrade to AMC 4.0
---

AMC 4.0 represents a major improvement over the previous version. The agent was rewritten in **Go**, which brought about the following performance and stability improvements:
   * Faster response to UI requests
   * Handling of large Aerospike clusters 
   * Less memory usage, and 
   * Strictly one and only one connection to a database node

AMC 4.0 also introduces important new features and improvements, such as: 
* Enabling Basic HTTP authentication between browser and agent
* Easier configuration and usage of AMC in non-root mode
* (Enterprise Edition-only, aka EE-only) Configuring clusters to always be monitored
* (EE-only) Supporting SSH-key based login for backup and restore
* (EE-only) Connecting to a TLS-enabled, secure Aerospike cluster
* (EE-only) Persisting notifications and showing them even after you close your browser
* (EE-only) Configuring email addresses to receive Aerospike cluster alerts

AMC 4.0 can only be used against Aerospike clusters on which every node runs Aerospike version 3.9 or later.
For the configuration details of the new features, please see our [Configure](/docs/amc/configure) section.

AMC 4.0 has no dependencies. Installation instructions for the different systems are provided below. 

**You MUST uninstall** the older version of AMC before installing AMC 4.0

### dpkg based systems like Debian, Ubuntu

uninstall the older amc version
```
sudo dpkg -P aerospike-amc-<edition>
```

install AMC
```
sudo dpkg -i aerospike-amc-<edition>-<version>.deb
```

start AMC
```
sudo service amc start
```

stop AMC
```
sudo service amc stop
```

### rpm based systems like CentOS, Red Hat
uninstall the older amc version
```
sudo rpm -e aerospike-amc-<edition>
```

install dependencies
```
sudo yum install -y initscripts
```

install AMC
```
sudo rpm -ivh aerospike-amc-<edition>-<version>.rpm
```

start AMC
```
sudo service amc start
```

stop AMC
```
sudo service amc stop
```

### zip installation

For generic Linux installations, you can use the generic zip package.

install AMC
```
sudo tar -xvf aerospike-amc-<edition>-<version>.tar.gz -C /
```

start AMC
```
sudo /etc/init.d/amc start
```

stop AMC
```
sudo /etc/init.d/amc stop
```

### MacOS

AMC is now managed via launchctl on MacOS. It will be restarted automatically if it has crashed.

install AMC
```
sudo tar -xvf aerospike-amc-<edition>-<version>-darwin.tar.gz -C /Library/ amc/ LaunchAgents/ Logs/
```

start AMC
```
sudo launchctl load  /Library/LaunchAgents/com.aerospike.amc.plist
```

stop AMC
```
sudo launchctl unload  /Library/LaunchAgents/com.aerospike.amc.plist
```

uninstall AMC
```
sudo /Library/amc/uninstall.sh
```
