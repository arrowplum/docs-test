---
title: Aerospike Connect for JMS - Outbound Delivery Mode Configuration
description: Configuring Aerospike Connect for JMS Outbound Connector delivery-mode section
---

## Delivery mode

This configuration determines how the messages are delivered to the JMS message broker.

There are two modes supported.

* **persistent** - The default delivery mode, instructs JMS to take extra care to ensure that a message is not lost in transit in case of a JMS failure.
    A message sent with this delivery mode is logged to stable storage when it is sent.
* **non-persistent** - Does not require JMS to store the message or otherwise guarantee that it is not lost if there are JMS failures.

In most cases persistent delivery mode is slower than non-persistent delivery mode.

### Example

For persistent delivery mode:
```
delivery-mode: persistent
```

For non-persistent delivery mode:

```
delivery-mode: non-persistent
```
