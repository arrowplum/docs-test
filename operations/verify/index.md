---
title: Verify Aerospike
description: This tutorial covers how to verify Aerospike is installed and running.
styles:
  - /assets/styles/ui/steps.css
---

### Overview
Use this tutorial to verify that Aerospike was successfully installed
and is running.

<a name="nextsteps"></a>

{{#steps}}
{{#steps-step 1 "Locate Essential Files" markdown=true}}

When you install Aerospike it places its essential files in the following
directories:
```bash
/etc/aerospike/                 - configuration files for Aerospike
/etc/aerospike/aerospike.conf   — default configuration for Aerospike
/etc/init.d/aerospike           — init script for Aerospike on non-systemd platforms
/etc/logrotate.d/aerospike      — logrotate configuration for Aerospike on non-systemd platforms
/opt/aerospike/bin/             — binaries including Aerospike server and tools
/opt/aerospike/doc/             — documents, including licenses
/opt/aerospike/sys/             — system data files, maintained by Aerospike
/opt/aerospike/usr/             — user data files
/var/log/aerospike/             — log files emitted by Aerospike
/usr/bin/asd                    — Aerospike Server daemon
```
See [Directory Structure](/docs/operations/manage/aerospike/directory_structure.html)
for more details.
{{/steps-step}}
{{#steps-step 2 "Verify Record Operations" markdown=true}}
Use the Aerospike [__aql__](/docs/tools/aql/index.html) tool (installed at `/opt/aerospike/bin/aql` and linked in `/usr/bin/aql`) to insert and read a few sample records. Start by creating a new object with the key “Aerospike” in the “test” namespace that is part of the default configuration and adding three fields, “name”, “address” and “email”:

{{#note}}
Mac OS X Vagrant Aerospike Server install also installs the Aerospike Tools in the Vagrant container. <BR>The Tools package, including aql, is accessible in the Vagrant container.
{{/note}} 

```bash
$ aql -h 127.0.0.1 -c "INSERT INTO test.demo (PK, name, address, email) VALUES ('Aerospike', 'Aerospike, Inc.', 'Mountain View, CA 94043', 'info@aerospike.com')"
INSERT INTO test.demo (PK, name, address, email) VALUES ('Aerospike', 'Aerospike, Inc.', 'Mountain View, CA 94043', 'info@aerospike.com')
OK, 1 record affected.
```

Retrieve the record and make sure it looks right:
```bash
$ aql -h 127.0.0.1 -c "select \* from test.demo where PK='Aerospike'"
select \* from test.demo where PK='Aerospike'
+-------------------+---------------------------+----------------------+
| name              | address                   | email                |
+-------------------+---------------------------+----------------------+
| "Aerospike, Inc." | "Mountain View, CA 94043" | "info@aerospike.com" |
+-------------------+---------------------------+----------------------+
1 row in set (0.002 secs)

OK

```

Delete the record and verify that it is deleted:
```bash
$ aql -h 127.0.0.1 -c "DELETE FROM test.demo where PK='Aerospike'"
DELETE FROM test.demo where PK='Aerospike'
OK, 1 record affected.
```

Note that the [__aql__](/docs/tools/aql/index.html) command line tool is meant to be used for basic validation only.  aql will create a new connection for every transaction and is not recommended as a production level client. Instead, use your application, integrated with Aerospike’s client libraries as described in the [Development section](/develop/).
{{/steps-step}}

{{#steps-step 3 "Next Steps" markdown=true}}
Now that you have a running server, let's put it to good use. Pick a client and start developing!
<div style="text-align: right;">
<a class="btn btn-primary" href="/develop">DEVELOP</a>
</div>

---

- [Benchmark](/docs/operations/install/common/benchmark.html) your installation.
- Install and connect to the [Aerospike Management Console](/docs/amc). AMC is preinstalled on the Aerospike Vagrant box and Amazon AMI image.
- [Configure](/docs/operations/configure) your node further and form a cluster by adding more nodes.
- Take a look at the complete set of [Tools & Utilities](/docs/tools).

{{/steps-step}}
{{/steps}}

