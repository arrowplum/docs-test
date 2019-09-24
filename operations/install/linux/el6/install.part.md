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
aerospike-server-community-<version>-el6/
```

Where `<version>` is the version of the package.

The contents of the directory should include:

- `license.txt` — licenses for Aerospike and other software included in the package.
- `aerospike-tools-<version>.el6.x86_64.rpm` — [Aerospike command-line tools and utilities](/docs/tools).
- `aerospike-server-community-<version>.el6.x86_64.rpm` — the Aerospike server package.

For RHEL7 versions replace el6 with el7 in the above file names.

{{/steps-step}}
{{#steps-step 3 "Install Aerospike Server & Tools" markdown=true}}

{{#info}}
By default, many distributions have SELinux enabled, which interferes with first time installation. [Disable during installation](/docs/operations/troubleshoot/install/index.html#selinux-prevents-adding-usergroup-aerospikeaerospike).
{{/info}}

To install the server and the tools packages, run the following:

```bash
cd aerospike-server-community-<version>-el6
sudo ./asinstall
```

Once the installation is done, the server should be ready to be used with the default configuration file.
See [Directory Structure](/docs/operations/manage/aerospike/directory_structure.html)
for more details.

---

If there are errors during the installation, consult the **[Troubleshooting]({{book.baseurl}}/operations/troubleshoot/install)** guide.

{{/steps-step}}
{{/steps}}
