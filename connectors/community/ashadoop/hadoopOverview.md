---
title: Apache Hadoop Overview
description: Overview of Apache Hadoop Ecosystem
---

[Apache Hadoop](https://hadoop.apache.org/) provides detailed introduction to Hadoop and its associated 
components comprising the Hadoop Ecosystem. Additionally, they also provide [instructions](http://hadoop.apache.org/docs/r2.7.0/hadoop-project-dist/hadoop-common/ClusterSetup.html) to set up an Apache Hadoop cluster. 
In our Hands On section, we provide step by step [instructions to set up a 3-node Apache Hadoop Cluster](/docs/connectors/community/ashadoop/handson/hadoopClusterSetup.html) along with 
a [instructions to set up a 4th node as an Edge Node](/docs/connectors/community/ashadoop/handson/hadoopEdgeNodeSetup.html) to access the cluster. 
A brief introduction to [Hadoop CLI](/docs/connectors/community/ashadoop/handson/hadoopCLI.html) is also provided.

# Apache Hadoop Ecosystem

The figure below shows some of the commonly used components in the Hadoop ecosystem. 

<center>
{{#figure "" "Apache Hadoop Ecosystem" size="large" }}
<img src="/docs/connectors/assets/images/HadoopEcosystem.png">
{{/figure}}
</center>

# HDFS

**NameNode:** Stores metadata - knows where chunks and their replicas are located.  
**Secondary NameNode:** Failover and High Availability solution to NameNode which can otherwise be a single point of failure.  
**DataNode:** Stores chunks of user data, master or replica.  

# YARN - Yet Another Resource Manager!
**JobTracker:** Keeps track of the overall job.  
**TaskTracker:** Tracks individual tasks local to the node that comprise the overall Hadoop job.  

# Data IO

**Sqoop:** For importing or exporting data between HDFS and Relational Databases.  
**Flume:** For importing or exporting data between HDFS and unstructured data sources, semi-structured data and log files.  

# Computation 

**Spark:** Designed to exploit in memory distributed computing on data stored in HDFS or other data sources, streaming batch or at rest, for interactive queries or iterative algorithms.  
**Java Map-Reduce:** Write map-reduce jobs in java - most efficient way to execute map-reduce jobs.  
**PIG Latin:** A scripting language that is extremely easy to use which under the covers compiles to map-reduce jobs.  
**Hive and Hive Query Language:** The data warehousing omponent for interacting with “tabular” data in HDFS with HiveQL being similar to SQL. Hive Queries translate to map-reduce jobs.  

# Cluster Management and Job Automation

**ZooKeeper:** It is a coordination service for distributed services. For example, it is used by HDFS and Apache HBase.  
**Oozie:** “Fire” off jobs on various Hadoop ecosystem components to run in co-ordination for automating daily schedule of operations.  
**Hue:** Hadoop User Experience is a Graphical User Interface for monitoring and managing a Hadoop cluster.  


