---
title: Install AMC on Other Linux distributions
description: Learn how to download and install Aerospike Management Console on other linux distributions.
styles:
  - /assets/styles/ui/steps.css
---

<style>
ol.steps {
  padding-top: 1em;
}
</style>

{{#todo markdown=true}}
  - check URLs for the download
{{/todo}}

Aerospike has provided zip files for easy installation of AMC on other Linux distributions.

## Installation of AMC version 4.0 and later
AMC 4.0 and above have no dependencies.

{{#steps}}

{{#steps-step 1 "Download the installation file" markdown=true}}
The location for AMC will depend on the edition.

| Edition    | Description |
| ---        | --- |
| [Community](/download/amc)  | This is the freely available edition that has all the basic functions. |
| [Enterprise](/enterprise/download/amc) | The Enterprise Edition includes more advanced management functions, including the ability to make dynamic changes to the configuration of the cluster. Customers with an Enterprise Edition subscription may download this. |
{{/steps-step}}

{{#steps-step 2 "Backup the AMC Configuration file" markdown=true}}

Create a backup of the current configuration file:
```
sudo cp /etc/amc/amc.conf /etc/amc/amc.conf.bac
```
{{/steps-step}}

{{#steps-step 3 "Install AMC" markdown=true}}

Once you have the file located on the server, you can install AMC by issuing the following commands:
```
sudo tar -xvf aerospike-amc-<version>.tar.gz -C /
```
{{/steps-step}}

{{#steps-step 4 "Restore the backup configuration" markdown=true}}

Restore the backup configuration file to `/etc/amc/amc.conf`:
```
sudo cp /etc/amc/amc.conf.bac /etc/amc/amc.conf
```
{{/steps-step}}

{{/steps}}


## Installation of AMC version 3.6 and earlier
Aerospike has created a special installation of AMC for installation on other forms of Linux. This package is only for the AMC Community Edition. If you have a need for the AMC Enterprise Edition on Linux, please contact Aerospike support.

### Requirements
The installation has the following requirements
- Linux
- Dual core processor
- 120 MB free hard disk space
- 2 GB RAM (4 GB recommended)
- Port 8081 should be free 

{{#steps}}

{{#steps-step 1 "Pre-install" markdown=true}}
You will need to make sure the following are installed:
- python (2.6+)
- gcc
- python-development

Methods for making sure these are installed vary by specific OS

{{/steps-step}}

{{#steps-step 2 "Download the installation file" markdown=true}}
The download for this package can be found on the [Aerospike download](/download/amc) page.
{{/steps-step}}

{{#steps-step 3 "Install AMC" markdown=true}}

Once you have the file located on the server, you can install AMC by issuing the following commands:
```
tar xvf aerospike-amc-<version>.tgz
cd aerospike-amc-<version>
sudo ./install
```
{{/steps-step}}

{{/steps}}

Note this is a script that you can use to uninstall it:
```bash
sudo /opt/amc/bin/uninstall
```

### Next Steps
See the [AMC User Guide](/docs/amc/user_guide) has instructions for how to start/stop the server and its usage. 



