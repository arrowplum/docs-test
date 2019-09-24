---
title: Recreating the Blog Post "Comparing NoSQL Databases&#58; Aerospike and Cassandra - Benchmarking for Real" on AWS
description: These steps will let you run the benchmark within the AWS environment using the Aerospike Aerobench AMI.
styles:
  - /assets/styles/ui/steps.css
---
You will need to set up an environment with the appropriate instances. Aerospike has created an AMI for your use that has Aerospike, Cassandra, and the client components preloaded.

{{#steps}}

{{!---
  ############################################################################
  #
  # STEP 1 - Planning
  #
  ############################################################################
---}}
{{#steps-step 1 "Planning" markdown=true}}
The Aerospike AMI is currently available in the US-West (N. California) region only. This AMI comes preloaded with all the software necessary for testing, and will require only minimal work to configure. These instructions use AWS instances that are close to the bare metal hardware we used in the blog post.

While not required, you may elect to use AWS Placement Groups. For a list of recommendations on production deployments on AWS, please see the [Aerospike AWS Recommendations Guide](/docs/deploy_guides/aws/recommendations).

## Amazon AWS Instances
US-West (N. California)

| Purpose         | Instance Type | Quantity |
| --------------- | ------------- | --- |
| Database server (Aerospike and Cassandra) | c3.8xlarge     | 3 |
| YCSB            | c3.8xlarge    | 1 |

{{/steps-step}}

{{!---
  ############################################################################
  #
  # STEP 2 - Next Steps
  #
  ############################################################################
---}}
{{#steps-step 2 "Next Steps" markdown=true}}
Once you have decided on the instance types to use, run the benchmarks. When you run the benchmarks, you will have to start with one database, run your tests, and then reset and do the same with the other. A reset is necessary because the SSD setup of the two databases is not compatible. 

Do them in the following order:
  1. [Set up the YCSB (client) instance](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aws.html)
  1. [Configure Aerospike on AWS](/docs/benchmarks/cassandra/simple_ycsb/aerospike_aws.html)
  1. [Run the YCSB Benchmark on Aerospike](/docs/benchmarks/cassandra/simple_ycsb/ycsb_aerospike.html)
  1. [Reconfigure the database instances for Cassandra](/docs/benchmarks/cassandra/simple_ycsb/cassandra_aws.html)
  1. [Run the YCSB Benchmark on Cassandra](/docs/benchmarks/cassandra/simple_ycsb/ycsb_cassandra.html)
  1. [Generate graphs from your data](/docs/benchmarks/cassandra/simple_ycsb/graphs.html)
{{/steps-step}}

{{/steps}}
