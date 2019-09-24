<a name="install"></a>
### Install Aerospike
If you have already installed the server, you can skip directly to **[Start Aerospike](#run)**.

{{#steps}}
{{#steps-step 2 "Extract the contents of the package" markdown=true}}

To extract the contents of the package, run the following:

```bash
tar -xvf aerospike.tgz && cd aerospike-server
```

{{/steps-step}}
{{#steps-step 3 "Initialize Aerospike Server" markdown=true}}

Next, we will need to initialize a directory to host an aerospike instance:

```bash
./bin/aerospike init --help # to see the initialization options
./bin/aerospike init
```

---

When a directory is initialized the to host an aerospike instance, it will be populated with the following:
```
./bin/aerospike         - The management script to manage this instance.
./bin/asd               - The aerospike server daemon.
./etc/aerospike.conf    - The configuration file used by this instance.
./share/                - Contains read-only files, used by this instance.
./var/                  - Contains runtime files generated by `asd`, including logs and data files.
```

{{/steps-step}}
{{#steps-step 4 "Install Aerospike Tools" markdown=true}}

{{#note markdown=true}}
Superuser privileges (`sudo`) may be required to install Aerospike tools.
{{/note}}

To install the tools package, download one of the following packages:

```bash
# Red Hat Variants (RHEL6):
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/el6'
# Red Hat Variants (RHEL7):
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/el7'
# Debian 8:
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/debian8'
# Debian 9:
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/debian9'
# Debian 10:
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/debian10'
# Ubuntu 14.04 LTS:
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/ubuntu14'
# Ubuntu 16.04 LTS:
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/ubuntu16'
# Ubuntu 18.04 LTS:
wget -O aerospike-tools.tgz 'https://www.aerospike.com/download/tools/latest/artifact/ubuntu18'
```

Decompress the `.tgz` file and then install the tools package:

    tar -xvf aerospike-tools.tgz && cd aerospike-tools-*
    # Red Hat Variants (RHEL6):
    rpm -Uvh aerospike-tools-*.el6.x86_64.rpm
    # Red Hat Variants (RHEL7):
    rpm -Uvh aerospike-tools-*.el7.x86_64.rpm
    # Debian 8:
    dpkg -i aerospike-tools-*.debian8.x86_64.deb
    # Debian 9:
    dpkg -i aerospike-tools-*.debian9.x86_64.deb    
    # Debian 10:
    dpkg -i aerospike-tools-*.debian10.x86_64.deb    
    # Ubuntu 14.04 LTS:
    dpkg -i aerospike-tools-*.ubuntu14.04.x86_64.deb
    # Ubuntu 16.04 LTS:
    dpkg -i aerospike-tools-*.ubuntu16.04.x86_64.deb
    # Ubuntu 18.04 LTS:
    dpkg -i aerospike-tools-*.ubuntu18.04.x86_64.deb

This installation will add useful tools under `/opt/aerospike/bin` and link them from `/usr/bin`.

{{/steps-step}}
{{/steps}}