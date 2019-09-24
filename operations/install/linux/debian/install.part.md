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
aerospike-server-community-<version>-debian10/
```

Where `<version>` is the version of the package.

The contents of the directory should include:

- `license.txt` — licenses for Aerospike and other software included in the package.
- `aerospike-tools-<version>.debian10.x86_64.deb` — [Aerospike command-line tools and utilities](/docs/tools).
- `aerospike-server-community-<version>.debian10.x86_64.deb` — the Aerospike server package.

For Debian 9 replace debian10 with debian9 in the above file names.<br>
For Debian 8 replace debian10 with debian8 in the above file names.


{{/steps-step}}
{{#steps-step 3 "Install Aerospike Server & Tools" markdown=true}}

To install the server and the tools packages on Debian 10, run the following:

```bash
cd aerospike-server-community-<version>-debian10
sudo ./asinstall
```

On Debian 9

```bash
cd aerospike-server-community-<version>-debian9
sudo ./asinstall
```

On Debian 8

```bash
cd aerospike-server-community-<version>-debian8
sudo ./asinstall
```

Once the installation is done, the server should be ready to be used with the default configuration file.
See [Directory Structure](/docs/operations/manage/aerospike/directory_structure.html)
for more details.

---

If there are errors during the installation, consult the **[Troubleshooting]({{book.baseurl}}/operations/troubleshoot/install)** guide.

{{/steps-step}}
{{/steps}}
