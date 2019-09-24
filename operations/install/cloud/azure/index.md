---
title: Install on Amazon
description: Aeropsike provides an Amazon Machine Image (AMI) to assist in quickly getting an Aerospike Community Server up and running on Amazon EC2.
styles:
  - /assets/styles/ui/steps.css
---

Aeropsike provides an Amazon Machine Image (AMI) to assist in quickly getting an Aerospike Community Server up and running on Amazon EC2.

---

<center><a href="https://aws.amazon.com/marketplace/pp/B00GKAVFXA" class="btn btn-lg btn-primary"><span class="fa fa-chevron-right"></span>&nbsp;Aerospike AMI on AWS Marketplace</a></center>

---

The following are instructions for creating an EC2 instance using the Aerospike AMI via AWS Marketplace.

{{#steps}}
{{#steps-step 1 "View the Aerospike on AWS Marketplace" markdown=true}}

Click on the **Aerospike on AWS Marketplace** button to view the Aerospike product page on AWS Marketplace. 

{{steps-action "Aerospike AMI on AWS Marketplace" "https://aws.amazon.com/marketplace/pp/B00GKAVFXA"}}

---

The product page provides details of the Aerospike AMI, including product description and pricing based on the instance type. Once you are ready to continue to the next step, click on the <img src="{{book.assets}}/aws-continue.png" alt="AWS Continue Button" style="height: 20px"/> button on that page, and you will be taken to the **Launch on EC2** page.

{{/steps-step}}
{{#steps-step 2 "Launch on EC2" markdown=true}}

The **Launch on EC2** page provides two options for creating a new EC2 instance:

- 1-Click Launch
- Launch with EC2 Console

Both options require you to review various options. Each of the remaining steps address each of the options. If you want to launch multiple instances together, then you should use the "Launch with EC2 Console" option and then configure the number of instances under the "Configure Instance" section of EC2 Console.

{{/steps-step}}
{{#steps-step 3 "Instance Type" markdown=true}}

The instance type defines the hardware / resources that will be available to the machine. Aerospike recommends using the **m1.large** instance. However, you can go as small as the m1.small or to large machines with more CPU and memory, such as c3.8xlarge. Aerospike only requires the machine to have at least 1 GB of RAM.

{{#warn}}
**WARNING** – Please note that **t1.micro** will not work with Aerospike. Aerospike requires a minimum of 1 GB or RAM, while the t1.micro only has 613 MB.
{{/warn}}


{{/steps-step}}
{{#steps-step 4 "VPC (Networking)" markdown=true}}

VPC (Virtual Private Cloud) is the mechanism Amazon provides to define a virtual network for a group of EC2 instances. Usually, a default is provided. If you do not have a default VPC or prefer to customize it, then you can create one and use it.

For details on VPC, please refer to the documentation linked below:

{{steps-action "VPC Documentation" "http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html"}}

{{/steps-step}}
{{#steps-step 5 "Security Group" markdown=true}}

The Aerospike AMI comes with a predefined security group. You can use this security group, or define your own. You will want to keep in mind that the security group should expose the following ports:

| Protocol | Port | Description |
| --- | --- | --- |
| TCP | 22 | SSH Port for accessing the instance. |
| TCP | 3000-3004 | Aerospike ports, for clients and other servers to communicate with this instance. |
| TCP | 8081 | HTTP Port for accessing AMC via a web browser. |

{{/steps-step}}
{{#steps-step 6 "Key Pair" markdown=true}}

Amazon requires users to utilize key pairs to access instances. If you do not already have a key pair, Amazon will guide you through the process of creating a new key pair, which you can then attach to your EC2 instance.

{{/steps-step}}
{{#steps-step 7 "Launch" markdown=true}}

Once you have ensure the above steps have been satisfied, then you can launch your new instance.

{{/steps-step}}
{{#steps-step 8 "Start Aerospike and AMC" markdown=true}}

Connect to your instance and start Aerospike and AMC:

```bash
$ ssh -i key_pair.pem ec2-user@xx.xx.xx.xx

[ec2-user@ip-172-xx-x-xxx ~]$ sudo service aerospike start
Starting and checking aerospike: asd (pid 8166) is running...
                                                           [  OK  ]
[ec2-user@ip-172-xx-x-xxx ~]$ sudo service amc start
Starting AMC....
AMC is started.
```

{{/steps-step}}
{{#steps-step 9 "Connect to AMC" markdown=true}}

Connect to AMC by going to the following URL: `http://<Public_IP>:8081` where 'Public_IP' is the public IP address of the EC2 instance launched.

{{/steps-step}}
{{#steps-step 10 "Verify" markdown=true}}
Use the Aerospike command line interface tool (installed as `/opt/aerospike/bin/cli1` and linked in `/usr/bin/cli`) to insert and read a few sample records. Start by creating a new object with the key “Aerospike” in the “test” namespace that is part of the default configuration and adding three fields, “name”, “address” and “email”:

```bash
$ cli -h 127.0.0.1 -n test -o set -k Aerospike -b name -v "Aerospike, Inc."
succeeded: key= Aerospike  set=   bin= name  value= Aerospike, Inc.
$ cli -h 127.0.0.1 -n test -o set -k Aerospike -b address -v "Mountain View, CA 94043"
succeeded: key= Aerospike  set=   bin= address  value= Mountain View, CA 94043
$ cli -h 127.0.0.1 -n test -o set -k Aerospike -b email -v "info@aerospike.com"
succeeded: key= Aerospike  set=   bin= email  value= info@aerospike.com
```

Retrieve the record and make sure it looks right:
```bash
cli -h 127.0.0.1 -n test -o get -k Aerospike
{'email': 'info@aerospike.com', 'name': 'Aerospike, Inc.', 'address': 'Mountain View, CA 94043'}
```

Delete the record and verify that it is deleted:
```bash
$ cli -h 127.0.0.1 -n test -o delete -k Aerospike
delete succeeded: key= Aerospike  set=
```

Note that the cli tool is meant to be used for basic validation only. Aerospike Database is not intended to be used through a command line tool. The inefficiencies of creating a new connection for every transaction would be overwhelming. Instead, use your application, integrated with Aerospike’s client libraries as described in the [DEVELOPMENT section](/develop/).
{{/steps-step}}

{{#steps-step 11 "Next Steps" markdown=true}}
Now that you have a running server, let's put it to good use. Pick a client and start developing!
<div style="text-align: right;">
<a class="btn btn-primary" href="/develop">DEVELOP</a>
</div>

---

- [Benchmark](/docs/operations/install/common/benchmark.html) your installation.
- Install and connect to the [Aerospike Management Console](/docs/amc). This is already installed as part the Aerospike Vagrant box and Amazon AMI image.
- [Configure](/docs/operations/configure) your node further and form a cluster by adding more nodes.
- Take a look at the complete set of [Tools & Utilities](/docs/tools).


{{/steps-step}}
{{/steps}}
