---
title: Planning a Deployment
description: |
  Follow these easy steps to acheive a successful deployment of the Aerospike database
styles:
  - /assets/styles/ui/steps.css

---

In order to use Aerospike Server successfully in either development or production environments, please review the system requirements.

### Recommended Minimum Requirements

The Aerospike server runs on most major 64-bit Linux distributions – CentOS (Red Hat), Ubuntu, and Debian. It is distributed in standard RPM and DEB packages appropriate to the installation environment. The Aerospike server is routinely tested on the following distributions:

- CentOS 6 (RHEL 6) and CentOS 7 (RHEL 7)
- Debian 8, 9 and 10
- Ubuntu 14.04, 16.04 and 18.04

The recommended minimum system requirements for Aerospike to function properly are:

- 64-bit Linux 
- 2 GB of RAM

Although 2 GB of RAM is the recommended minimum (OK for development), you will want to ensure you have sufficient amount of RAM for the amount of data you intend to store.

{{#todo}}Add link for Cloud / Amazon instructions.{{/todo}}

If you are planning for a production deployment, we recommend you review the deployment planning guide below.

### Deployment Planning

The recommended minimum system requirements is sufficient for a development environment, but not for a production environment. For production deployments, please review the following steps to ensure your production environment is successfully deployed.

{{#steps}}

{{#steps-step 1 "Estimating the Size of your Data" markdown=true}}
Before you begin, you need to answer a couple of fundamental questions about what you want to store.  In order to size the hardware correctly, you must make a rough estimate of the data.

- How many records will you need?  (for example, how many user profiles are you expecting)
- How much data do you plan to store in each record?

{{#note}}Aerospike allows you to add more storage later, but since hardware is relatively inexpensive, it's a good idea to start with a rough estimate of what you'll need.<br>
There is a current limitation of individual devices and files in Aerospike not be configured larger than 2 TiB.{{/note}}
{{/steps-step}}

{{#steps-step 2 "Determining the Type of Storage" markdown=true}}
Aerospike allows you to store your data:

- In memory (without persistence)
- In memory (with persistence; data is backed up on disk)
- On flash (SSD) storage (with indexes in memory)

{{#warn}}Once you set the storage method, you may not be able to change it later without losing data.{{/warn}}
{{/steps-step}}

{{#steps-step 3 "Determining the Number of Namespaces" markdown=true}}

The Aerospike Database stores data in namespaces (roughly equivalent to a database in an RDBMS).  Each namespace is configured according to its storage requirements.  Many Aerospike installations use a single namespace with all data being handled in the same storage media.  But for example, if you want to have one group of data in memory and another group of data on SSD, you will need to configure two namespaces. 

{{#warn}}Adding a new namespace requires a full cluster restart.{{/warn}}
{{/steps-step}}

{{#steps-step 4 "Capacity Planning" markdown=true}}

For assistance on how to properly size your deployment, including memory and storage requirements, please use the **Capacity Planning Guide**. You may skip this step if you are unsure of your capacity requirements or just want to try out the database.

  {{steps-action "Capacity Planning Guide" "operations/plan/capacity"}}

{{/steps-step}}
{{#steps-step 5 "Server Hardware" markdown=true}}

For assistance on getting the proper hardware for your deployment, please use the **Server Hardware Requirements**

  {{steps-action "Server Hardware Requirements" "operations/plan/hardware"}}

{{/steps-step}}
{{#steps-step 6 "Flash Storage" markdown=true}}

For assistance on acquiring the right Flash storage for your deployment, please use the **Flash Storage Guide**

  {{steps-action "Flash Storage Guide" "operations/plan/ssd"}}

{{/steps-step}}
{{#steps-step 7 "Network" markdown=true}}

For assistance on acquiring the right Flash storage for your deployment, please use the **Network Guide**

  {{steps-action "Network Guide" "operations/plan/network"}}

{{/steps-step}}
{{/steps}}

 

{{#note markdown=true}}The Aerospike Sales Engineering team is available to help you size your hardware and determine that your proposed hardware configuration is suitable for your project. [Contact us](/contact/) for help with sizing your hardware.

**If you are installing Aerospike for evaluation:**

- For evaluation purposes, you may find it simpler to run a test database using DRAM.  Depending on your SSD, the setup process can take some time.  In addition, we don't recommend that you overprovision SSDs that are virtual drives on a laptop.
- Be careful of using Amazon to evaluate performance. Amazon is a shared platform and network latency can vary depending on traffic to other sites that are sharing the server.  If you are looking to use Amazon only for evaluating, you should know that Amazon is not a suitable choice for those that will deploy on bare metal.
- If using a VM for testing, note that a VM adds latency to any storage, especially SSDs.  As a result, you won't get maximal performance.  Production servers typically don't run inside a VM, so this is usually only an issue when testing/evaluating.{{/note}}



