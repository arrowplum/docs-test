<a name="install"></a>
### Create an Aerospike Virtual Machine

{{#steps}}
{{#steps-step 2 "Create an Aerospike Directory" markdown=true}}

Vagrant requires each virtual machine to have its own working directory.
You can create the working directory anywhere, but for new users, we recommend
the following:

```bash
mkdir ~/aerospike-vm && cd ~/aerospike-vm
```

{{#note}}
To manage this virtual machine, all Vagrant commands must be run from within
this directory. The remainder of this document assumes you are in this directory.
{{/note}}

{{/steps-step}}
{{#steps-step 3 "Initialize the Aerospike Virtual Machine" markdown=true}}

To initialize the virtual machine, enter the following at the command prompt:

```bash
vagrant init aerospike/aerospike-ce
```

The image will be downloaded from
<a href="https://vagrantcloud.com/aerospike/boxes/aerospike-ce" target="_blank">Vagrant Cloud</a>.

{{/steps-step}}
{{/steps}}

