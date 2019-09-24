---
title: Aerospike Connect for Spark Tutorial
description: Learn how to deploy and configure Aerospike Connect for Spark.
---

## Part 2 : Join, Intersect and Increment Operations

The load operations we described in [Part 1](/docs/connectors/enterprise/spark/tutorial_1.html) of our tutorial can be used to load the entire contents of an Aerospike set into a Spark DataFrame.

There may be times when you have a large dataset stored in Aerospike and you would only like to load the records for a limited number of keys which are already in a Spark Dataset. An example of this could be a stream processing system which collects userIDs of "active" users over a period of time and then loads the details for those users from Aerospike in order to facilitate processing the user data. In this case, you may not want to load all the users from the Aerospike database as the memory required to do so could overwhelm the Spark cluster. This is where the ```aeroJoin``` function comes into play.

#### aeroJoin

The ```aeroJoin[A]``` function extends the Dataset class to allow the user to load data from an Aeropsike database using batch read calls. The ```aeroJoin[A]``` function accepts the column name of the primary key and the set name. The properties from the ```A``` class determine which bins will be loaded from the Aerospike database. It returns a typed ```Dataset[A]``` where ```A``` is a user defined class:

```scala
aeroJoin[A](keyCol: String, set: String): Dataset[A]
```

Let's walk through an example of using the ```aeroJoin``` function. First we will save some sample data in the Aerospike DB:

```scala
	import com.aerospike.spark._

	case class Person(name: String, age: Long, sex: String, ssn: String)
	val people = Seq(
	    Person("jimmy", 20, "M", "555-22-3333"),
	    Person("johnny", 32, "M", "555-33-2222"),
	    Person("linda", 28, "F", "555-23-2323")).toDS().save("people", "ssn")
```

Our data model for a ```Person``` consists of a name, age, sex and primary key: ssn. Now build up a Dataset with a couple of ssn keys.

```scala
	case class PersonSSN(ssn: String)
	val peopleSSN = Seq(
	      PersonSSN("555-22-3333"),
	      PersonSSN("555-33-2222")
	    ).toDS()
```

Finally, perform the ```aeroJoin``` operation to load the records

```scala
	val peopleDS = peopleSSN.aeroJoin[Person]("ssn", "people")
```

```peopleDS``` will be a Dataset of ```Person``` objects with all the fields populated from the Aerospike database.

You can also use the Spark ```join``` function to combine the two datasets. This would be useful if you had additional data in the ```peopleSSN``` Dataset which you wanted to combine with the data from the Aerospike database:

```scala
	peopleCombined = peopleSSN.join(peopleDS, "ssn")
```

Here is an entire sample scala application which uses the aeroJoin function:

