---
title: Step-by-step instructions to maximize throughput on Amazon EC2 to obtain 1M TPS
description: This tutorial covers how to tune a single Amazon EC2 instance to achieve 1 Million transactions per second with Aerospike.
styles:
  - /assets/styles/ui/steps.css
---
<style>
ol.steps {
  padding-top: 1em;
}
</style>

### Initial Setup

{{#steps}}
{{#steps-step 1 "Create a VPC" markdown=true}}

{{#info}}
This is a requirement for EC2’s Enhanced Networking feature, which is needed for obtaining the high network throughput required for this benchmark.
{{/info}}

- In the AWS console, click on “**Services**” -> “**VPC**”. Click on “**Start VPC Wizard**”
- Select the “**VPC with Single Public Subnet**” option
- In the Step 2 page, enter the following details:
  - IP CIDR block: 10.2.0.0/16
  - VPC name: aerospike-benchmark-vpc
  - Public subnet: 10.2.0.0/24
  - Availability zone: us-east-1a
  - Subnet name: aerospike-benchmark-subnet
  - Leave the rest of the options to defaults and click on “**Create VPC**”

{{/steps-step}}
{{#steps-step 2 "Create a Security Group" markdown=true}}

- In the VPC dashboard on the AWS console, click on the “**Security Groups**” item in the left menu
- Click on “**Create a Security Group**”
- Give the name, group-name and description as “**aerospike-benchmark-sg**”
- Select the VPC which you created above i.e. “**aerospike-benchmark-vpc**”
- Next, add “Inbound Rules” in this Security Group
- Create a rule for SSH
  - Type: SSH
  - Source: 0.0.0.0/0
- Create a rule for the Aerospike server
  - Type: Custom TCP Rule
  - Port Range: 3000-3003
  - Source: 0.0.0.0/0

{{/steps-step}}
{{#steps-step 3 "Create a Key Pair" markdown=true}}
  
- In the EC2 dashboard on AWS console, click on “**Key Pairs**"
- Create a Key Pair with the name “**aerospike-benchmark-key**” and download the pem file on your local machine. You will need this to connect via SSH to the instances.
- You should change the permissions to the pem file so that its readable only by you
```chmod 400 aerospike-benchmark-key.pem```
  
{{/steps-step}}
{{/steps}}

### Launch EC2 Instance

{{#steps}}
{{#steps-step 4 "Launch Instance" markdown=true}}

- On the AWS console, go to “**Services**” -> “**EC2**” and click on “**Launch Instance**”
- Select the “**Amazon Linux (HVM)**” AMI. It is important that the virtualization type be HVM. This is usually the first AMI listed.
- Step 2: Choose the r3.8xlarge instance and click “Next”
- Step 3: Configure Instance. Use the following details:
  - Number of instances: 1
  - Network: Use the “**aerospike-benchmark-vpc**” which you created earlier
  - Subnet: Select “**aerospike-benchmark-subnet**”
  - Auto-assign Public IP: Enable
  - Placement Group: Create a new placement group, and name it “**aerospike-placement**”
  - Leave the rest of the options their default value
- Step 4: Add Storage
  - Increase the root device size to 20 GB
  - Select the Volume Type as “**Magnetic**” (We will be doing an in-memory database test, so the storage type does not matter)
- Step 5: Tag Instance. Put a name=aerospike-server tag for easy identification
- Step 6: Configure Security Group. Select the security group “**aerospike-benchmark-sg**” which you created earlier
- Step 7: Review and Launch the instance. Use the “**aerospike-benchmark-key**” when prompted for keypair
- Go to the EC2 dashboard and click on “Instances”
- Wait for the instance to get into the “Running” state and note down its Public IP address

{{#note}}
Note that placement groups are not recommended for Production Workloads.
{{/note}}

{{/steps-step}}
{{/steps}}

### Instance Setup and Aerospike Install

{{#steps}}
{{#steps-step 5 "Enhanced Networking" markdown=true}}

- Login to the instance:
```
ssh -i /path/to/aerospike-benchmark-key.pem ec2-user@<Public-IP-of-server>
```
- Once you are logged in to the instance, type:
```
sudo su - 
yum update
```
- Check if “Enhanced Networking” is enabled:
```
ethtool -i eth0 | grep ixgbevf
```
{{#info}}
In the output, make sure the ixgbevf driver is being used. If this is not the case, make sure you used the **Amazon Linux HVM AMI**. If you want to enable Enhanced Networking on another AMI, please follow the instructions [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html).
{{/info}}

{{/steps-step}}
{{#steps-step 6 "Install Aerospike Server" markdown=true}}


- Download and install the Aerospike server
```bash
wget --output-document=aerospike.tgz https://www.aerospike.com/download/server/3.3.22/artifact/el6
tar -xzvf aerospike.tgz
cd aerospike-server-community-*-el6
./asinstall
```
{{/steps-step}}
{{#steps-step 7 "Add more ENIs" markdown=true}}


- Go to the EC2 dashboard on the AWS console
- Click on “**Network Interfaces**” -> “**Create Network Interface**”
- Use the same subnet: 10.2.0.0/24
- Use the security group created earlier: “**aerospike-benchmark-sg**”
- Click on "Create" and then "Attach". Now, select the instance-id of your server instance (You can get the instance-id of your running server instance from the EC2 dashboard of the AWS console)
- Run ifconfig on the server; the new interface should appear there
- Add 2 more ENIs so that the instance has a total of 4 network interfaces - eth0 to eth3

{{/steps-step}}
{{#steps-step 8 "Adjust network IRQ affinities" markdown=true}}


- Update aerospike.conf file to set service-threads and transaction-queues equal to number of cores. And set transaction-threads-per-queue to 3 in case there is namespace with data not in memory.

- Determine the IRQs of all the network interfaces from “/proc/interrupts”:
```bash
for i in {0..3}; do echo eth$i; grep eth$i-TxRx /proc/interrupts | awk '{printf "  %s\n", $1}'; done
```
{{#info}}
This command should give the IRQ numbers for eth0 to eth3. In this case, these were 115, 116, 118, 119, 121, 122, 124 and 125
{{/info}}
- Set up the first two processor cores to get eth0 IRQs, the next two processor cores to get eth1 IRQs, etc.
{{#note}}
Note that the number to be “echoed” is the hex bitmask denoting the CPU. (https://cs.uwaterloo.ca/~brecht/servers/apic/SMP-affinity.txt)
{{/note}}

```bash
echo 1 > /proc/irq/115/smp_affinity
echo 2 > /proc/irq/116/smp_affinity
echo 4 > /proc/irq/118/smp_affinity
echo 8 > /proc/irq/119/smp_affinity
echo 10 > /proc/irq/121/smp_affinity
echo 20 > /proc/irq/122/smp_affinity
echo 40 > /proc/irq/124/smp_affinity
echo 80 > /proc/irq/125/smp_affinity
```

{{#info}}
Remember that the CPU cores are indicated using **hexadecimal bitmask**. So, the bitmasks for processor 2 and 3 are 2^2 and 2^3 (in hexadecimal) respectively.
{{/info}}

<img src="{{book.assets}}/images/instructions1mpts_irq.png" width="800" alt="Instructions1MPTS IRQ">

{{/steps-step}}
{{#steps-step 9 "Start Aerospike Server" markdown=true}}

- Start the Aerospike server:
```
service aerospike start
```
- Note the PID of the Aerospike server:
```
cat /var/run/aerospike/asd.pid
```
- Change [process CPU affinity](http://linux.die.net/man/1/taskset) of Aerospike server:
```
taskset -p ffffff00 <PID of asd>
```
{{#info}}
This makes sure the Aerospike server does not run on the first 8 cores, leaving them completely free for handling network interrupts.
{{/info}}

{{/steps-step}}
{{/steps}}

### Client Setup and Benchmark

{{#steps}}
{{#steps-step 10 "Client Setup" markdown=true}}


- We will use 4 client instances. To get started,  click on “**Launch Instance**” on the EC2 dashboard and choose “**Amazon Linux HVM AMI**”
- Choose a **r3.2xlarge** instance
- On the "Step 3" page, change the number of instances to 4
- For the rest of steps, use the same values as mentioned in Step 4 above.
{{#note}}
Make sure you use the same “Placement Group” named "**aerospike-placement**" which was used for the server
{{/note}}
- Connect to each of the instances via SSH using the aerospike-benchmark-key.pem
- On each of the instances, type:
```
sudo yum update
```
- Make sure that “Enhanced Networking” is enabled:
```
ethtool -i eth0 | grep ixgbevf
```
{{#note}}
In the output, make sure the **ixgbevf** driver is being used.
{{/note}}

{{/steps-step}}
{{#steps-step 11 "Run Benchmark Tool" markdown=true}}

{{#info}}
We will use the Aerospike Java benchmark tool.
{{/info}}

- Begin by installing Maven:
```bash
sudo yum install -y java-1.7.0-openjdk-devel.x86_64 java-1.7.0-openjdk-javadoc.noarch
wget http://apache.spinellicreations.com/maven/maven-3/3.2.3/binaries/apache-maven-3.2.3-bin.tar.gz
tar -zxvf apache-maven-3.2.3-bin.tar.gz
export M2_HOME=~/apache-maven-3.2.3
export M2=$M2_HOME/bin
export PATH=$M2:$PATH
mvn -version
```
- Next, download and compile the benchmark tool:
```bash
wget --output-document aerospike-client-java.tgz https://www.aerospike.com/download/client/java/3.0.31/artifact/tgz
tar -zxvf aerospike-client-java.tgz
cd aerospike-client-java-*
./build_all
```
- On the server, note the 4 internal IP addresses corresponding to the 4 network interfaces:
```
for i in {0..3}; do echo -n eth$i; ifconfig eth$i | grep "inet addr"; done
```
- Run the benchmark tool with 10 million keys, 100 byte string values, 50% read and 50% write workload, with 90 threads pointing to the **eth0 IP address**:
```
cd benchmarks
./run_benchmarks -h <server-eth0-IP> -p 3000 -n test -k 10000000 -S 1 -o S:100 -w RU,50 -z 90 -latency 10,1
```
- This should give around 250 KTPS combined reads and writes:
<img src="{{book.assets}}/images/instructions1mpts_bm.png" width="800" alt="Instructions1MPTS Benchmark run">
  
- Run the benchmark tool from each of the 4 clients using the 4 different internal IPs of each of the ENIs of the server
  - This way the traffic is distributed on all of the network interfaces of the server.
  - With around 250 KTPS from each client, the server should now give a throughput of around 1 Million TPS

{{/steps-step}}
{{#steps-step 12 "Monitoring" markdown=true}}

- To check the number of keys in the database, run “**aql**” command line tool on the server:
```
aql> STAT SYSTEM
```
- Look for the line showing "**objects**"
{{#note}}
Note that as you begin the tests, the database will be empty and will gradually be filled with the 10 million records. This usually takes around 5 minutes with an 80%/20% read/write load.
{{/note}}
- Run “**top**” on the server and press “**1**”. This will display details of all the cores on the instance.
  - When running the benchmark with 4 ENIs on the server and 4 clients, you should see the first 8 cores with high interrupt processing load (`%si`). (This indicates that the network interrupts are being handled by these cores)
  - On the rest of the cores, you should see almost no interrupt handling and user-space and kernel-space load. These are the aerospike threads.

{{/steps-step}}
{{/steps}}

