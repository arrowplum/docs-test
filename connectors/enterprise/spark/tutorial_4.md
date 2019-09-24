---
title: Aerospike Connect for Spark Tutorial
description: Learn how to deploy and configure Aerospike Connect for Spark.
---

## Part 4 : Performance Tuning

Aerospike is a highly performant database and in order to take advantage of this performance you will need to tune Aerospike Connect for Spark for high performance as well.

For details on Spark tuning, please see the [tuning documentation](http://spark.apache.org/docs/2.3.0/tuning.html) on the Spark website.

In this section we will discuss a few points related to performance tuning of Aerospike Connect for Spark.

#### Number of partitions

Number of partitions (parallelism) :
Spark divides data into a number of partitions using the divide and conquer principle. Since Spark was written as an alternative to Mahout on Hadoop, by default, it will create a partition for each block of the Hadoop Distributed File System (HDFS) and hence, the default size of the block is 64MB. The default size can be extended to 128MB.

The recommended partitions could be same as the number of cores. But it can be extended.
For example, to read a text file to create a RDD using number of partitions,

```
	val conf = new SparkConf().setAppName("test").setMaster("local")
	val sc = new SparkContext(conf)
	val lines = sc.textFile("<data file>", 10)
```

where 10 is the minimum number of partitions.

Avoid setting too many partitions as it will increase the communication cost involved.

#### Batch max requests

The ```aeroJoin``` function uses Aerospike batch read requests in order to read data from the Aerospike database. The Aerospike database has a configuration for ```batch-max-requests```. Aerospike Connect for Spark also has a configuration for batch max requests:  ```aerospike.batchMax```. The default for both of these is 5000. The ```aerospike.batchMax``` should not be set to a value > ```batch-max-requests```. The configuration for these batch request settings can be tuned to suit your use case.

#### Configuring memory

To avoid getting "out of memory" errors when creating a DataFrame from an external data file or external source, configure it as follows:

Change directory to spark conf
```
	cd /opt/aerospike-spark-1.0.0/conf/
```

Check spark-defaults.conf file. If it doesn’t exist then create one as follows:
```
	cp /opt/aerospike-spark-1.0.0/conf/spark-defaults.conf.template  /opt/aerospike-spark-1.0.0/conf/spark-defaults.conf
```

Edit spark-defaults.conf, set memory to 3g or greater
```
	spark.driver.memory    3g
```

#### Data serialization

Spark communicates between nodes in the cluster or two different processes over the network. To optimize communication, it serializes the data. Serialization can be done using different methods. Normally, spark will use Java serialization. However, better speed and performance can be achieved using the Kryo library. 
```
	val conf = new SparkConf().setAppName("test").setMaster("local")
	conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```

You may additionally register the classes you intend to serialize with Kryo in advance for better performance. (Use conf.registerKryoClasses(.....))
This will improve performance when Spark shuffles the data during map-reduce and while serializing the RDD to disk.

#### Worker thread pool

Aerospike Connect for Spark uses a pool of worker threads when writing data from a DataFrame to an Aerospike database. The default number of threads in this pool is 15. Depending on a number of variables including the Aerospike cluster size and configuation, network performance and record size you may need to adjust the size of this worker pool to optimize the performance.
```
	val conf = new SparkConf().setAppName("test").setMaster("local")
	conf.set("aerospike.maxthreadcount", "20")
```

#### Conclusion

That concludes our tutorial on Aerospike Connect for Spark. Be sure to check out [configuration reference](/docs/connectors/enterprise/spark/reference.html) which describes all the configuration parameters for Aerospike Connect for Spark.
