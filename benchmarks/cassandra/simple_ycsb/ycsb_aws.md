---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" on AWS - YCSB
description: Comparing Aerospike and Cassandra Database On AWS
styles:
  - /assets/styles/ui/steps.css
---

As mentioned previously, the Aerospike AMI is currently available in the US-West (N. California) region only.


{{#info markdown=true}}
The Aerospike AMI is based on Amazon Linux, not CentOS. Indeed, the best practice from most vendors is to use Amazon Linux, which shares binary and other compatibility with CentOS 6 but includes Amazon-approved drivers and tuning.
{{/info}}

{{#info markdown=true}}

You will need to log into your instances. As this AMI is based on Amazon Linux, login by entering the following:
```bash
$ ssh -i [KEYFILE] ec2-user@[INSTANCE_IP]
```
where KEYFILE is your private key used to launch the instance, and INSTANCE_IP is the public IP address of the instance.
{{/info}}

To set up the YCSB client instance, follow these steps:

{{#steps}}

{{!--- 
  ############################################################################
  #
  # STEP 1 - Planning
  #
  ############################################################################ 
---}}
{{#steps-step 1 "Planning" markdown=true}}
While you may choose to use any instances, we recommend a single c3.8xlarge instance. An instance this large is required to stress the servers, and avoids the complexity of merging multiple YCSB result files.

You must have sufficient RAM, disk space, CPU and network to make full use of the YCSB client.

{{#note markdown=true}}
It is highly recommended to select the same availability zone for all instances in this testing. While different deployment patterns may mandate different placements, a performance test with all the servers in the same availability zone will yield the most comparable results. Using a placement group is also recommended, but not required.
{{/note}}

{{/steps-step}}

{{!--- 
  ############################################################################
  #
  # STEP 2 - Launch the YCSB Client Instance
  #
  ############################################################################ 
---}}
{{#steps-step 2 "Launch the YCSB Client Instance" markdown=true}}


To set up an instance for the client:
  1. Login to the AWS console and navigate to the "EC2 dashboard".
  1. Make sure you are using the "US West (N. California)" region (upper right-hand corner).
  1. Go to the "Instances" page (left-hand menu bar).
  1. Click on "Launch Instance".
  1. In "Step 1: Choose an Amazon Machine Image (AMI)", click on "Community AMIs" on the left-hand bar.
  1. In the "Search community AMIs" text box, enter `aerobench` to search for the Aerospike Benchmark AMI. Select the latest version of the aerobench AMI. Version 1.0 was used for these instructions.
  1. In "Step 2: Choose an Instance Type", select the instance type for the YCSB instances, which is c3.8xlarge (if you want to use VPCs, you may elect to use c4.8xlarge). Once you have selected the instance type, click on "Next: Configure Instance Details" at the bottom right.
  1. In "Step 3: Configure Instance Details", enter "1" for the number of instances. Under "Subnet", choose the subnet of the availability zone you have chosen. If you are using VPCs, you will need to change "Auto-assign Public IP" to "Enable" in order to get a public IP address.  Click on "Next: Add Storage."
  1. In "Step 4: Add Storage", for the Root volume, change the size to 18 GiB. This will make room for the benchmark logs. Click on "Next: Tag Instance".
  1. In "Step 5: Tag Instance", you may enter any tags you feel are relevant here. Click on "Next: Configure Security Group".
  1. In "Step 6: Configure Security Group", you will need to create a new security group to open the required ports:
    1. Enter "aerobench" into the "Security group name" box.
    1. Enter "Aerospike benchmark" into the Description.
    1. Click on "Add rule" at the bottom and enter "3000-3004" and change the source from "Custom TCP" to "Anywhere".
    1. Click on "Add rule" at the bottom and enter "7000" and change the source from "Custom TCP" to "Anywhere".
    1. Click on "Add rule" at the bottom and enter "7199" and change the source from "Custom TCP" to "Anywhere".
    1. Click on "Add rule" at the bottom and enter "9042" and change the source from "Custom TCP" to "Anywhere".
    1. Click on "Review and Launch" on the bottom right.
  1. Check all values and hit "Launch" in the lower right-hand corner.
  1. Select your keys, click on the acknowledgment, and hit "Launch Instances". 

Your instance should launch in roughly 5-10 minutes.


{{/steps-step}}

{{!--- 
  ############################################################################
  #
  # STEP 3 - Next Steps
  #
  ############################################################################ 
---}}

{{#steps-step 3 "Next Steps" markdown=true}}
All the software on the instance is already loaded and configured, so there is nothing else to do here.

Now you must configure the database instances with Aerospike. Because those steps will make full use of the SSDs, you should not work on the Cassandra instructions until you have completed your tests of Aerospike.

Now [launch/configure Aerospike on AWS](/docs/benchmarks/cassandra/simple_ycsb/aerospike_aws.html).
{{/steps-step}}

{{/steps}}
