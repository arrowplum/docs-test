---
title: Install on Google Compute Engine
description: Get started quickly by installing Aerospike on a Google Compute Engine VM instance.
styles:
  - /assets/styles/ui/steps.css
scripts:
  - /assets/scripts/utils/target_blank.js
steps:
  next_steps: 9
  
---

{{#info}}Installing Aerospike on a Google Compute Engine VM instance is similar to the [Ubuntu installation](/docs/operations/install/linux/ubuntu). The following steps are a quick way to get started with an in-memory configuration.{{/info}}

{{#tbd}}For further details and deployment planning, please see [Google Compute Engine Capacity Planning](/docs/operations/gcp/instance){{/tbd}}

{{#info}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/info}}

{{#steps}}
{{#steps-step 1 "Google Developers Console" markdown=true}}
We assume that you have already signed up for the Google Cloud Platform. If not, [do this next](https://cloud.google.com/).

* Go to your Google Developers Console at: https://console.developers.google.com/
* Either select a **Project** where you want to install Aerospike, or create a new project by clicking on "**Create Project**".

{{#note}}
Google Compute Engine has a **quota** limit of 8 CPU cores for the **Free Trial**. We suggest that you make a request to increase this quota for your Project.
{{/note}}

{{/steps-step}}

{{#steps-step 2 "Create VM" markdown=true}}
On the Project Dashboard, click on "**Create Instance**".

Use the following options:
* Name: aerospike-server
* Zone: us-central-1f (Choose a zone which is geographically close to your location)
* Machine Type: n1-standard-2
* Boot Source: New disk from image
* Image: Ubuntu 16.04 LTS
* Boot disk: Standard persistent disk

Use defaults for other options and click on "**Create**".

Your instance should come up and be listed in about a minute.

{{/steps-step}}
{{#steps-step 3 "Firewall Rules" markdown=true}}

* On the instance page, click on the "**default**" link under the "Network" section. Here you can add firewall rules for the Project.
* Under **Firewall Rules**, click on "**Create New**"
* Use the following values to create a rule for Aerospike server
  * Name: Aerospike Server:
  * Source IP Ranges: 0.0.0.0/0
  * Procols and Ports: tcp:3000-3004
* Add another firewall rule for the Aerospike Management Console
  * Name: Aerospike Management Console:
  * Source IP Ranges: 0.0.0.0/0
  * Procols and Ports: tcp:8081
  
{{/steps-step}}
{{#steps-step 4 "Install Aerospike" markdown=true}}

Click on the "**SSH**" button. You may connect via the browser or the "gcloud" command line tool.

```bash
sudo su -
apt-get update
wget -O aerospike-server.tgz https://www.aerospike.com/download/server/latest/artifact/ubuntu16
tar -zxvf aerospike-server.tgz
cd aerospike-server-community-*
./asinstall
```

{{/steps-step}}
{{#steps-step 5 "Install Aerospike Management Console (AMC)" markdown=true}}

```bash
wget -O aerospike-management-console.deb https://www.aerospike.com/download/amc/latest/artifact/ubuntu12
dpkg -i aerospike-management-console.deb
apt-get -f install
```

{{/steps-step}}
{{#steps-step 6 "Start Aerospike and AMC" markdown=true}}
```bash
service aerospike start
service amc start
```

{{/steps-step}}
{{#steps-step 7 "Connect to AMC" markdown=true}}

Connect to AMC by going to the following URL in your browser: `http://<Public_IP>:8081` where 'Public_IP' is the public IP address of the VM launched. In the AMC connect window, enter localhost for the hostname. 

{{/steps-step}}
{{#steps-step 8 "Configure Network and Storage" markdown=true}}

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

