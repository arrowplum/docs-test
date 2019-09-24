<a name="commands"></a>
### Halting and Restarting the Virtual Machine

{{#steps}}
{{#steps-step 3 "Stop Aerospike" markdown=true}}

To halt the virtual machine and stop **asd**, run:

```bash
vagrant halt
```

{{/steps-step}}
{{#steps-step 4 "Restart Aerospike and AMC" markdown=true}}

To bring the VM back up and restart Aerospike and AMC, run:

```bash
vagrant up
vagrant ssh
sudo service aerospike start
sudo service amc start
```

{{/steps-step}}
{{/steps}}

