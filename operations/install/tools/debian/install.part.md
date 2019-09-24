<a name="install"></a>
### Install Aerospike Tools
If you have already installed the Tools, you can skip directly to **[Validate Aerospike Tools Installation](#validate)**.

{{#steps}}
{{#steps-step 2 "Extract the contents of the package" markdown=true}}

To extract the contents of the package, run the following:

```bash
tar -xvf aerospike-tools.tgz
```

---

This should extract the contents in to a directory named similar to:

```bash
aerospike-tools-<version>-debian10/
# for Debian 9, replace "debian10" with debian9
# for Debian 8, replace "debian10" with debian8
```

Where `<version>` is the version of the package.

The contents of the directory should include:

- `asinstall` — A script to automate the installation process
- `aerospike-tools-<version>.debian10.*.x86_64.deb` — [Aerospike command-line tools and utilities](/docs/tools).

{{/steps-step}}
{{#steps-step 3 "Install Aerospike Tools" markdown=true}}

Please check the requirements [here](/docs/operations/install/tools/requirements) and install if any package is missing.

To install the tools packages, run the following:

```bash
cd aerospike-tools-<version>-debian10
# for Debian 9, replace "debian10" with debian9
# for Debian 8, replace "debian10" with debian8
sudo ./asinstall
```

{{#note}} Tools installation installs PEX binary for `asadm` tool which might be slow in start-up phase due to PEX packaging overhead. For faster startup times, please install the non-pex version of `asadm` from [source](https://github.com/aerospike/aerospike-admin). {{/note}}

Once the installation is complete, the tools should be ready to use.

---

{{/steps-step}}
{{/steps}}


