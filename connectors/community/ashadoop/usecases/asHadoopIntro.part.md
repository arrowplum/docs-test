Apache Hadoop comprises two frameworks: Hadoop Distributed File System (HDFS) and Map-Reduce Computation.  It runs on a bank of servers (nodes) - practically as few as 4 to thousands. It is designed to reduce network transfer of large amounts of data making it an efficient choice for dealing with tera and petabytes of data.  

HDFS stores files with no limitations on size, in chunks of 64Mb or larger, replicated for fault tolerance on a minimum of 3 nodes. Since the data is chunked into blocks on different nodes, HDFS is a write-once, read multiple times system. Data cannot be updated but can be appended to. The Map-Reduce framework runs computations on this stored distributed data by bringing “computation to the data” (instead of the traditional model of bringing data to the computation engine) and then reducing or aggregating the results. Map-Reduce greatly reduces data transfer between nodes.

HDFS allows enterprises to store petabyte size files of data and extract useful information from this data using the map-reduce computational framework. This paradigm shift allows enterprises to capture any and all incoming data for later processing rather than the traditional model of selectively filtering data upfront that fits into pre-defined schemas and store on relational databases of limited capacity.

Due to the batch nature of the Map-Reduce framework, while it is much more efficient than the traditional model to obtaining BI from transactional relational data to data mart transformations, for certain web scale use cases, map-reduce is just too slow. 

To address the need for low latency, NoSQL databases such as Aerospike, have filled the niche for key-value store along with projects such as Apache Spark for in-memory real time computations.
Aerospike in the Hadoop Ecosystem
In all integrations with Hadoop, the underlying concept is to be able to enrich the operational data on Aerospike as well as provide the operational data from Aerospike to the Hadoop ecosystem for enterprise wide analytics - and in turn enrich the analytics data set with updates from operational data on Aerospike. In real time applications, Aerospike is also used as a results store, append incremental updates from machine learning algorithms running on enterprise wide data and providing operational data for models feeding real time web applications.


<center>
{{#figure "" "Aerospike in the Hadoop Ecosystem" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeInHadoop.png" >
{{/figure}}
</center>
        	
 
## Advantage of using Aerospike in a Hadoop Ecosystem 

Aerospike is a distributed database supporting key-value store and document-oriented data models. It supports sub-millisecond read-write latency even at petabyte scale, providing a distinct advantage as an operational cache in a hadoop ecosystem. Aerospike’s secondary indexing capability further helps in extracting higher performance.

## Typical Use Cases

- [Ad Targeting and Real-Time Bidding](/docs/connectors/community/ashadoop/usecases/asForRTB.html)
- [Fraud Detection](/docs/connectors/community/ashadoop/usecases/asForFraud.html) 
- [Content Personalization](/docs/connectors/community/ashadoop/usecases/asForContent.html)
- [Recommendation Engines](/docs/connectors/community/ashadoop/usecases/asForRecoEngine.html)
- [Customer Churn Analysis, Predictive Modelling and Risk Analysis](/docs/connectors/community/ashadoop/usecases/asForChurn.html)
- [Internet of Things](/docs/connectors/community/ashadoop/usecases/asForIOT.html)  

 
