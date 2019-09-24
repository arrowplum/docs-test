---
title: Event-base Models
description: 
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

The Aerospike libevent client API requires the application to specify the event base to return the transaction-completing callback.

The Aerospike libevent library is a convenient high-performance platform for running event-based applications. The application sets up one or more event bases, which dispatch events on application request. Events are _callbacks_. Callbacks include socket writable and readable callbacks, timers, and signals. An event base runs an event loop in a dedicated thread, which makes the callbacks. The application then does all its work using callbacks.

### Database Transactions

The libevent model enables asynchronous, non-blocking network transactions. The Aerospike libevent client allows libevent-based applications to initiate asynchronous, non-blocking writes and reads to the Aerospike database.

To initiate a database transaction, the application makes an Aerospike libevent client API call, which is non-blocking, and the client makes a callback to complete the transaction. When making the initiating call, the application specifies the event base for the callback.


Callbacks must be made on the same event base (thread) as the non-blocking call. This typically helps performance, as it avoids mutex locks  used when callbacks are made on a different thread from the initiating call. Though the application can specify a different callback event base from its initiating call, this 'cross-threaded' mode is not supported.

### Managing the Cluster

The Aerospike libevent client performs cluster management tasks within the libevent framework. Applications can specify the event base for cluster management, or an let the client internally create a separate event base and thread for cluster management. Applications can be completely single threaded by specifying that cluster management use the same event base as database transactions. 
