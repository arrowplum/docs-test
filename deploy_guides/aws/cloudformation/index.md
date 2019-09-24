---
title: CFT (CloudFormation) Quick Install
description: |
  Here are the descriptions of our CloudFormation templates and how to effectively use them in your environment.
scripts:
  - /assets/scripts/utils/target_blank.js

---

Aerospike provides CloudFormation templates to quickly provision an entire Aerospike cluster with recommended settings.

The quickest and simplest way to get Aerospike running on EC2 is through Aerospike's provided CloudFormation templates. With these, you will be able to simply start clusters in your own VPCs, with various instance types. You'll be able to add and remove cluster nodes easily.

These templates allow you use the Aerospike Community Edition (open source), with sensible default parameters. You may eventually want to configure a variety of settings. You may also want to additional features and support that come with Aerospike Enterprise. Please contact Aerospike directly.

There is no AWS MarketPlace charge for using Aerospike's client and server software. 

You can take advantage of this via our [Marketplace Offering](https://aws.amazon.com/marketplace/pp/B00LW9382A). We also publish our templates on [Github](https://github.com/aerospike/aws-cloudformation). We provide 2 options toward a quick install: 

* **Option 1:** Using the Marketplace. This is simpler as it combines the subscription and deployment steps. 
* **Option 2:** Using CloudFormation directly. Allows customization of the CloudFormation template before deploying.

# Common Steps

* Determine the instance types you'd like.

* Determine whether you'd like to use EBS for persistence, and the size of the EBS volumes. This is best done through the Amazon [EC2 Capacity Planning Guide](/docs/deploy_guides/aws/plan).

* Create a namespace file. If you do not create one, then Aerospike will be deployed with default RAM namespaces `test` and `bar`. They may be incorrectly sized for the instances you choose.

This file will specify the namespaces the cluster has, the RAM and storage configuration, and other parameters such as replication. If you are familiar with Aerospike configuration files, these are the namespace stanzas of a configuration file, and will be appended to the end of the aerospike.conf file as-is. 

{{#note}}
The CloudFormation template works best with one namespace. Only one persistant namespace can be defined, as the template is limited to one ephemeral storage device and one EBS volume. Multiple namespaces can be used if the persistence is not required by dividing the ram and/or devices amongst namespaces.
{{/note}}

If you require persistence, this is best done with EBS. EBS volumes are always type gp2 and attached under **/dev/sdg**. An ephemeral volume is always available (if the instance type has them) under **/dev/sdf** .

An example namespace configuration file is included in the Github repository that also includes the CloudFormation templates -- [aerospike/aws-cloudformation](https://github.com/aerospike/aws-cloudformation) .

This file must be located where the EC2 launch systems can find them. The simplest method is to upload a file to S3, then making the file public. The direct link is available via the properties tab of the S3 object.

{{#note}}
The configuration file must be sensible for your instance. If your cluster fails to start, or behaves poorly, the reason is likely to be the namespace configuration does not match the properties of the instance type, the EBS configuration and size, and ephemeral storage configuration.
{{/note}}

# Choose Launch Method
### Option 1: Launch with Marketplace

Navigate to the Aerospike [AWS Marketplace offering](https://aws.amazon.com/marketplace/pp/B00LW9382A).

Select New VPC Cluster or Existing VPC Cluster options from the marketplace.

![Marketplace Start](/docs/deploy_guides/aws/assets/images/marketplace_start.png)

Click Continue

Select the Version of Aerospike and Region you'd like to deploy into.

![Marketplace Launch](/docs/deploy_guides/aws/assets/images/marketplace_launch.png)

Click Launch with CloudFormation Console.

You are now taken to the CloudFormation Console with the Aerospike CFT already loaded in the S3 templace URL field. Proceed to the [Specify Details](#specify-details) section below.

![CloudFormation Start](/docs/deploy_guides/aws/assets/images/cf_start.png)

{{#note}}
**New VPC Cluster** will create an new VPC and provision all resources within it.  

**Existing VPC Cluster** will provision the Aerospike cluster in an existing VPC subnet of your choice.
{{/note}}

### Option 2: Use CloudFormation Directly

Validate the region is the one you want to use.

#### Download the template

1. Determine whether you wish to launch in an existing VPC or a new VPC.

2. Download the correct template. 
The repo is [aerospike/aws-cloudformation](https://github.com/aerospike/aws-cloudformation). You can clone this repo to your local system, or simply download the JSON template file directly from the Github web interface.

#### Create a stack

1. Go to the AWS CloudFormation console at [https://console.aws.amazon.com/cloudformation/home](https://console.aws.amazon.com/cloudformation/home) .

2. Change region to the one you've determined to use.

3. Click the "Create a stack" button.

#### Select a template

4. Supply the template. Under the "choose a template" section, select "upload to S3", and select the file you've downloaded. In future steps, you might consider reusing the S3 URL.

![CloudFormation Start](/docs/deploy_guides/aws/assets/images/cft_start.png)

# Specify details

CloudFormation will ask you for a number of parameters.

![CloudFormation Details](/docs/deploy_guides/aws/assets/images/cf_details.png)

1. Give a name to your stack.

2. Enable or disable CloudWatch. This extra AWS expense may be worth enabling if you use Cloudwatch for service monitoring.
{{#info}}
Statistics are: Cluster Integrity, Free Memory, Free Disk and Number of Objects
{{/info}}

3. Choose EBS size. If you chose persistence in EBS when you specified your cluster, enter the correct size from your sizing step. Enter 0 to not use EBS volumes. 

4. Choose an instance type from the ones available in the drop-down.
For more info on which instance to use, refer to Aerospike [AWS Capacity Planning](/docs/deploy_guides/aws/plan).

5. Choose a valid existing keypair. If you don't have a keypair in AWS already,
[create one first](http://docs.aws.amazon.com/gettingstarted/latest/wah/getting-started-create-key-pair.html) 

6. Specify the URL of your namespace file. This is usually an S3 URL as recommended above. This step can be skipped, in which case two RAM namespaces `test` and `bar` will be defined. They may be incorrectly sized for the instances you choose.

7. Enter the initial number of instances desired.

8. Enter the CIDR block from which you permit SSH access. You can use many online sites like [whatismyip](http://whatismyip.org/)
to find out your IP. For single IP addresses appending /32 is required. A "wide open" SSH configuration is "0.0.0.0/0". Only 1 entry is permitted.

9. Choose if you'd like dedicated tenancy. There will be additional costs with this option.

10. If deploying to an existing VPC, choose the VPC and the subnet within that VPC.

Click next.

# Specify tags

![CloudFormation Tags](/docs/deploy_guides/aws/assets/images/cf_tags.png)

Enter user-defined tags desired. The template does not requre any tags of this sort.

{{#note}}
It is useful to define a `Name` (case sensitive) tag, such that newly deployed instances have an name already when viewed in the EC2 console.
{{/note}}

# Review

![CloudFormation Review](/docs/deploy_guides/aws/assets/images/cf_review.png)

Check "I acknowledge that this template might cause AWS CloudFormation to create IAM resources." This is required for cluster discovery.

See Architecture section for details.

Review and click Create.

# Confirm cluster

In order to use your cluster, you will need the IP addresses of the instances for use in Aerospike tools or Aerospike enabled applications.

Go to your EC2 console and find the IP addresses of the instances that were created.

Fire off some load using the [java benchmark client](/docs/client/java/benchmarks.html) and watch the
  load with [AMC](/docs/amc) 

# System Access

SSH access is enabled on the instances under the **ec2-user** user using the key-pair you've selected during stack creation.
You are prompted to enter the IP (in CIDR format) from where you permit SSH access also during stack creation. 

# Cost
* EC2 instances (4 x t2.large ~= $270/mo)
* GP2 EBS volume per instance (4 x 50GB ~= $20/mo)
* Cloudwatch metrics per instance
  * 4 metrics x 5 minute polling ~= 35000 API requests/mo = $0.35 + $2 for 4 metrics ~= $2.35/mo
* SQS queue
  * 1 message on creation, 1 message per instance per scale-in. Fits into free tier. Would require a constant >2.5 scale-in events per second to exceed free tier.
* Dedicated tenancy. See [Dedicated instance pricing](https://aws.amazon.com/ec2/purchasing-options/dedicated-instances/) on AWS.


# Architecture

Upon instance startup, instances will run a userdata script that will query AWS for instances based on the unique StackID tag CloudFormation generates.
This functionality requires the ec2-describe instance policy and utilizes IAM roles for this.

This script will then determine the private IP addresses and modify the clustering section of aerospike configs to use those addresses.

This cluster is resiliant to any node being added/dropped. Additional nodes added with autoscaling will be able to automatically join the cluster.
**Nodes leaving the cluster must be triggered by autoscaling to avoid data loss**. The scripts make sure
that nodes are removed only when data is fully replicated. The script does this by sending an SQS message with the node ID that will be terminated. The each node polls SQS for its own message, and once the node finds an SQS message for itself, it first
checks for data migrations. If no migrations are occuring, it data is fully replicated in the cluster and node removal is safe.  If there are data
migrations occuring, then node shutdown is unsafe, and the node will not be removed. 

By default, Aerospike port 3000 and AMC port 8081 are open globally (0.0.0.0/0). You may want to lock this down to just your own IP range. As Aerospike Community Edition does not include security, you will certainly want
to change this default, or use Aerospike Enterprise Edition.

# Pricing
The Aerospike AMI contains only free and open source software. You will be prompted to subscribe to the AMI before this CF template can be used if you deploy directly from 
CloudFormation. Pricing is entirely the standard AWS cost of instances. Please see EC2 pricing [here](https://aws.amazon.com/ec2/pricing/). 
