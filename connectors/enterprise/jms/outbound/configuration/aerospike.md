---
title: Configure Aerospike Server for JMS - Outbound
description: Configure Aerospike Server for JMS - Outbound
---

## Aerospike Database setup

Aerospike Connect for JMS outbound relies on the [Change Notification](/docs/guide/change_notification/index.html)
feature of the Aerospike Database. In the Aerospike server configuration, xdr
and change detection must be enabled. In addition a new datacenter must be
configured as an "http" xdr datacenter and the namespace must point to this as
xdr remote datacenter.

See the [Change Notification Configuration](/docs/guide/change_notification/configure.html)
documentation for reference but the relevant configuration might be:

```
xdr {
        enable-xdr true
        enable-change-notification true
        xdr-digestlog-path /etc/aerospike/digestlog.log 100G
        xdr-compression-threshold 1000
        xdr-info-port 3004
        datacenter my-jms {
                dc-type http
                http-version v1
                http-url http://my.jms.host:8080/
        }
}
namespace test {
        enable-xdr true
        xdr-remote-datacenter my-jms
        ...
}
```
