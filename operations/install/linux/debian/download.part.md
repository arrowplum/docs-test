<a name="download"></a>
### Download Aerospike

The following are the steps to download Aerospike prior to installing. If you have already downloaded Aerospike, then you can skip to **[Install Aerospike](#install)**.

{{#steps}}
{{#steps-step 1 "Download Aerospike Server Community Edition" markdown=true}}

---
To download the latest stable release for Debian 10, run the following:

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/debian10'
```

For Debian 9, run the following:

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/debian9'
```

For Debian 8, run the following:

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/debian8'
```
---

For release notes and details on releases, visit the **[Download](/download)** page.

For the **Enterprise Edition** latest version, use the following command (debian10 in this example):
```bash
wget -O aerospike.tgz 'https://www.aerospike.com/enterprise/download/server/latest/artifact/debian10'
```

For the **Enterprise Edition** version 4.5.x or earlier, use the following command specifying the version (4.5.3.5 in this example) and Operating System distribution (debian10 in this example) along with the relevant credentials:
```bash
wget -O aerospike.tgz 'https://www.aerospike.com/enterprise/download/server/4.5.3.5/artifact/debian10' --user='<user>' --password='<password>'
```

{{/steps-step}}
{{/steps}}
