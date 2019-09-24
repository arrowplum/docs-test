---
title: Aerospike Connect for Spark
description: Learn about Aerospike Connect for Spark.
---

### Overview

The Aerospike Connect for Spark addresses the challenges of real-time business insights by leveraging the speed of Aerospike to accelerate complex event processing.

Aerospike Connect for Spark provides the capability to gain closed-loop business insights by operating on both transactional and stream datasets.

Lower TCO for real-time analytics can be obtained since Aerospike Conect for Spark can operate on larger datasets yet with a smaller cluster size.


### Ways to use the Aerospike Connect for Spark

#### Joins using Aerospike

Aerospike Connect for Spark provides a join function which allows you to read records from Aerospike given a Dataset which contains keys to the records of interest. This operation takes advantage of Aerospike's batch read functionality.

#### Use Spark SQL to fetch data from Aerospike

Aerospike Connect for Spark provides the capability to use Spark SQL in order to query records from an Aerospike cluster.

#### Load Aerospike data into Spark for processing

Through the use of Aerospike's Spark connector, you can easily load data already in Aerospike into Spark for proccessing. As Aerospike supports queries and indexes, you can load just the data you need for this job.

A practical example would be a user profile store running at high velocity supporting an application that needs personalization, and is using Spark to generate or update models. By adding a secondary index on an Aerospike "bin" (column) for a "last modified time", you can easily pull profiles that have been modified recently. As Aerospike supports batch reads, and efficient in-memory row-based access, you won't have to scan through mountains of HDFS data to find the few profiles that have been updated.

#### Aerospike DataFrames

Using Aerospike as the backing store for your DataFrame will improve deployability, and allow you to work efficiently over more data. Aerospike efficiently supports Flash (SSD) storage, which will allow you to easily access terabytes of real-time storage inexpensively. As this would be a shared DataFrame - supporting atomic in-database increments and more complex user defined data structures through atomic in-database user defined functions - you can use it for multiple computations simultaneously.

#### Shared counters in Aerospike

A common need for Spark jobs is counters. Whether it's counting packets by IP address for Fraud detection, or counting in-game clicks for monitization optimization, you'll need counters. Counters are simple in Aerospike: there is an in-database increment feature, and you can keep a number of counters grouped together under one key: or keep a single 64 bit integer under a single key (use "data in memory" and "data in index" for higher performance).

### Tutorial

We have created a [tutorial](/docs/connectors/enterprise/spark/tutorial.html) which demonstrates how to configure and run Aerospike Connect for Spark.
