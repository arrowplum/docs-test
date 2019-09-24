---
title: Your First Aerospike Programs - Java 
description: A guide to writing two Aerospike programs, which both operate on multiple records
assets: /docs/guide/assets
---

This guide provides a guide towards writing your first two Aerospike programs. The first program will use an array to populate a desired number of records in an Aerospike DB. 
The second program will randomly read multiple records from the Aerospike DB.

The prerequisite for using these programs is an Aerospike server accessible on your network.

## First Program - writing records

### Importing Packages

We must import the appropryiate packages from the Aerospike Java libraries and some Java utilities:

```java
import com.aerospike.client.AerospikeClient;
import com.aerospike.client.policy.WritePolicy;
import com.aerospike.client.Bin;
import com.aerospike.client.Key;
import com.aerospike.client.Record;
import com.aerospike.client.Value;

import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;
```

### Source code

Then it's on to some actual code. We define the Main function and then read two (optional) command line arguments. Those arguments will provide an initial primary key value and a count of how many records with consecutive keys we want to insert into the DB. We have established some defaults for those arguments.

```java
    public static void main(String[] args) {
        // Default is to write 100,000 records
        int lowKeyVal = 0;
        int numKeys = 100000;
        
        if (args.length > 2 || args.length == 1) {
            System.err.println("Invalid number of arguments");
            System.exit(1);  
        } else if (args.length == 2)
            try {
               lowKeyVal = Integer.parseInt(args[0]);
               numKeys = Integer.parseInt(args[1]);
            } catch (NumberFormatException e) {
                System.err.println("Both arguments must be integers");
                System.exit(1);
            
            }
```

The next step is to connect to an Aerospike server. The hostname or IP address of any server in an Aerospike cluster (group of servers) will do:

```java
       AerospikeClient client = new AerospikeClient("192.168.1.25", 3000);
```

Let's define a variable which will store a random integer to be used as an integer bin (field) in our record. 
We'll also define the Key object which will be used to provide the key value of the records that we add to the DB:

```java
        int randomInt;
        Key key = null;
```

Now it's time for the heart of our program. Save the start time of our loop. 
Then execute the loop to write the desired number of records into the DB.

```java
        long startTime = System.currentTimeMillis();
        for (int i = lowKeyVal; i < lowKeyVal + numKeys; i++) {
```

For each iteration of our loop:
1. Instantiate a Key object with the key n sequence starting from the first requested key through the count of requested keys. You will probably want to change the name of the namespace ("test") to match the namespace in your Aerospike cluster. The string "demo" represents the set (table) name.
2. Generate a random number for the numeric bin value within the range of 1 to 100,000.
3. Create a numeric Bin object containing that value.
4. Create a string Bin obect containing that value converted to a string.
5. Write that record to the Aerospike DB, using a default WritePolicy.

```java
            key = new Key("test", "demo", i);
            randomInt = ThreadLocalRandom.current().nextInt(1, 100000);
            Bin int_bin = new Bin("intbin", randomInt);
            Bin str_bin = new Bin("strbin", String.valueOf(randomInt));
            client.put(new WritePolicy(), key, int_bin, str_bin);
        }
```

Now, to complete the program, we need to calculate the average number of records added per second and properly close the Aerospike connection.

### Executing and monitoring

Before the program exits, it will print the number of records added per second. There are two simple ways to monitor the write operations from the server side.

One option is to execute the [asloglatency](/docs/tools/asloglatency/) utility on a server in the cluster. An example of the command the output is as follows. The output from the command is a sequence of 10 second duration samples for "write" operations  executed on that server with percentages of those operations completing in greater than 1, 8, and 64 ms. Finally, the last column displays the number of those operations performed per second. At the bottom, average and maximum statistics are displayed.

```bash
asloglatency -N test -h write -f "Feb 06 2019 21:38:24"
Histogram : {test}-write 
Log       : /var/log/aerospike/aerospike.log
From      : 2019-02-06 21:38:24

Feb 06 2019 21:38:24
               % > (ms)
slice-to (sec)      1      8     64    ops/sec
-------------- ------ ------ ------ ----------
21:38:34    10   0.00   0.00   0.00      305.0
21:38:44    10   0.00   0.00   0.00      315.5
21:38:54    10   0.00   0.00   0.00      315.8
21:39:04    10   0.00   0.00   0.00      308.7
21:39:14    10   0.03   0.00   0.00      330.6
21:39:24    10   0.00   0.00   0.00      331.2
21:39:34    10   0.03   0.00   0.00      325.4
21:39:44    10   0.03   0.00   0.00      321.9
21:39:54    10   0.00   0.00   0.00      324.4
-------------- ------ ------ ------ ----------
     avg         0.01   0.00   0.00      319.0
     max         0.03   0.00   0.00      331.2
```

