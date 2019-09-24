---
title: Receiver
assets: /docs/guide/assets
---

The receiver which will be a web-server should handle two operations:
1. POST request
1. GET request

The records will be sent in a POST request to the receiving web server. The
format of the payload will be explained in the section below. On receiving the
post request, the web server should send HTTP-OK. If the sender (XDR) receives a
different response other than HTTP-OK, including timeouts, it will relog and
reship the record again until it gets a HTTP-OK. Failures or delays in response,
till the configurable timeout, will block all further sending of all endpoints
on this namespace, including all endpoints within the native XDR shipping
system, in the way specified in the current XDR system.

## Health
The receiver is also expected to handle a GET request. The receiver must reply
with an HTTP-OK message. This is sent every 5 seconds (with 1 sec timeout) as a
background health check. This is used to proactively remove malfunctioning
endpoints, and then to detect endpoints which become operational. This
lightweight GET call is preferred over a lengthy POST operation.

If a location does not respond to any health probes for 30 seconds, we declare
the location bad. The server will not ship records to an unhealthy location If
all the locations become bad, we declare that the whole destination is down. We
trigger a datacenter down notification which which will wait for the datacenter
to begin processing data again. Once at least one location becomes healthy, we
resume shipping.

## Payload
The picture below summarizes the payload at the HTTP layer.

{{#figure "" "Change Notification Payload" size="large"}}
![]({{book.assets}}/change_notification_payload.png)
{{/figure}}

As usual with any HTTP request, there are some common headers which are necessary for client-server communication. Other than that, we set the following headers:
* "Content-type" : It is set as “application/octet-stream” as the  content of
the POST fields is a binary format.
* "Content-length" : It is an integer value equal to the total number of bytes
in the POST fields.

The data in the POST body will be a binary message in the same format currently
used by XDR, which we will explain in-depth in later sections. We decided to
reuse the aerospike client-server wire protocol itself to represent the record
and the operation (update/delete). The existing open source client
implementations in various programming languages knows how to handle this
format.

The first byte represents the binary message version described below. The
following information regarding static record forms will be version 1. Other
versions may be later designed with alternate data formats including shipping of
operations and deltas. The parsing library validates the version number, and
respond with HTTP error 501 (NOT SUPPORTED) for any message with a version
that is not supported by this endpoint.