```scala
package com.aerospike.spark

import com.typesafe.scalalogging.slf4j.LazyLogging
import org.apache.spark.SparkConf
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}
import org.apache.spark.sql.{Row, SaveMode, SparkSession}

/**
 * This example will load some data into a "Business" namespace in an aerospike database.
 * It will then use aeroJoin to take a sequence of ids and load the appropriate customer data, filter it, and print out the result.
 *
 * Prereqs: A working Aerospike Connect for Spark and an Aerospike server running on default port on localhost with at least 1 namespace named "Business"
 */
object aeroJoinExample extends LazyLogging with Serializable {
  val conf: SparkConf = new SparkConf()
    .setAppName("AeroJoin")
    .set("aerospike.seedhost", "localhost")
    .set("aerospike.port", "3000")
    .set("aerospike.namespace", "Business")

  val session: SparkSession = SparkSession.builder()
    .config(conf)
    .master("local[*]")
    .appName("Aerospike Example(II)")
    .config("spark.ui.enabled", "false")
    .getOrCreate()

  val schema: StructType = new StructType(Array(
    StructField("key", StringType, nullable = true),
    StructField("customer_id", StringType, nullable = false),
    StructField("last", StringType, nullable = true),
    StructField("first", StringType, nullable = true),
    StructField("stars", IntegerType, nullable = true)
  ))

  def main(args: Array[String]) {
    import session.implicits._

    loadCustomerData()

    val ids = Seq("customer1", "customer2",
      "customer3", "customer4", "IDontExist")

    val customerIdsDF = ids.toDF("customer_id").as[CustomerID]

    val customerDS = customerIdsDF.aeroJoin[CustomerKV]("customer_id", "Customers")
    customerDS.foreach(b => println(b))

    val bestCustomers = customerDS.filter(customer => customer.stars > 4)
    bestCustomers.foreach(b => println(b))

    bestCustomers.map(c => new Customer(c.key, c.customer_id, c.first, c.last, c.stars)).toDF("key", "customer_id", "last", "first", "stars").
      write.mode(SaveMode.Overwrite).
      format("com.aerospike.spark.sql").
      option("aerospike.updateByKey", "customer_id").
      option("aerospike.set", "BestCustomers").
      save()
  }

  def loadCustomerData(): Unit = {
    val rows = Seq(
      Row("Fraser_Malcolm", "customer1", "Fraser", "Malcolm", 5),
      Row("Hawke_Bob", "customer2", "Hawke", "Bob", 4),
      Row("Keating_Paul", "customer3", "Keating", "Paul", 1),
      Row("Im_Nothere", "secretcustomer", "Nothere", "Im", 5),
      Row("Howard_John", "customer4", "Howard", "John", 1)
    )

    val customerRDD = session.sparkContext.parallelize(rows)
    val customerDF = session.createDataFrame(customerRDD, schema)

    customerDF.write.
      mode(SaveMode.Overwrite).
      format("com.aerospike.spark.sql").
      option("aerospike.updateByKey", "customer_id").
      option("aerospike.set", "Customers").
      save()

  }
}

case class CustomerID(customer_id: String)

case class Customer(key: String, customer_id: String, first: String, last: String, stars: Long)

case class CustomerKV(__key: Any, key: String, customer_id: String, first: String, last: String, stars: Long) extends AeroKV

```


#### aeroIntersect

The ```aeroIntersect``` function extends the Dataset class to allow the user to filter records from the Dataset which do not exist in the Aerospike database. The user must specify the column in the Dataset that contains the primary key of the records and the name of the Aerospike set in which to perform the intersect. The ```aeroIntersect``` call will return a Dataset which is a subset of the initial Dataset.

Using the same data model as the ```aeroJoin``` example, perform the ```aeroIntersect``` operation:

```scala
	val ds = Seq(
	      PersonSSN("555-22-3333"),
	      PersonSSN("555-22-4444")).toDS()

	val filtered = ds.aeroIntersect("ssn", "people")
```

The resulting ```filtered``` Dataset will contain only the ```PersonSSN``` objects which have a corresponding record in the Aerospike database.


#### aeroIncrease

The```aeroIncrease``` function enables dataset to increase some value (optional with default of 1) on a bin of Aerospike set.  The bin type can be either long/int or map.  Use will need to specify the field in dataset that holds Aerospike record key in which to perform this operation.  As the operation will be performed on Aerospike server side which would be atomic guaranteed by Aerospike database.

Example:

```scala
    val ds = Seq(
	      PersonSSN("555-22-3333"),
	      PersonSSN("555-22-4444")).toDS()

    //add 5 to "PTO" bin of set "employee"
    ds.aeroIncrease("ssn", "employee", "PTO", None, 5)

    //add 1 to the value of the map bin "TimeOff" with key "WFH"
    ds.aeroIncrease("ssn", "employee", "TimeOff", Option("WFH")) 
```

#### Continue to Part 3

In [Part 3](/docs/connectors/enterprise/spark/tutorial_3.html) of this tutorial we will look at ways to tune the performance of Spark with Aerospike.

