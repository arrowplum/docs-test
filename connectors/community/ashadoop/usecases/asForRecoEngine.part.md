### Recommendation Engines

Recommendation Engines are applicable to many verticals. Updated data from web applications and algorithms writing to Aerospike is pulled into HDFS, combined with data pulled from enterprise transaction systems into Hadoop and further combined with unstructured data from various sources already coming into HDFS. By running chained map-reduce jobs, updated and enhanced data for the web application is pushed back on to Aerospike for low latency key-value lookup and recommending at web application level in “real-time”. 


<center>
{{#figure "" "Using Aerospike in Recommendation Engine" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeInRecoEngine.png" >
{{/figure}}
</center>


Aerospike key value store is ideal for building co-occurrence matrix over time by customer id and finding item based recommendations for next transactions and displaying them to the online shopper in real time, much like RTB for display advertising. “Other customers who bought this item, also bought …”. Each item will have its own column or bin and each record will be maintained by unique customer id to aggregate unique customer’s purchase history (history matrix) and use it to compute inter-item purchase affinities or the co-occurrence matrix and store it back on Aerospike. 

This concept of item based recommender can be further enhanced by building co-relations between items and events. Events may be specific to the customer based on social profile,  specific to a geo-location inferred from the customer IP address or any other factor relevant to the use case.

Complex models quantify user persona, context and social data. For example numeric score of  sentiment analysis done on reviews or comments posted by a customer are added to the user profile against the relevant item. This allows adding weightage to the otherwise unweighted purchase data.

Finally, of all possible candidates that may be recommended, a relevance score for each item is calculated to decide the ones that must be recommended and stored in the relevance matrix.  This data is essentially key-value data, constantly growing and computed upon, requiring millisecond level latencies. Aerospike is an ideal choice for storing and updating recommendation engine operational data from history matrix, co-occurrence matrix to the relevance matrix.


<center>
{{#figure "" "Recommendation Engine" size="large" }}
<img src="/docs/connectors/assets/images/RecoEngine.png" >
{{/figure}}
</center>

