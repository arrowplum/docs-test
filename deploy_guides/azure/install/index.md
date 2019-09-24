---
title: Install on Azure
description: Get started quickly by installing Aerospike on Azure 
styles:
  - /assets/styles/ui/steps.css
scripts:
  - /assets/scripts/utils/target_blank.js
steps:
  next_steps: 10
  
---

{{#info}}Installing Aerospike on a Microsoft Azure VM instance is similar to the [Ubuntu installation](/docs/operations/install/linux/ubuntu). The following steps are a quick way to get started with an in-memory configuration.{{/info}}

{{#tbd}}For further details and deployment planning, please see our [Cloud Capacity Planning](/docs/operations/plan/cloud){{/tbd}}

{{#info}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/info}}

Aerospike provides a pre-installed image on the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/aerospike.aerospike-database-vm). You can choose to use this image to quickly get started, or create your own image.

## Marketplace Offering

{{#steps}}
{{#steps-step 1 "Azure Developers Console" markdown=true}}
We assume that you have already signed up for Azure. If not, [do this now](https://azure.microsoft.com).

* Go to your Azure Console at: https://portal.azure.com
* Go to Virtual machines, then click Add.
* Search for Aerospike, and select Aerospike Database Community Edition

---

* From the [Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/aerospike.aerospike-database-vm), click Get it Now
* Click Continue and it will take you into the Azure Portal to the Aerospike Database Community Edition page

{{/steps-step}}
{{#steps-step 2 "Create VM" markdown=true}}
On the Aerospike Database Community Edition page, click "Create"

Enter the following options:
1. Basics
  * Name: aerospike-server
  * VM disk type: SSD
  * User name: Give it a username
  * Authentication type: Aerospike recommends "SSH public key"
  * SSH Public key: The contents of your public key
  * Resource group: Choose to deploy into an existing group or create a new one.
  * Location: Choose a location geographically close to you. Azure locations also limit VM type availability. 
2. Size
  * Aerospike recommends GS or LS instances. DSv2 can also suffice.
3. Settings
  * Storage: Choose whether you'd like to use managed disks
  * Storage account: If you did not choose managed disks, choose the storage account you'd like to use.
  * Network: Leave as defaults
  * Extension: Add any extensions you'd like.
  * High availability: Aerospike recommends creating an availability zone with the maximum number of update domains and fault domains available for your selected zone.
  * Monitoring: Leave as defaults
4. Summary
  * Confirm the above values and click OK
5. Buy 
  * Click Purchase

{{/steps-step}}
{{/steps}}

## Manual Install

{{#steps}}
{{#steps-step 1 "Create a VM" markdown=true}}
Create an standard Linux VM in Azure. 

{{#info}}
Aerospike recommends Ubuntu 14 or 16 LTS versions.
{{/info}}

Your instance should come up and be listed in about a minute.

{{/steps-step}}
{{#steps-step 2 "Firewall Rules" markdown=true}}

Create a new Network Security Group (NSG) in the same Resource Group.

Add the following inbound rules:

  * Name: Aerospike Server:
  * Priority: 100
  * Source: Any
  * Service: Custom
  * Protocol and Ports: tcp:3000-3003
  * Action: Allow

  Add another firewall rule for the Aerospike Management Console

  * Name: Aerospike Management Console:
  * Priority: 200
  * Source: Any
  * Service: Custom
  * Procols and Ports: tcp:8081
  * Action: Allow

  And finally to secure:
  
  * Name: Default Deny
  * Priority: 900
  * Source: Any
  * Service: Custom
  * Protocol: Any
  * Port range: 1024-65535
  * Action: Deny

Attach this NSG to the network interface of your VM.

{{/steps-step}}
{{#steps-step 3 "Install Aerospike" markdown=true}}

With your VM selected, click on the "Connect" button. You will see a connection string to use like:
azureuser@&lt;Public_IP&gt;

Once connected, you can run the following:

```bash
sudo su -
apt-get update
wget -O aerospike-server.tgz https://www.aerospike.com/download/server/latest/artifact/ubuntu14
# Or ubuntu16
tar -zxvf aerospike-server.tgz
cd aerospike-server-community-*
./asinstall
```

{{/steps-step}}
{{#steps-step 4 "Install Aerospike Management Console (AMC)" markdown=true}}


```bash
wget -O aerospike-management-console.deb https://www.aerospike.com/download/amc/latest/artifact/ubuntu12
dpkg -i aerospike-management-console.deb
apt-get -f install
```

{{/steps-step}}
{{#steps-step 5 "Start Aerospike and AMC" markdown=true}}

```bash
sudo service aerospike start
sudo service amc start
```

{{/steps-step}}
{{#steps-step 6 "Connect to AMC" markdown=true}}

Connect to AMC by going to the following URL in your browser: `http://<Public_IP>:8081` where 'Public_IP' is the public IP address of the VM launched. In the AMC connect window, enter localhost for the hostname. 

{{/steps-step}}
{{#steps-step 7 "Configure Network and Storage" markdown=true}}


Read the instructions for getting a cluster up:
<div style="text-align: right;">
<a class="btn btn-primary" href="/docs/operations/configure/network">CONFIGURE NETWORK</a>
</div>

Check out how to setup storage to persist the data:
<div style="text-align: right;">
<a class="btn btn-primary" href="/docs/operations/configure/namespace/storage">CONFIGURE STORAGE</a>
</div>

{{/steps-step}}
{{/steps}}

{{md (path-resolve (path-dirname page.src) "../../../../operations/install/common/nextsteps.part.md") }}
