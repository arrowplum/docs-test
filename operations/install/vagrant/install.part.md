<a name="install"></a>
### Install Vagrant

If you have already installed and used Vagrant, you may continue to 
{{#if is_osx}}**[Install on OS X](/docs/operations/install/vagrant/mac)**{{/if}}{{#unless is_osx}}**[Install on Windows](/docs/operations/install/vagrant/win)**{{/unless}}
or view the <a href="#commands">vagrant commands</a>.

{{#steps}}
{{#steps-step 1 "Install Vagrant" markdown=true}}

If you do not already have Vagrant installed, please
<a href="http://www.vagrantup.com/downloads.html" target="_blank">download and install Vagrant</a>.

{{/steps-step}}
{{/steps}}

{{#info markdown=true}}
Vagrant works with a virtualization system such as Oracle VirtualBox or VMWare Fusion.
{{/info}}

{{#steps}}
{{#steps-step 2 "Install VirtualBox" markdown=true}}
For new users, we recommend using VirtualBox, a free virtualization product
available from Oracle. Vagrant is well integrated with VirtualBox, so you
can expect it to work with little effort.

<a href="https://www.virtualbox.org/wiki/Downloads" target="_blank">Download and
install VirtualBox</a>.

{{#warn}}
Remember to initialize the virtual machine, by entering the following at the command prompt:

```
vagrant init aerospike/aerospike-ce
```
<BR>
The image will be downloaded from
<a href="https://vagrantcloud.com/aerospike/boxes/aerospike-ce" target="_blank">Vagrant Cloud</a>.
{{/warn}}

{{/steps-step}}

{{#note markdown=true}}
Or alternatively
{{/note}}

{{#steps-step 2 "Install the Vagrant VMWare Provider" markdown=true}}
A Vagrant VMWare provider is also available, but requires a commercial license.

Purchase and install the <a href="http://www.vagrantup.com/vmware" target="_blank">Vagrant
VMWare provider</a>.

{{/steps-step}}
{{/steps}}

