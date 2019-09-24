---
title: Install on Ubuntu and Debian
description: Install Aerospike Management Console on Ubuntu and Debian
styles:
  - /assets/styles/ui/steps.css
---

<style>
ol.steps {
  padding-top: 1em;
}
</style>


Aerospike has provided deb files for easy installation of AMC on Ubuntu and Debian.

## Installation of AMC version 4.0 and later
AMC 4.0 and above have no dependencies.

{{#steps}}

{{#steps-step 1 "Download the deb file" markdown=true}}
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

{{#steps-step 3 "Install the deb file" markdown=true}}

Once you have the file located on the server, you can install AMC by issuing the standard command:
```
sudo dpkg -i aerospike-amc-<version>.deb
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
### Requirements
The installation of the rpm requires the following:
- Debian 6+/Ubuntu 12.04
- Dual core processor
- 120 MB free hard disk space
- 2 GB RAM (4 GB recommended)
- Port 8081 should be free 

{{#steps}}

{{#steps-step 1 "Pre-install" markdown=true}}
You will need to make sure the following are installed:
- python (2.6+)
- gcc
- python-dev

If you are missing any of the above, you can install them by running the command:
```bash
sudo apt-get install <package>
```
Where &lt;package&gt; is the missing package.

**Enterprise Edition**

If you are installing AMC Enterprise Edition and wish to use the advanced management features, there are some additional pre-requisites:
- You should install ansible on the AMC host. Follow the directions at: [http://docs.ansible.com/ansible/intro_installation.html](http://docs.ansible.com/ansible/intro_installation.html).
- If you would like to run AMC as a non-root user, you will have to have give the user sudo permissions. Add the following line in the sudo file using `visudo`: 
```bash
<user> ALL=(ALL:ALL) NOPASSWD: ALL 
```
Where <user> is the sudo user that will be running AMC.
- Install `pip`. You can find the installation instructions at [https://pip.pypa.io/en/latest/installing.html](https://pip.pypa.io/en/latest/installing.html)
- You should install the following using the `pip` command:
```bash
sudo pip install markupsafe
sudo pip install paramiko
sudo pip install ecdsa
sudo pip install pycrypto
sudo pip install bcrypt
```




{{/steps-step}}

{{#steps-step 2 "Download the deb file" markdown=true}}
The location for AMC will depend on the edition.

| Edition    | Description |
| ---        | --- |
| [Community](/download/amc)  | This is the freely available edition that has all the basic functions. |
| [Enterprise](/enterprise/download/amc) | The Enterprise Edition includes more advanced management functions, including the ability to make dynamic changes to the configuration of the cluster. Customers with an Enterprise Edition subscription may download this. |

{{/steps-step}}

{{#steps-step 3 "Install the deb file" markdown=true}}

Once you have the file located on the server, you can install AMC by issuing the standard command:
```
sudo dpkg -i aerospike-amc-<version>.deb
```
{{/steps-step}}

{{/steps}}

### Next Steps
See the [AMC User Guide](/docs/amc/user_guide) has instructions for how to start/stop the server and its usage.



