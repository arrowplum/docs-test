---
title: Aerospike Community Connectors
description: Community-driven connectors for Aerospike.

---

These community driven connectors can be used with a wide variety of frameworks.

###Spring Data Connector

The Spring Data connector is an open source community project hosted under the Spring umbrella. It provides a data layer to Aerospike that is familiar to Spring developers. Spring Data for Aerospike works in harmony with the full Spring suite including Spring Boot and Spring MVC.

This connector supports the standard Spring Data features:
- Powerful repository and custom object-mapping abstractions
- Dynamic query derivation from repository method names
- Implementation domain base classes providing basic properties
- Support for transparent auditing (created, last changed)
- Possibility to integrate custom repository code
- Easy Spring integration via JavaConfig and custom XML namespaces
- Advanced integration with Spring MVC controllers

<a href="/docs/connectors/community/spring/" class="primary button">Learn more</a> 

###Hadoop Connector

The Hadoop connector shows examples of InputFormat and OutputFormat programming. The integration allows you to pull data directly from Hadoop clusters in order to analyze or check any form of unstructured data.

This basic but powerful tool provides a building block to integrate Aerospike with a huge number of Hadoop projects, such as Hive, Pig, Flume.

Besides connector code itself, [Aerospike's Hadoop GitHub repository](https://github.com/aerospike/aerospike-hadoop) includes classic examples like word count and how to sessionize web data.

When you integrate Aerospike's capabilities at key-value operations into your architecture, you'll achieve a Hadoop system which is capable of not just streaming analytics, but also computations that require random access. For example, imagine analyzing a customer dataset by a variety of audience segments, skipping easily through a large dataset to operate on only the data you need.

In all integrations with Hadoop, the underlying concept is to be able to enrich the operational data on Aerospike as well as provide the operational data from Aerospike to the Hadoop ecosystem for enterprise wide analytics - and in turn enrich the analytics data set with updates from operational data on Aerospike. In real time applications, Aerospike is also used as a results store, append incremental updates from machine learning algorithms running on enterprise wide data and providing operational data for models feeding real time web applications.

<a href="/docs/connectors/community/ashadoop/" class="primary button">Learn more</a>

##Launchpad

Our Launchpad features community projects that make use of Aerospike in
interesting ways, as well as projects seeded by Aerospike and moved over to the
community side for continued support and development.

<a href="/community/launchpad/" class="primary button">Visit Launchpad</a>
