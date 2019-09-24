### Fraud Detection 
Consider a financial institution offering online banking to its hundreds of millions of users, thousands of branch offices across all major cities around the world spread across all continents and over 50 countries.  Every transaction across millions of credit card holders has to be screened in real time using reliable fraud detection models where false positives can be very detrimental to business. The models must be continually updatable as new fraud patterns are discovered and provide a reliable fraud score so business rules can be effectively employed to catch a majority of the fraudulent payments before they are adjudicated.

#### Defining Fraud

Detecting fraud is identifying a transaction associated with a user as an outlier because it does not fit the user’s normal behavior. In case of online transactions, the detection must be in real-time to carry business value.

Fraud is also defined as an over-payment or payment made towards a false claim applied to any industry making payments - be it credit card, health care claims, auto insurance claims, reimbursement from government programs for services rendered, refunds of tax payments, underpayment of taxes through fraudulent returns etc. 

Traditionally, transactions were stored on an operational data store and fraud was detected post-adjudication of the claim by analysing the transaction data at a later point in time. With cost of collection, significance threshold of the amount involved, along with the probability of successful collection, post-adjudication fraud detection did not provide much business value other than being a weak deterrent and costing businesses significant hit at their bottom lines. 

Running SQL queries on the relational data store improves the probability of detecting fraud pre-adjudication but cannot keep up with changing fraud patterns, limitations by the data schema on what is possible to query and the additional overhead of running computations on the storage layer. Algorithms written in Java, using Key Value databases as low-latency shared caches, utilizing latest machine learning libraries are replacing the traditional SQL approach in pre-adjudication fraud detection.

#### Pre-Adjudication Fraud Detection

Pre-adjudication fraud detection drives cost of collections and recovery to zero. It must however be quantified and balanced against an “irate consumer” penalty in terms of consumer retention and time spent in addressing appeal challenges to claim denials when the algorithm flags false positives.  False positives in a real time fraud detection system can be a significant drawback and must be minimized.

Pre-adjudication fraud detection can further be separated into two distinct categories - transaction fraud detection or “real-time” fraud detection which has a maximum time to detect window of 150ms, and, claims fraud detection where consumer files a claim but adjudication time window is fairly wide - often hours, days to weeks.  Clearly real-time fraud detection methodologies can also be applied to claims fraud detection but not vice-versa, always.

#### Claims Fraud Detection 
Claim processing time reduction and significant cost savings are achieved by a payment processing vendor by allowing its clients to submit claims electronically which are approved using pattern matching algorithms combined with social network analysis.

#### Real Time Fraud Detection 
Typical fraud detection scenarios involve - Transaction level fraud detection, Suspicious Account activity detection using payment frequency and profile changes, Inter-Account Suspicious Activity detection. Latency of computational algorithms accessing various data sources is critical due to temporal, geographical and user profile correlation on tens of thousands of attributes. Employing a fast and scalable database like Aerospike for attribute lookup for the machine learning algorithms as part of the Hadoop based architecture enables achieving strict SLAs.

Real-time fraud detection requires a low-latency, scalable data store with extremely low latencies which has user specific attributes associated with a userid, ingests user transaction attributes, has attributes to match patterns, attributes specific to a userid that help identify an outlier in context of a specific user and is able to run real time analysis on each incoming transaction.

A key value datastore like Aerospike is an ideal solution for storing user attributes, pattern attributes, stream ingesting transaction attributes specific to a userid and running a user-defined function to identify an outlier or serving as a cache for another real time analytics solution such as Apache Spark or Apache Flink. 

Computation of the fraud score typically involves multiple queries into the key value store. A transaction originates with an ip address, has a specific userid tied to it, with a shipping address and a merchant id. The fraud detection algorithm not only analyses this transaction in isolation starting with basic user authenticity validation but then queries the key value store for past transaction from the same ip address, checks for past pattern of stored fraud scores, looks at patterns on the beneficiary of the transaction, eg the shipping address and query its past transactions and their fraud score etc. Finally, for this transaction it must write back its fraud score to the key value store for future use, before presenting the final fraud score to the rules engine.

The figure below suggests a fraud detection ecosystem. Architecturally, we separate the compute layer from petabyte key value storage layer.


<center>
{{#figure "" "Aerospike in Fraud Detection" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeInFraudDetection.png" >
{{/figure}}
</center>


#### Standalone Transaction Level R/T Fraud Detection Implementation 

After Identity validation, scoring algorithms perform pattern matching against user attributes and transaction attributes, explore relationships of entities involved in the transaction in conjunction with their historical data, resulting in a “fraud score”. Business rules decide action to be taken based on the score. 

#### Quantifying the fraud detection problem SLAs

Checkout counters, whether in-store clerks or online carts, cannot be held up for real time fraud detection for each transaction for more than 50 to 150ms.

Typical web scale fraud detection storage layer requirements:
10 to 50 billion records.
300 TB of unique data before replications.
2 million transactions per second.
50 ms fraud detection time.
Pre-adjudication analysis must catch 80% of the fraudulent claims.
Local and domain replication for high availability.

This kind of workload can be satisfied with Multi-level Cell (MLC) flash instead of petabytes of RAM which can be prohibitively expensive. Aerospike is optimized to run on SSDs and becomes a compelling key value store choice.

#### Importance of Relationships in Cross Transaction Analysis
Increasingly fraud is not perpetrated by a single individual but a network of individuals working together.  Whether it is credit card fraud or medicare billing fraud between network of participating doctors referring each other and billing without even seeing the patient, detecting network of fraudulent relationships is becoming an increasingly important tool in fraud detection.

The algorithm must be able to identify common relationships in high fraud score transactions. For example, fraudulent payments through different transactions going to same or related bank accounts, progression of activities of common denominators through time such as deposit activities over time in suspected bank accounts, etc.

Effective Implementation of real-time fraud detection therefore results in multiple queries to the storage layer once a fraud pattern matches to an incoming transaction to discover other relationships and a possible network of fraud.  While a first transaction may be allowed to go through because the score indicated a “may be”, with a second transaction, fraud score could go the “confirmed” level after not only pattern matching but analysis of the connected entities in the associated network of these transactions.

#### SQL or NoSQL?
Switching from SQL to NoSQL gains time to market. Fraud patterns can be updated a lot faster and implemented in the fraud detection pattern matching code if they use a NoSQL database. Patterns, such as user spending patterns, geo-location (via IP address) etc., have to be continually updated using data in a sliding time window as ever evolving new patterns are discovered in the 2 to 4 billion ip addresses, their recent behavior and their risk model. Modifications and updates to actionable algorithms can be put in-play faster in a Java based implementation on the compute layer with a separate NoSQL storage layer instead of the traditional combined compute and storage layers running SQL queries.

Separating the compute layer and writing algorithms in Java, moving from SQL to NoSQL,  enables usage of cutting edge machine learning libraries that have been shown to work.  Furthermore, licensing and maintenance cost of using traditional relational data stores for doing everything in SQL is exorbitant.

Traditional RDBMS based fraud detection solutions at web scale lack:  
*Agility:* ability to change matching patterns easily, update scoring models  
*Scalability:* ability to scale to 100’s of TB of data while maintaining throughput  
*TCO* - total cost of ownership including licensing costs  
*Algorithms* - ability to leverage latest developments in machine learning algorithms  
*High Availability* - failover and high availability across geographic zones with cross data-center replication of the storage layer across the globe in near real-time.  
*Social Networks:* adding relationships based algorithms on top of pattern matching  
