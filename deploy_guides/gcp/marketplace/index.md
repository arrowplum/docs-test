---
title: Marketplace Quick Install
description: |
  Here are the instructions on how to quickly spin up a cluster in GCP
scripts:
  - /assets/scripts/utils/target_blank.js

---

Aerospike provides Deployment Manager templates to quickly provision an entire Aerospike cluster with recommended settings.

The quickest and simplest way to get Aerospike running on GCP is through Aerospike's provided [Deployment Manager](https://github.com/aerospike/gce-deployment-manager) templates. With these, you will be able to simply start clusters in your own network, with various instance types. 


You can take advantage of this via our [GCP Marketplace Offering](https://console.cloud.google.com/marketplace/details/aerospike-prod/aerospike-ee-3).

* **Option 1:** Using the GCP Marketplace. This is simpler as it combines the subscription and deployment steps. 
* **Option 2:** Using Deploy Manager directly. Allows customization of the Deployment Manager template before deploying, as well as streamlining namespace configurations.

There is no charge for using Aerospike's client software. Usage of Aerospike Enterprise requires an existing license with Aerospike.

# Common Steps

* Determine the instance types you'd like.

* Determine whether you'd like to use Persistent Disks, and the size of the disks.

* Create a namespace file. If you do not create one, then Aerospike will be deployed with default RAM namespaces `test` and `bar`. They may be incorrectly sized for the instances you choose.

This file will specify the namespaces the cluster has, the RAM and storage configuration, and other parameters such as replication. If you are familiar with Aerospike configuration files, these are the namespace stanzas of a configuration file, and will be appended to the end of the aerospike.conf file as-is. 

If you require persistence, this is best done with Persistent SSDs. 

{{#note}}
The configuration file must be sensible for your instance. If your cluster fails to start, or behaves poorly, the reason is likely to be the namespace configuration does not match the properties of the instance type, the Persistent Disk configuration and size, and local storage configuration.
{{/note}}

# Choose Launch Method
### Option 1: Launch with Marketplace

Navigate to the Aerospike [GCP Marketplace offering](https://console.cloud.google.com/marketplace/details/aerospike-prod/aerospike-ee-3).

Click "Launch on Compute Engine"

![Marketplace Start](/docs/deploy_guides/gcp/assets/cloud_launcher.png)

Fill out the following information for your cluster:

![Marketplace Configs](/docs/deploy_guides/gcp/assets/cloud_launcher_configs.png)

1. A deployment name
2. Zone to deploy into
3. Machine type
4. Number of nodes in your cluster

    Network Configurations:

5. Network to deploy into. This is greyed out if there is only 1 network.
6. Subnetwork to deploy into. This is greyed out if there is only 1 subnetwork.

    Disk Configurations:

7. Data disk size. This is your SSD Persistent disk. Only 1 is deployed unless Shadow Device config is checked below.
8. Local SSDs. These are ephemeral SSDs. Specify the number of local SSDs to attach.
9. Shadow Device config. This option will take the number of Local SSDs specified and provision the same number of Data disks. Each data disk will have the size specified in Data disk size field.
   * {{#note}}Data disk size should be at least 375GB for proper Shadow device configurations.{{/note}}
   * For example: Data disk size=500GB, Local SSDs=2, with Shadow Device config will result in a system with:  
       2 Local SSDs  
       2 Data Disks, 500GB each
   * See our [Shadow Device storage engine](https://www.aerospike.com/docs/operations/configure/namespace/storage#recipe-for-shadow-device) page for more details.
10. (Optional) Boot Disk Type. This can be left as Standard Persistent Disk.


Click "Deploy"

{{#info}}
The GCP Marketplace option does NOT provide custom namespace functionality at this time.
{{/info}}

{{#note}}
Aerospike Clusters deployed through GCP Marketplace do NOT have virtual addresses configured. This means that Aerospike Clients are unable to connect to deployed clusters from outside of the network that the cluster is deployed into.
{{/note}}

### Option 2: Use Deployment Manager templates

Obtain our deployment manager template from our [GitHub](https://github.com/aerospike/gce-deployment-manager).

Edit `config.yaml` with the parameters you desire.

Starting from resources->properties:

1. numReplicas - The number of nodes in your cluster.
2. namePrefix - The naming schema of your nodes.
3. zone - The zone to deploy your cluster
4. machineType - Your prefered VM size
5. network - Your network
6. bootDiskType - The disk type of the OS disk
7. numLocalSSDs - The number of locally attached NVMe SSDs. Note that these are an additional cost
8. useShadowDevice - This option will take the number of Local SSDs specified and provision the same number of Data disks. Each data disk will have the size specified in Data disk size field.
   * {{#note}}Data disk size should be at least 375GB for proper Shadow device configurations.{{/note}}
   * For example: Data disk size=500GB, Local SSDs=2, with Shadow Device config will result in a system with:
        2 Local SSDs
        2 Data Disks, 500GB each
   * See our [Shadow Device storage engine](https://www.aerospike.com/docs/operations/configure/namespace/storage#recipe-for-shadow-device) page for more details.
9. diskSize - The size of the attached Data Disk. Use `0` to not use one.
10. namespace - Your namespace stanza. This makes the following section redundant.


Deploy by running:
```bash
$ gcloud deployment-manager deployments create <project name> --config config.yaml
```

or by running the included script which simply wraps the above command:
```bash
$ ./deploy.sh <project name>
```

## Customize Namespace

{{#info}}
This section is redundant if you specified a namespaze stanza from Option 2 above.
{{/info}}

Log into each server and add your custom namespace stanzas. Be sure to take advantage of the disk configurations that was made during initial deployment.

Local disks can be found under `/dev/disk/by-id/google-local-ssd-{disk#}` where {disk#} is the id number of the Local SSD.

Data disks can be found under `/dev/disk/by-id/google-{deployment-name}-{vm-id}-data-{disk#}`, where {deployment-name} is the name chosen during deployment,
{vm-id} is the id number of the vm within the cluster, and {disk#} is the id number of the disk to the VM. 

* {disk#} is 1 if deployed without the Shadow Device option.
* {deployment-name}-{vm-id} is essentially {vm-name} without the '-vm' suffix.

Data disks deployed with the Shadow Device config is named `/dev/disk/by-id/google-{deployment-name}-{vm-id}-shadow-data-{disk#}`

An example namespace config with 1 Local SSD and 1 Shadow Device:

```
namespace ssd {
    replication-factor 2
    memory-size 16G

    storage-engine device {
        device /dev/disk/by-id/google-local-ssd-1 /dev/disk/by-id/google-aerospike-ee-1-1-shadow-data-1
		write-block-size 512k
    }
}
```

# Confirm cluster

In order to use your cluster, you will need the IP addresses of the instances for use in Aerospike tools or Aerospike enabled applications.

Go to your GCP Console and find the IP addresses of the instances that were created. The first node's public IP address is also given as an output of the deployment.

Fire off some load using the [java benchmark client](/docs/client/java/benchmarks.html) and watch the
  load with [AMC](/docs/amc) 

# System Access

SSH access is enabled via the browser through the Browser Console, or through `gcloud compute ssh`. Once your ssh keys have been imported into the instance, you may use any other SSH utilites to connect.

The ssh user is `ubuntu`.

# Architecture

Deployment Manager attaches every instance's private IP address to their metadata. On startup, each instance will run a script to rewrite Aerospike's
config file to specifically cluster together nodes specified in the aforementioned metadata.

By default, Aerospike port 3000 and AMC port 8081 are open globally (0.0.0.0/0). You may want to lock this down to just your own IP range.

The base image that Aerospike is deployed on already contains the following optimizations:
* scsi-mq enabled for local SSDs.
