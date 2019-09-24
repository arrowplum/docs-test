---
title: Getting Started
description: Step by step hands on to run Aerospike Connector Examples
styles:
  - /assets/styles/ui/steps.css
---

In the following sections, we describe in detail how to use the Aerospike Map-Reduce Connector.

{{#steps}}
{{!--------
  - STEP - INSTALL APACHE HADOOP CLUSTER
----------}}
{{#steps-step 1 "Install Multi-Node Apache Hadoop Cluster" markdown=true}}

We will start by installing a 3 node standalone Apache Hadoop cluster.

{{steps-action "Install Apache Hadoop Cluster" "/docs/connectors/community/ashadoop/handson/hadoopClusterSetup.html"}}

{{/steps-step}}

{{!--------
  - STEP - INSTALL EDGE NODE
----------}}
{{#steps-step 2 "Install Apache Hadoop Edge Node" markdown=true}}

We will then install a 4th Edge Node. 
It will be configured to access the Apache Hadoop
cluster and run map-reduce jobs on it. 

{{steps-action "Install Hadoop Edge Node" "/docs/connectors/community/ashadoop/handson/edgeNodeSetup.html"}}

{{/steps-step}}


{{!--------
  - STEP - INSTALL AEROSPIKE SERVER
----------}}
{{#steps-step 3 "Install Aerospike Server" markdown=true}}

To minimize the number of nodes deployed, we will also install the Aerospike server
on the Edge Node, though in production, Aerospike should be its own cluster.

{{steps-action "Install Aerospike Server" "/docs/connectors/community/ashadoop/handson/asdSetup.html"}}

{{/steps-step}}


{{!--------
  - STEP - INSTALL AEROSPIKE HADOOP CONNECTOR
----------}}
{{#steps-step 4 "Install Aerospike Hadoop Connector" markdown=true}}

On the 4th Edge Node, we will also install the Aerospike Hadoop Connector.  

{{steps-action "Install Aerospike Hadoop Connector" "/docs/connectors/community/ashadoop/handson/asHdConnSetup.html "}}

{{/steps-step}}


{{!--------
  - STEP - SETUP HADOOP CLUSTER
----------}}
{{#steps-step 5 "Users on Hadoop Cluster and Edge Node" markdown=true}}

As part of the installation and configuration of the Apache Hadoop Cluster and the Edge Node, we will add two users - *hduser* and *hdclient*. *hduser* will be the hadoop administrator with sudo privileges. 
*hdclient* will be a typical hadoop developer who will access the Hadoop Cluster from the Edge Node.

{{/steps-step}}


{{!--------
  - STEP - EXAMPLES
----------}}
{{#steps-step 6 "Hands-on Examples" markdown=true}}

We will demonstrate the examples discussed in the Examples Overview section 
using the Aerospike Map-Reduce Hadoop Connector.  

{{steps-action "Examples" "/docs/connectors/community/ashadoop/handson/examples.html"}}
{{/steps-step}}

{{/steps}}
