<a name="run"></a>
### Run Aerospike

Aerospike includes an init script for running the server, located in `/etc/init.d/aerospike`. This script will manage the Aerospike Server Daemon (**asd**) located in `/usr/bin`. Systemd platforms will use a systemd service instead of the init script.

An in-memory test namespace is configured by default. Please see the [configuration section](/docs/operations/configure) to add storage devices, configure a cluster, and tune your configuration to your hardware.

If you change the user for the Aerospike process, then you will need to ensure the user has read and write permissions for `/var/log/aerospike` and `/opt/aerospike`.

For **systemd platforms** such as Ubuntu 16.04, please see [systemd](/docs/operations/manage/aerospike/systemd).

{{#steps}}
{{#steps-step 4 "Start Aerospike" markdown=true}}

You can start **asd** by running one of the following:

```bash
sudo /etc/init.d/aerospike start

or

sudo systemctl start aerospike

or

sudo service aerospike start

```

{{/steps-step}}
{{#steps-step 5 "Verify Aerospike is Running" markdown=true}}

You can verify whether **asd** had started successfully by checking the status:

```bash

sudo /etc/init.d/aerospike status
# asd (pid 16816) is running...

or

sudo systemctl status aerospike

or

sudo service aerospike status


```

You can also search the server log at `/var/log/aerospike/aerospike.log` for the
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

