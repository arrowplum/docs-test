---
title: Aerospike Connect for Spark Tutorial
description: Learn how to deploy and configure Aerospike Connect for Spark.
---

## Part 3 : Spark SQL and Aerospike

Aerospike Connect for Spark allows you to take advantage of Spark SQL and push the query filters down to the Aerospike database. This helps minimize the number of records that are physically loaded into memory.
In this section of the tutorial we will look at using Aerospike Connect for Spark and Spark SQL to query records from Aerospike.

#### Aerospike Connect for Spark configuration

Assume you have a ```people``` set in Aerospike and would like to load the records for the people who are at least 18 years old. First set the format of the context to ```com.aerospike.spark.sql``` and configure the aerospike set to use for the query:

```scala
    val peopleDF = sqlContext.read.
      format("com.aerospike.spark.sql").
      option("aerospike.set", "people").
      load
```

#### Create a view and perform the query

Now build the logical view and execute the query. 

```scala
    peopleDF.createOrReplaceTempView("people")

    val voterDF = spark.sql("select * from people where age >= 18")
    voterDF show
```

The ```voterDF``` DataFrame will contain all the records from ```people``` that are at least 18 years old.

#### Summary

The Spark SQL extensions built into Aerospike Connect for Spark allow you to use a familiar language to easily select and filter data from Aerospike when loading it into Spark for processing.

#### Continue to Part 4

In [Part 4](/docs/connectors/enterprise/spark/tutorial_4.html) of this tutorial we will look at ways to tune the performance of Aerospike Connect for Spark.

