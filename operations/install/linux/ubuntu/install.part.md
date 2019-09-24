<a name="install"></a>
### Install Aerospike
If you have already installed the server, you can skip directly to **[Start Aerospike](#run)**.

{{#steps}}
{{#steps-step 2 "Extract the contents of the package" markdown=true}}

To extract the contents of the package, run the following:

```bash
tar -xvf aerospike.tgz
```

---

This should extract the contents in to a directory named similarly to:

```bash
aerospike-server-community-<version>-ubuntu18/
# for ubuntu 16, replace "ubuntu18" with ubuntu16
# for ubuntu 14, replace "ubuntu18" with ubuntu14
```

Where `<version>` is the version of the package.

The contents of the directory should include:

- `license.txt` — licenses for Aerospike and other software included in the package.
- `aerospike-tools-<version>.ubuntu18.x86_64.deb` — [Aerospike command-line tools and utilities](/docs/tools).
- `aerospike-server-community-<version>.ubuntu18.x86_64.deb` — the Aerospike Server package.

For Ubuntu 16 replace ubuntu18 with ubuntu16 in the above file names.<BR>
For Ubuntu 14 replace ubuntu18 with ubuntu14 in the above file names.

{{/steps-step}}
{{#steps-step 3 "Install Aerospike Server & Tools" markdown=true}}

{{#note markdown=true}}
For Ubuntu, you must install with the command line. The Aerospike .deb files aren't signed properly to install through clicking.
{{/note}}

To install the server and tools packages, run the following:

```bash
cd aerospike-server-community-<version>-ubuntu18.04
# for ubuntu 16.04, replace "ubuntu18.04" with ubuntu16.04
# for ubuntu 14.04, replace "ubuntu18.04" with ubuntu14.04
sudo ./asinstall
```

Once the installation is done, the server should be ready to be used with the default configuration file.
See [Directory Structure](/docs/operations/manage/aerospike/directory_structure.html)
for more details.

---

If there are errors during the installation, consult the **[Troubleshooting]({{book.baseurl}}/operations/troubleshoot/install)** guide.

{{/steps-step}}
{{/steps}}

