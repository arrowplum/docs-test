<a name="run"></a>
### Run Aerospike

Aerospike includes an init script for running the server, located in `./bin/aerospike`. This script will manage the Aerospike Server Daemon (**asd**) located in `./bin`.


{{#note markdown=true}}
The aerospike instance will store log files in `./var/log` and system data in `./share`.  If you change the user for the Aerospike process, then you will need to ensure the user has permissions for `./var/log` and `./share`.
{{/note}}

{{#steps}}
{{#steps-step 5 "Start Aerospike" markdown=true}}

You can start **asd** by running:

```bash
sudo ./bin/aerospike start
```

{{/steps-step}}
{{#steps-step 6 "Verify Aerospike is Running" markdown=true}}

You can verify whether **asd** had started successfully by checking the status:

```bash
./bin/aerospike status
# info: process running
```

---

You can also search the server log at `./var/log/aerospike.log` for the
successful startup message:

```bash
grep cake /var/log/aerospike/aerospike.log
```
You should see:
```
Jun 22 2014 03:35:33 GMT: INFO (as): (as.c::376) service ready: soon there will be cake!
```

If there are errors during start up, consult the **[Troubleshooting]({{book.baseurl}}/operations/troubleshoot/startup)** guide.

{{/steps-step}}
{{/steps}}

