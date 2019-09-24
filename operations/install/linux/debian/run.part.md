<a name="run"></a>
### Run

Aerospike includes an init script for running the server, located in `/etc/init.d/aerospike`. These scripts will manage the Aerospike Server Daemon (**asd**) located in `/usr/bin`.

An in-memory test namespace is configured by default. Please see the [configuration section](/docs/operations/configure) to add storage devices, configure a cluster, and tune your configuration to your hardware.

If you change the user for the Aerospike process, then you will need to ensure the user has read and write permissions for `/var/log/aerospike` and `/opt/aerospike`.

For **systemd platforms** such as Debian 8, please see [systemd](/docs/operations/manage/aerospike/systemd).

{{#steps}}
{{#steps-step 4 "Start Aerospike" markdown=true}}

You can start **asd** by running:

```bash
sudo /etc/init.d/aerospike start
```

```bash
# Debian 8 or 9
sudo systemctl start aerospike
```

{{/steps-step}}
{{#steps-step 5 "Verify Aerospike is Running" markdown=true}}

You can verify whether **asd** had started successfully by checking the status:

```bash
sudo /etc/init.d/aerospike status
# asd (pid 16816) is running...
```

```bash
# Debian 8 or 9
sudo systemctl status aerospike
```

You can also verify the startup of the process by checking the contents of the log in `/var/log/aerospike/aerospike.log`.

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

