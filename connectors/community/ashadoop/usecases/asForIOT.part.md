### Internet of Things

With “things” getting their own ip addresses, with a local controller, having a ‘sensor’ that produces data and an ‘actuator’ that consumes data, an internet of things is evolving. In contrast to things talking to each other and taking autonomous decisions in true spirit of the “internet of things” (IOT), an alternative implementation involving “things on the internet” (TOI) augmented by a real time central controller system interacting with a family of things on the internet is finding numerous and exciting applications.  Hybrid models involving IOT and TOI are also being contemplated for networks that can get isolated during brief periods of time.

Here are some examples of applications involving a real time controller reading data from, and, sending control signals to, a network of things. 
- Telecom traffic monitoring and control
- Power Generation - consumption monitoring and production control
- Aviation systems monitoring for preventive maintenance
- Traffic monitoring, alerts and real time traffic re-routing
- Autonomous cars monitoring and control
- Oil and gas production monitoring, control and preventive maintenance
- Automated weather station and geo-activity data collection and alert generation

Data being generated in real time needs a buffer before it is stored in a persistent store because data generation is bursty in nature.  While queueing systems can provide smoothing of the flow rate, at internet scale, Apache Kafka has emerged as the distributed data log store and message broker of choice.

Apache Kafka implements a distributed log of machine data and serves it in a publisher-subscriber of “topics” model. It was designed to handle 175 TB of inflight data with replication and consistency, 1.5ms latency, serving tens of thousands of producers and thousands of consumers with ability to handle 7 million writes per sec and 35 million reads per second. It has a tight Hadoop integration for storing all data. Consumers can subscribe to specific topics of interest. 

At these data rates, in a multi-producer (sensors and devices) and consumer (controllers needing databases) implementation, as shown below, Aerospike is an appropriate low latency key value store as a consumer to off load data from an Apache Kafka message broker and distributed queue, which has a finite capacity.


<center>
{{#figure "" "Aerospike in Internet Of Things" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeInIOT.png" >
{{/figure}}
</center>

