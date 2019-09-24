---
title: Install Problems
---

### /opt/aerospike directory conflict
If you get the following error during installation (versions up to 3.3.5):

```
Preparing packages...
    file /opt/aerospike conflicts between attempted installs of aerospike-server-community-3.3.5-1.el6.x86_64 and aerospike-tools-3.2.13-1.el6.x86_64
    file /opt/aerospike/data conflicts between attempted installs of aerospike-server-community-3.3.5-1.el6.x86_64 and aerospike-tools-3.2.13-1.el6.x86_64
    file /opt/aerospike/doc conflicts between attempted installs of aerospike-server-community-3.3.5-1.el6.x86_64 and aerospike-tools-3.2.13-1.el6.x86_64
```

Rerun the rpm command using the --force option:

```bash
sudo rpm -i aerospike-server-community-<version>.el6.x86_64.rpm --force
```
 
### Directory and file conflicts require manual uninstall

If you have installed some previous versions of Aerospike or Citrusleaf, there may be conflicts.

For a perfect installation, we recommend checking for, and removing, older versions of Aerospike and Citrusleaf.
This is not necessary for recent versions of Aerospike, but may be required for some older versions of Aerospike or Citrusleaf.

In Debian and Ubuntu,

```
sudo dpkg -l | fgrep aerospike
sudo dpkg -l | fgrep citrusleaf
sudo dkpg -r < package name >
```

In RedHat and Centos,

```
sudo rpm -qa | fgrep aerospike
sudo rpm -qa | fgrep citrusleaf
sudo rpm -e < package name >
```

### SeLinux prevents adding user/group aerospike/aerospike
If you see the following error during installation:

```
Preparing packages...
Installing /opt/aerospike
Adding group aerospike
Adding user aerospike
useradd: cannot create directory /opt/aerospike
error: %pre(aerospike-tools-3.2.13-1.el6.x86_64) scriptlet failed, exit status 12
error: aerospike-tools-3.2.13-1.el6.x86_64: install failed
Adding group aerospike
groupadd: group 'aerospike' already exists
Adding user aerospike
useradd: cannot create directory /opt/aerospike
error: %pre(aerospike-server-community-3.3.5-1.el6.x86_64) scriptlet failed, exit status 12
error: aerospike-server-community-3.3.5-1.el6.x86_64: install failed
```

Use `sestatus` to determine the configuration for SELinux. If the mode is `enforcing`, you will need to disable it.

Disable SELinux during the installation process where the user and group are created. You can re-enable it later.

Some distributions support disabling SELinux temporarily:

```bash
sudo setenforce 0
```

To re-enable:

```bash
sudo setenforce 1
```

To quickly check:
```bash
getenforce
```

To disable SELinux permanently, edit the file `/etc/selinux/config` and reboot. You may need to create this directory and file if it does not exist.

```bash
sudo vi /etc/selinux/config
```

The contents of the file should set SELINUX to `disabled` instead of `enforcing`:

```
SELINUX=disabled 
SELINUXTYPE=targeted
```

