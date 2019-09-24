<a name="download"></a>
### Download Aerospike
If you have already downloaded the server, you can skip directly to **[Install Aerospike](#install)**.

{{#steps}}
{{#steps-step 1 "Download Aerospike Server Community Edition" markdown=true}}

---
To download the latest stable release for el6, run the following:

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/el6'
```

Or if your distribution does not include a copy of wget, you can use curl:
```
curl -L 'https://www.aerospike.com/download/server/latest/artifact/el6' > aerospike.tgz
```

For el7, run the following:

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/el7'
```

---

For release notes and details on releases, visit the **[Download](/download)** page.

For the **Enterprise Edition** latest version, use the following command (el7 in this example):
```bash
wget -O aerospike.tgz 'https://www.aerospike.com/enterprise/download/server/latest/artifact/el7'
```

For the **Enterprise Edition** version 4.5.x or earlier, use the following command specifying the version (4.5.3.5 in this example) and Operating System distribution (el7 in this example) along with the relevant credentials:
```bash
wget -O aerospike.tgz 'https://www.aerospike.com/enterprise/download/server/4.5.3.5/artifact/el7' --user='<user>' --password='<password>'
```

{{#info}}
The `el6` package has been validated to work with modern Red Hat-based
distributions, including CentOS, Fedora, Oracle Linux, Amazon Linux, and Red Hat
Enterprise Linux (RHEL) 7.
{{/info}}

{{/steps-step}}
{{/steps}}