Another approach for those those that prefer to view graphs is to use the [Aerospike Management Console Utility](/docs/amc). Data regarding the number of writes per second in the entire cluster are displayed both in graphical form and in text.

![alt text](/docs/client/java/assets/AMC_screenshot.png "Aerospike Management Console")

## Second Program - reading records

### Source code
The setup for the program to read records from an Aerospike DB is similar to that of the program to write records:

```java
mport com.aerospike.client.AerospikeClient;
import com.aerospike.client.policy.WritePolicy;
import com.aerospike.client.policy.Policy;
import com.aerospike.client.Bin;
import com.aerospike.client.Key;
import com.aerospike.client.Record;
import com.aerospike.client.Value;

import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;

public class SecondApp {

    public static void main(String[] args) {
```

We're defining three command line arguments with default values for the low key value and the range of key values as well as the number of random records that we'd like to read within that range.

```java
        int lowKeyVal = 0;
        int numKeys = 100000;
        int numReads = 100000;
        
        if (args.length > 0 && args.length != 3) {
            System.err.println("Invalid number of arguments");
            System.exit(1);  
        } else if (args.length == 3) 
            try {
               lowKeyVal = Integer.parseInt(args[0]);
               numKeys = Integer.parseInt(args[1]);
               numReads = Integer.parseInt(args[2]);
            } catch (NumberFormatException e) {
                System.err.println("All arguments must be integers");
                System.exit(1);
            }
```

We then establish our connection to the Aerospike server. You will probably want to change the IP address to the IP address or hostname of any server in your Aerospike cluster:

```java
       AerospikeClient client = new AerospikeClient("192.168.1.25", 3000);
```

Now it's time for the heart of our program. Save the start time of our loop. 
Then execute the loop to write the desired number of random records from the Aerospke DB:

```java
        long startTime = System.currentTimeMillis();
	for (int i = 0; i < numReads; i++) {
```

For each iteration of our loop:
1. Generate a random key value within the requested range of keys.
2. Instantiate a Key object using that random key value against the set (table) "demo" in the namespace "test". You will probably want to change the name of the namespace ("test") to match the namespace in your Aerospike cluster. The string "demo" represents the set (table) name.
3. Issue the statement to read the entire record. If no record were found with the given key, the return value would be "null".

The statement to print out the two bins (values) in the retrieved record is commented out. If you uncomment that statement, you will have to add a check for a null returned Record. Otherwise, your program will terminate due to an exception when a record isn't found.

```java
            randomInt = ThreadLocalRandom.current().nextInt(lowKeyVal, lowKeyVal + numKeys);
            key = new Key("test", "demo", randomInt);
            Record record = client.get(new Policy(), key);
	    //System.out.println("int:" + record.getInt("intbin") + " string: " + record.getString("strbin"));
        }
```

We're done with our loop now. Capture the current time and print the calculated number of reads/second. Then close the connection to Aerospike:

```java
        long endTime = System.currentTimeMillis();
        System.out.println("records read/sec: " + numReads/((endTime - startTime)/1000));
        client.close();
    }
}
```

### Executing and monitoring

As detailed in the section covering the program to write records to the DB, you can use either [asloglatency](/docs/tools/asloglatency/) or the [Aerospike Management Console Utility](/docs/amc) to monitor the activity on the Aerospike server and cluster.

## Caveats and Moving Forward

Don't use the throughput numbers shown in this example as a baseline for your expected performance. The most common limitations on server throughput are due to either the single-threaded nature of the test program and/or the network throughput of the client. The throughput numbers shown above were obtained in a tests with a single client program connecting to a single Aerospike server via WIFI. Much greater throughput values can be obtained by utilizing multiple copies (and/or threads) of the client software executing on multiple clients on a wired network. 

A suggested approach to evaluation throughput/latency limitations on an Aerospike cluster is to take advantage of the [Java Benchmark Utility](/docs/client/java/benchmarks.html), included in the Java Client Library download. Utilizing that utility has often been a shortcut towards evaluating Aerospike throughput/latency against desired production loads without the need to write any programs. The caveats regarding the number of clients and the network connectivity will, of course, pertain to executions the of [Java Benchmark Utility](/docs/client/java/benchmarks.html). Often, customers think that they have reached the limits of throughput of their Aerospike cluster, when in reality they're reached limits of the load that the clients and network can produce against the Aerospike server. 
