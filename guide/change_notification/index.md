---
title: Change Notification Development Guide
assets: /docs/guide/assets
---

Change notification system will notify external systems when a record gets
modified. Refer to the
[architecture](/docs/architecture/change-notification.html) for a high-level
understanding of this feature. 

The change notification infrastructure builds on Aerospike's internal
synchronization mechanism called XDR (Cross Datacenter Replication). Many of the
configuration elements are the same as the current XDR system, and anyone
familiar with XDR will find the Notification system easy to use. Aerospike
continues to maintain the XDR system for cluster synchronization due to its
higher performance. Refer to the [XDR
architectural](/docs/architecture/xdr.html) documentation.

A Java library is provided which allows decoding of these HTTP messages.
Download `"Change Notification SDK"` package from the [Aerospike Enterprise
downloads](/enterprise/download/sdk/) section. The package demonstrates
all the key elements of a receiver. It can be used to transform the message
into a different data format as required by a downstream application or perform
complex event processing. A sample application is also provided to give an
end-to-end example to demonstrate how the events from the database writes can
be consumed by the receiver. The sample application prints JSON representations
of the messages sent by the Change Notification framework. It can be launched
using the included Jetty Runner, which is a stand-alone distribution of the
high-performance Jetty web server and `javax.servlet` container.

The rest of the guide is organized as follows:
* Make sure the [delivery guarantees](/docs/guide/change_notification/guarantees.html)
are acceptable by the application being developed. If necessary, the application
may need to be modified to fit within the guarantees.
* Choose the [protocol](/docs/guide/change_notification/protocol.html) based on the
development environment and performance requirements.
* Plan to build the [receiver](/docs/guide/change_notification/receiver.html) in sync
with the sender's requirements.
* [Configure](/docs/guide/change_notification/configure.html) Change Notification
Framework.
* Understand the [API](/docs/guide/change_notification/api.html) of the Java parsing library to consume the notifications.
* Try the [demo application](/docs/guide/change_notification/demo_app.html) to understand end-to-end flow of the notifications.


## Deployment Scenarios
When the receiver web application is ready, several factor can influence the deployment setup. Some of the factor to consider are:
1. Throughput : This probably plays the biggest role. The Aerospike server is very efficient in pushing out the updates. The receiver also should be capable of receiving at the same rate to have impedance matching. A single webserver may not be able to take the load at which the publisher sends out events. Definitely, multiple webservers should be used to be able to handle the load. There are two choices:
 1. Without Load Balancer (recommended) : Aersopike server as a publisher will load balance across multiple webservers. So, a traditional load balancer is not necessary.
 1. With Load Balancer : If you already have a set of webservers that are behind a load balancer, and you want to reuse those you may want to use this approach.
1. Local LAN vs WAN : The choice will primary be driven based on where the receiver will process the events. As the Change Notification framework is built on XDR, it can handle cross-geo publishing too. As with regular XDR, it will handle link down scenarios etc. The main factor influencing this choice will be latency tolerance. Obviously local LAN will have much smaller latencies compared to WAN links.
1. Choice of webserver : We do not have a preference for a particular webserver but we strongly recommend to use a webserver which supports HTTP v2 protocol. We can achieve very high throughput by using the multiplexing functionality of HTTP v2. Our demo application uses jetty web server as jetty is considered a fast web server.

