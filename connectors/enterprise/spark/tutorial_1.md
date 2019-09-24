---
title: Aerospike Connect for Spark Tutorial
description: Learn how to configure Aerospike Connect for Spark.
---

## Part 1 : Read and Write DataFrames

Be sure to follow the [installation instructions](/docs/connectors/enterprise/spark/installation.html) and complete the installation of Aerospike Connect for Spark. In this section of the tutorial we will walk through reading data from an Aerospike database into a Spark DataFrame and writing a Spark DataFrame to an Aerospike database.

#### Set up imports

Launch the spark shell and import the `com.aerospike.spark.sql._` package

```scala
$ spark-shell
scala> import com.aerospike.spark.sql._
import com.aerospike.spark.sql._
```
and any Aerospike packages and classes. For example:
```scala
scala> import com.aerospike.client.AerospikeClient
import com.aerospike.client.AerospikeClient

scala> import com.aerospike.client.Bin
import com.aerospike.client.Bin

scala> import com.aerospike.client.Key
import com.aerospike.client.Key

scala> import com.aerospike.client.Value
import com.aerospike.client.Value
```

#### Load data into Aerospike

Load some data into Aerospike with:

```scala
    val TEST_COUNT = 100
    val namespace = "test"
    var client=AerospikeConnection.getClient(AerospikeConfig("localhost",3000,1000))
    Value.UseDoubleType = true
    for (i <- 1 to TEST_COUNT) {
      val key = new Key(namespace, "spark-test", "spark-test-"+i)
      client.put(null, key,
         new Bin("one", i),
         new Bin("two", "two:"+i),
         new Bin("three", i.toDouble)
      )
    }
```

Try a test with the loaded data:

```scala
    import org.apache.spark.sql.{ SQLContext, SparkSession, SaveMode}
    import org.apache.spark.SparkConf

    spark.conf.set("aerospike.seedhost", "localhost")
    spark.conf.set("aerospike.port",  "3000")
    spark.conf.set("aerospike.namespace", "test")
    
    val sqlContext = spark.sqlContext
    val thingsDF = sqlContext.read.
      format("com.aerospike.spark.sql").
      option("aerospike.set", "spark-test").
      load
      
    thingsDF.registerTempTable("things")
    val filteredThings = sqlContext.sql("select * from things where one = 55")
    val thing = filteredThings.first()
```

Note the configuration for the Aerospike namespace: ```spark.conf.set("aerospike.namespace", "test")```. A given spark context is tied to a single Aerospike namespace.

### Loading and Saving DataFrames 

Aerospike Connect for Spark provides functions to load data from Aerospike into a DataFrame and save a DataFrame into Aerospike

#### Loading data

```scala
    val thingsDF = sqlContext.read.
      format("com.aerospike.spark.sql").
      option("aerospike.set", "spark-test").
      load 
```

You can see that the read function is configured by a number of options, these are:
- `format("com.aerospike.spark.sql")` specifies the function library to load the DataFrame.
- `option("aerospike.set", "spark-test")` specifies the Set to be used e.g. "spark-test"
Spark SQL can be used to efficently filter (where lastName = 'Smith') Bin values represented as columns. The filter is passed down to the Aerospike cluster and filtering is done in the server. Here is an example using filtering:

```scala
    val thingsDF = sqlContext.read.
      format("com.aerospike.spark.sql").
      option("aerospike.set", "spark-test").
      load 
    thingsDF.registerTempTable("things")
    val filteredThings = sqlContext.sql("select * from things where one = 55")
```

Additional meta-data columns are automatically included when reading from Aerospike, the default names are:
- `__key` the values of the primary key if it is stored in Aerospike
- `__digest` the digest as Array[byte]
- `__generation` the gereration value of the record read
- `__expiration` the expiration epoch
- `__ttl` the time to live value calcualed from the expiration - now
 
These meta-data column name defaults can be be changed by using additional options during read or write, for example:
```scala
    val thingsDF = sqlContext.read.
      format("com.aerospike.spark.sql").
      option("aerospike.set", "spark-test").
      option("aerospike.expiryColumn", "_my_expiry_column").
      load 
```

#### Saving data

A DataFrame can be saved in Aerospike by specifying a column in the DataFrame as the Primary Key or the Digest.

##### Saving by Digest

In this example, the value of the digest is specified by the "__digest" column in the DataFrame.
```scala
    val thingsDF = sqlContext.read.
      format("com.aerospike.spark.sql").
      option("aerospike.set", "spark-test").
      load 
        
    thingsDF.write.
      mode(SaveMode.Overwrite).
      format("com.aerospike.spark.sql").
      option("aerospike.set", "spark-test").
      option("aerospike.updateByDigest", "__digest").
      save()                
```

##### Saving by Key

In this example, the value of the primary key is specified by the "key" column in the DataFrame.
```scala
      import org.apache.spark.sql.types.StructType
      import org.apache.spark.sql.types.StructField
      import org.apache.spark.sql.types.LongType
      import org.apache.spark.sql.types.StringType
      import org.apache.spark.sql.DataFrame
      import org.apache.spark.sql.Row
       
      val schema = new StructType(Array(
          StructField("key",StringType,nullable = false),
          StructField("last",StringType,nullable = true),
          StructField("first",StringType,nullable = true),
          StructField("when",LongType,nullable = true)
          )) 
      val rows = Seq(
          Row("Fraser_Malcolm","Fraser", "Malcolm", 1975L),
          Row("Hawke_Bob","Hawke", "Bob", 1983L),
          Row("Keating_Paul","Keating", "Paul", 1991L), 
          Row("Howard_John","Howard", "John", 1996L), 
          Row("Rudd_Kevin","Rudd", "Kevin", 2007L), 
          Row("Gillard_Julia","Gillard", "Julia", 2010L), 
          Row("Abbott_Tony","Abbott", "Tony", 2013L), 
          Row("Tunrbull_Malcom","Tunrbull", "Malcom", 2015L)
          )
          
      val inputRDD = sc.parallelize(rows)
      
      val newDF = sqlContext.createDataFrame(inputRDD, schema)
  
      newDF.write.
        mode(SaveMode.Ignore).
        format("com.aerospike.spark.sql").
        option("aerospike.set", "spark-test").
        option("aerospike.updateByKey", "key").
        save()       
```

##### Using TTL while saving

Time to live (TTL) can be set individually on each record. The TTL should be stored in a column in the DataSet before it is saved. 

To enable updates to TTL, and additional option is specified:

```scala
    option("aerospike.ttlColumn", "expiry")
```

### Schema

Aerospike is Schema-less and Spark DataFrames use a Schema. To facilitate the need for schema, Aerospike Connect for Spark samples 100 records via a scan then reads the Bin names and infers the Bin type.

The number of records scanned can be changed by using the option:

```scala
    option("aerospike.schema.scan", 20)
```
Note: the schema is derived each time `load` is called. If you call `load` before the Aerospike namespace/set has any data, only the meta-data columns will be available.

#### Continue to Part 2

Continue to [Part 2](/docs/connectors/enterprise/spark/tutorial_2.html) of this tutorial to learn how to perform join and intersect operations within Spark which will load matching records from an Aerospike database.

