---
title: Upgrade AMC from Community to Enterprise
description: Learn how to upgrade an AMC Community Version to AMC Enterprise version. 
---
The Enterprise edition of AMC includes additional features like alerts, management features of cluster. </br>
You can use the following steps to upgrade AMC Community edition to the AMC Enterprise edition.</br></br>
{{#steps}}

{{#steps-step 1 "Backup the AMC Configuration file" markdown=true}}
In AMC 4.x create a backup of the current configuration file:
```
sudo cp /etc/amc/amc.conf /etc/amc/amc.conf.bac
```
{{/steps-step}}

{{#steps-step 2 "Uninstall Community AMC Package" markdown=true}}
For Redhat/Centos distributions:</br>
```
sudo rpm -e aerospike-amc-community
```
For Debian/Ubuntu distributions:</br>
```
sudo dpkg -r aerospike-amc-community
```
{{/steps-step}}

{{#steps-step 3 "Install AMC Enterprise Packages" markdown=true}}
You will get Enterprise AMC package with Enterprise database package. For more information on downloading and installing, see the [Installation Documentation](/docs/amc/install).</br>
{{/steps-step}}

{{#steps-step 4 "Restore the backup configuration" markdown=true}}
In AMC 4.x restore the backup configuration file to `/etc/amc/amc.conf`:
```
sudo cp /etc/amc/amc.conf.bac /etc/amc/amc.conf
```
{{/steps-step}}

{{#steps-step 5 "Start AMC service" markdown=true}}
Run the following command:</br>
```
sudo /etc/init.d/amc start
```
For more information on starting the service, see the [AMC user guide documentation](/docs/amc/user_guide).
{{/steps-step}}

{{/steps}}
