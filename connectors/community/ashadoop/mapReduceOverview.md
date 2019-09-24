---
title: Map-Reduce Overview
description: Overview of Map-Reduce paradigm in the Apache Hadoop Ecosystem
---
MapReduce is the computational engine in Hadoop which brings computation to the data by splitting the task into two stages, Map and Reduce. During, the Map phase, records of input data are transformed into Key-Value pairs. After shuffling and sorting, this data is reduced in a subsequent reduce phase.


# Exploring Map-Reduce  
The figures below show how the map phase, partitioning or shuffle-and- sort, and the reduce phase work on a bag of coins to count quarters, dimes, nickels and pennies. In this example we use 3 mappers and 2 reducers.

We also enumerate key components of the map-reduce operation which relate directly to similar provisions in the Aerospike Hadoop Connector. 

<center>
{{#figure "" "Map-Reduce Overview" size="large"}}
<img src="/docs/connectors/assets/images/MapRedOverview.png" >
{{/figure}}
</center>

The input data, a bag of coins, is split three-ways, with one split 
handed over to each mapper. Each mapper works on its share of the 
data, generates intermediate data on the local file system which is 
then provided to the reducers. In our example, we have two reducers, 
each working on its assigned keys. Reducer #1 works on Quarters, 
Reducer #2 works on Dimes, Nickels and Pennies. 
Mappers produce sorted data in 
each partition, one partition per reducer. 
The Shuffle and Sort phase provides the reducers sorted data 
for their partition, assimilated from the intermediate output of each mapper, merged and sorted.
Reducers then aggregate the data to produce the final results.

In the next two figures, we dig deeper on how the intermediate data is handled between the mapper and the reducer.

<center>
{{#figure "" "Mapper Details" size="large" }}
<img src="/docs/connectors/assets/images/MapperDetails.png" >
{{/figure}}
</center>

A mapper takes its input split and reads one record at a time as defined by the `RecordReader`.
Data is read into a memory buffer where it is sorted by each partition corresponding to each reducer.
Within the partition, data is sorted by the keys of that partition (reducer). When the memory buffer is full,
this memory pass is spilled over to the disk. Depending on the size of the input data, multiple memory passes result in 
multiple spills to disk which are then merged and sorted. If a combiner is specified (optional), a mini-reduce is performed
on this intermediate data using the `OutputCollector` and thereby helping the reducer's workload. 
This intermediate output from each mapper becomes the input
to the shuffle and sort phase discussed below.

<center>
{{#figure "" "Shuffle, Sort and Reduce" size="large" }}
<img src="/docs/connectors/assets/images/ShuffleSortReduce.png" >
{{/figure}}
</center>

In the shuffle and sort phase, interdediate output from mappers is merged by partition. 
Data within the partition is sorted by keys for that partition. 
The partition file is then read by its associated reducer producing the final results per the `OutputCollector`.


#### Key Terms for Implementing  Map-Reduce 

These concepts are used to configure the map-reduce job in the [Job Driver](/docs/connectors/community/ashadoop/handson/examples/jobDriver.html).
 
*InputSplit:* Decide how to split the source data between the chosen number of mappers.  
*RecordReader:* Define what comprises a unit of data to be read and mapped into a key-value pair.  
*OutputCollector:* Defines the output format.  
*Reporter:* For monitoring map-reduce job progress.  
*Partitioner:* Decide, based on the number of reducers (equal to number of partitions), which partition operates on which set of output keys from the mapper.  
*Combiner:* Optional, runs on each mapper after map phase to do a “mini-reduction” before shuffling and sorting and providing the data to the reducer.  Depending on the nature of the data, may improve performance. 


