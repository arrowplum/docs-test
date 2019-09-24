<a name="nextsteps"></a>

---

{{#steps}}
{{#steps-step steps.verify "Verify" markdown=true}}
Use the Aerospike [__aql__](/docs/tools/aql/index.html) tool (installed at `/opt/aerospike/bin/aql` and linked in `/usr/bin/aql`) to insert and read a few sample records. Start by creating a new object with the key “Aerospike” in the “test” namespace that is part of the default configuration and adding three fields, “name”, “address” and “email”:

```bash
$ aql -h 127.0.0.1 -c "INSERT INTO test.demo (PK, name, address, email) VALUES ('Aerospike', 'Aerospike, Inc.', 'Mountain View, CA 94043', 'info@aerospike.com')"
INSERT INTO test.demo (PK, name, address, email) VALUES ('Aerospike', 'Aerospike, Inc.', 'Mountain View, CA 94043', 'info@aerospike.com')
OK, 1 record affected.

```

Retrieve the record and make sure it looks right:
```bash
aql -h 127.0.0.1 -c "select * from test.demo where PK='Aerospike'"
select * from test.demo where PK='Aerospike'
+-------------------+---------------------------+----------------------+
| name              | address                   | email                |
+-------------------+---------------------------+----------------------+
| "Aerospike, Inc." | "Mountain View, CA 94043" | "info@aerospike.com" |
+-------------------+---------------------------+----------------------+
1 row in set (0.002 secs)

OK

```

---
Now would be a good time to check out the Aerospike Management Console (AMC).
{{#if amc_installed}}
Make sure AMC is running and visit the following URL: `http://localhost:8081/` (If port 8081 was already in use then a different port would be mapped. Refer to the output when launching the VM for the correct port if that's the case.)
{{else}}
<div style="text-align: right;">
<a class="btn btn-primary" href="/docs/amc/install/">Install AMC</a>
</div>
{{/if}}
---

Delete the record and verify that it is deleted:
```bash
$ aql -h 127.0.0.1 -c "DELETE FROM test.demo where PK='Aerospike'"
DELETE FROM test.demo where PK='Aerospike'
OK, 1 record affected.
```

Note that the [__aql__](/docs/tools/aql/index.html) command line tool is meant to be used for basic validation only.  aql will create a new connection for every transaction and is not recommended as a production level client. Instead, use your application, integrated with Aerospike’s client libraries as described in the [DEVELOPMENT section](/develop/).
{{/steps-step}}

{{#steps-step steps.next_steps "Next Steps" markdown=true}}
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

