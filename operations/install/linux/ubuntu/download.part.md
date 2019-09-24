<a name="download"></a>
### Download Aerospike
If you have already downloaded the server, you can skip directly to **[Install Aerospike](#install)**.

{{#steps}}
{{#steps-step 1 "Download Aerospike Server Community Edition" markdown=true}}

---

To download the latest stable release, run the following:

```bash
wget -O aerospike.tgz 'https://www.aerospike.com/download/server/latest/artifact/ubuntu18'
# for ubuntu 16.04, replace ubuntu18 with ubuntu16
# for ubuntu 14.04, replace ubuntu18 with ubuntu14
```

---

For release notes and details on releases, visit the **[Download](/download)** page.

For the **Enterprise Edition** latest version, use the following command (ubuntu18 in this example):
```bash
wget -O aerospike.tgz 'https://www.aerospike.com/enterprise/download/server/latest/artifact/ubuntu18'
```

For the **Enterprise Edition** version 4.5.x or earlier, use the following command specifying the version (4.5.3.5 in this example) and Operating System distribution (ubuntu18 in this example) along with the relevant credentials:
```bash
wget -O aerospike.tgz 'https://www.aerospike.com/enterprise/download/server/4.5.3.5/artifact/ubuntu18' --user='<user>' --password='<password>'
```



{{/steps-step}}
{{/steps}}
