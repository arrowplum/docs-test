---
title: Install on Amazon EC2
description: Aerospike is providing Aerospike Community Server as an Amazon Machine Image (AMI) for quick setup.
styles:
  - /assets/styles/ui/steps.css
scripts:
  - /assets/scripts/utils/target_blank.js
steps:
  next_steps: 10
  
---

{{#note}}
For quickly spinning up a cluster, consider using Cloudformation. Please see the
[Amazon Cloudformation](/docs/deploy_guides/aws/cloudformation) page for instructions.
{{/note}}

{{#info}}
This quick start guide is meant for development purposes using in-memory storage only.
For further details and deployment planning in AWS, please see [Amazon EC2 Capacity Planning](/docs/operations/plan/ec2_capacity)
{{/info}}

{{#info}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/info}}

To quickly start using the Aerospike Community server, follow these instructions.

{{#steps}}
{{#steps-step 1 "Obtain Aerospike AMI" markdown=true}}

You can either use the Aerospike Community Edition AMI on the AWS Marketplace, or prepare your own AMI.

---

**Aerospike on AWS Marketplace**

1. Click the **Aerospike AMI on AWS Marketplace** button to view the Aerospike product page.

{{steps-action "Aerospike AMI on AWS Marketplace" "https://aws.amazon.com/marketplace/pp/B00LW9382A/"}}

2. Click on the **Continue** button on that page; you will be taken to the **Launch on EC2** page.

3. Click on the "**Manual Launch**" tab, then click on "*Launch with EC2 console**" for the region you want to use.
{{#info}}
For information on the other Deployment Options (Existing VPC Cluster or New VPC Cluster), see the [Cloudformation](/docs/deploy_guides/aws/cloudformation)
page.
{{/info}}

---

**Prepare your own AMI**

1. Launch an OS of your choice, then [install Aerospike](https://www.aerospike.com/docs/operations/install/linux)
according to the OS you've chosen. 
{{#info}}
We recommend using the latest Amazon Linux for best performance and compatibilty within AWS.
{{/info}}

2. Optionally: [Install AMC](https://www.aerospike.com/docs/amc/install)

3. Set the Aerospike service (and AMC service if installed) to start on boot.

{{#note}}
The commands for setting a service to start on boot is as follows:

* RHEL: `chkconfig aerospike on; chkconfig amc on`
* Debian/Ubuntu: `update-rc.d aerospike defaults; update-rc.d amc defaults`
* SystemD: `systemctl enable aerospike; systemctl enable amc`

{{/note}}

4. [Create your AMI](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html) once you've configured your
[Namespace](https://www.aerospike.com/docs/operations/configure/namespace) and other [configuration parameters](https://www.aerospike.com/docs/operations/configure)

{{/steps-step}}
{{#steps-step 2 "Instance Type" markdown=true}}

Aerospike recommends using the **r3.2xlarge** instance or higher depending upon your RAM requirement for data storage, but you can go as small as the t2.small.

For more advice on choosing instance types, see [Amazon EC2 capacity planning](/docs/operations/plan).

{{/steps-step}}
{{#steps-step 3 "VPC (Networking)" markdown=true}}

Aerospike recommends deployment into an Amazon VPC. For setup and details, please refer to this Amazon documentation:

{{steps-action "VPC Documentation" "http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html"}}

{{/steps-step}}
{{#steps-step 4 "Security Group" markdown=true}}

Aerospike requires the following ports to be open in your security group (create a new security group if required): 

| Protocol | Port | Description |
| --- | --- | --- |
| TCP | 22 | SSH Port for accessing the instance. |
| TCP | 3000-3003 | Aerospike ports, for clients and other servers to communicate with this instance. |
| TCP | 8081 | HTTP Port for accessing AMC via a web browser. |

{{/steps-step}}
{{#steps-step 5 "Launch" markdown=true}}

If you have completed the above steps, launch your instance and ssh into it.

An in-memory test namespace is configured by default. Please see the [configuration section](/docs/operations/configure) to add storage devices, configure a cluster, and tune your configuration to your hardware.

{{/steps-step}}
{{#steps-step 6 "Start Aerospike and AMC" markdown=true}}

```
[ec2-user@ip-xxx-xx-x-xxx ~]$ sudo service aerospike start
Starting and checking aerospike: asd (pid 8166) is running...
                                                           [  OK  ]```
```
[ec2-user@ip-xxx-xx-x-xxx ~]$ sudo service amc start
Starting AMC....
AMC is started.
```

{{/steps-step}}
{{#steps-step 7 "Connect to AMC" markdown=true}}

Connect to AMC by going to the following URL: `http://<Public_IP>:8081` where 'Public_IP' is the public IP address of the EC2 instance launched. 

Starting from AMI for Aerospike server 3.4.1, you will get a prompt for authorization. 
The username is `amc`.  

The password is the instance-id of the instance you are running. You can find your instance-id by looking on the EC2 console, or by running in a terminal from within the instance:

 `curl http://169.254.169.254/latest/meta-data/instance-id`
    


{{/steps-step}}
{{#steps-step 8 "Clustering Aerospike" markdown=true}}

Since multicast traffic is not permitted in AWS, a cluster must be formed utilizing 
[mesh configuration](/docs/operations/configure/network/heartbeat#mesh-unicast-heartbeat).

SSH into each instance and configure mesh networking on the private IP address for each node.

{{/steps-step}}
{{#steps-step 9 "Amazon EC2 Recommendations" markdown=true}}

For setting up a cluster on Amazon EC2, read the recommendations.

{{steps-action "Recommendations for Amazon EC2" "/docs/deploy_guides/aws/recommendations"}}

{{/steps-step}}
{{/steps}}

{{md (path-resolve (path-dirname page.src) "/docs/operations/install/common/nextsteps.part.md") }}
