<a name="run"></a>
### Run

Aerospike includes an init script for running the server, located in `/etc/init.d/aerospike`. This script will manage the Aerospike Server Daemon (**asd**) located in `/usr/bin`.

An in-memory test namespace is configured by default. Please see the [configuration section](/docs/operations/configure) to add storage devices, configure a cluster, and tune your configuration to your hardware.

If you change the user for the Aerospike process, then you will need to ensure the user has read and write permissions for `/var/log/aerospike` and `/opt/aerospike`.

For **systemd platforms** such as CentOS 7, please see [systemd](/docs/operations/manage/aerospike/systemd).

{{#steps}}
{{#steps-step 4 "Start Aerospike" markdown=true}}

You can start **asd** by running:

```bash
sudo service aerospike start
```

{{#note markdown=true}}
For Fedora and some RedHat derivatives, you may run into the [network interface startup problem](/docs/operations/troubleshoot/startup)
{{/note}}

{{/steps-step}}
{{#steps-step 5 "Verify Aerospike is Running" markdown=true}}

You can verify whether **asd** had started successfully by checking the status:

```bash
sudo service aerospike status
# asd (pid 16816) is running...
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

