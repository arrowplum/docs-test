---
title: Integrating Aerospike and Apache Hadoop
description: Integration of Aerospike into the Apache Hadoop Ecosystem
---
Aerospike Hadoop Connector is the glue that connects Aerospike with the Hadoop framework. The data from Aerospike can be pulled into Hadoop and processed using map-reduce. Further, this processed data can be written back to Aerospike. The connector provides all the functionality to connect Aerospike and Hadoop together seamlessly. When reading from Aerospike it will automatically split the Aerospike data required for map-reduce. The user does not have to know how Aerospike works. But still the user can run map-reduce jobs using data from Aerospike. The architecture is simple and includes the classes required to connect with Aerospike.

# Map-Reduce Connector Architecture

<center>
{{#figure "" "Aerospike Hadoop Map-Reduce Connector" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeHadoopConnector.png">
{{/figure}}
</center>

## How does it work?

Aerospike has provided basic Java classes for writing map-reduce job. These classes include AerpspikeSplits, InputFormat, AerpospikeRecordReader, OutputFormat, AerospikeRecord and more. This Aerospike Hadoop connector provides basic functionality to pull data from Aerospike and write back to it.  Using these classes, custom map-reduce jobs can be implemented. AerospikeSplits splits input data pulled from Aerospike database nodes using bin numeric range, if required. These will be submitted to the Hadoop mapper, which will map key-value pairs. Reducer can write results to Aerospike database or hdfs as required.


## Features
*Input splits:* For map-reduce it is important to split input data into number of chunks, which would be processed by the distributed environment. This connector checks the number of Aerospike nodes and automatically creates one split per node. This way Aerospike is not over loaded while reading data.

*Singleton client:* It allows user to create a singleton client.

*Multi-Threaded Reader:* While reading from Aerospike, the connector launches as many threads as the number of nodes, creating one reader thread per node.

