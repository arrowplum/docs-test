<a name="run"></a>
### Run Aerospike

Aerospike Server Daemon (**asd**) is managed by the virtual machine, so
starting and stopping the virtual machine will automatically start and stop **asd**.

{{#steps}}
{{#steps-step 4 "Start Aerospike" markdown=true}}

To start up the virtual machine and **asd**, run the following from the command prompt:

```bash
vagrant up
```

This will got through several stages to startup the virtual machine, operating
system and **asd**. The status of each stage will be displayed in the console.

- **Port Forwarding** – One stage will be to forward ports on the Windows
  machine to ports in the virtual machine. This means an application on the
  Windows machine will be able to connect and communicate with an application
  in the virtual machine. **asd** will be listening on port 3000 in the virtual
  machine. By default, port 3000 on the Windows machine will be forwarded to
  asd. However, there are times when port 3000 will not be available on the
  Windows machine, so a different port will be used. To know which port on the
  Windows machine is being used, check the start up messages.

{{/steps-step}}
{{#steps-step 5 "Verify Aerospike and AMC are Running" markdown=true}}

You can verify whether asd and amc have started successfully by checking the
status:

```bash
$ vagrant ssh -c "sudo service aerospike status"
asd (pid XXXX) is running...
Connection to 127.0.0.1 closed.
$ vagrant ssh -c "sudo service amc status"
Retrieving AMC status....
AMC is running.
Connection to 127.0.0.1 closed.
```


You can also search the server log at `/var/log/aerospike/aerospike.log` for the
successful startup message:

```bash
vagrant ssh -c "sudo grep -i cake /var/log/aerospike/aerospike.log"
```
You should see:
```
Jun 22 2014 03:35:33 GMT: INFO (as): (as.c::376) service ready: soon there will be cake!
Connection to 127.0.0.1 closed.
```

Make sure AMC is running and visit the following URL: `http://localhost:8081/` (If port 8081 was already in use then a different port would be mapped. Refer to the output when launching the VM for the correct port if that's the case.)		

If there are errors during start up, then consult the **[Troubleshooting]({{book.baseurl}}/operations/troubleshoot)** guide.

{{/steps-step}}
{{/steps}}
