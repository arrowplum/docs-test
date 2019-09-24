---
title: Map-Reduce Job Driver
description: Map-Reduce Job Driver Configuration
---

All the examples will be launched using the Driver segment of the code 
that submits the job to Hadoop. A typical driver configuration is shown below. 
Refer to example source code for specific implementations.

### Configuring a Map-Reduce job Driver


```java
log.info("run starting");

final Configuration conf = getConf(); //create config instance

JobConf job = new JobConf(conf, WordCountInput.class); //create job

job.setJobName("AerospikeWordCountInput"); //set name for m/r job

job.setInputFormat(AerospikeInputFormat.class); //set input format for map phase

job.setMapperClass(Map.class); //How mapping will happen - provide class name

job.setCombinerClass(Reduce.class); //You can write special combiner class or use reducer class

job.setReducerClass(Reduce.class); //set class for reducer phase


job.setOutputKeyClass(Text.class);  //How output needs to written - provide class

job.setOutputValueClass(IntWritable.class); //What value needs to be written to output

job.setOutputFormat(TextOutputFormat.class); //What is the output format - provide class

FileOutputFormat.setOutputPath(job, new Path(args[0])); //Where output needs to be written - provide path

JobClient.runJob(job); //running the map-reduce job

log.info("finished");
```
