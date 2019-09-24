---
title: Change Notification Configuration
assets: /docs/guide/assets
---

If you are new to XDR, refer to [XDR config
guide](/docs/operations/configure/cross-datacenter/index.html) to know about
general XDR configuration and old configs. The content below talks only about
the new configs necessary for the Change Notification Framework.

Change Notification is an Enterprise Edition feature requiring a [`feature-key-file`](/docs/reference/configuration#feature-key-file) with *asdb-change-notification* set to *true*.

The default location of the feature-key-file is `/etc/aerospike/features.conf`.  
Contact Aerospike sales/support if you do not have a feature-key.

To enable the Change Notification framework functionality, the Aerospike Configuration Parameter [`enable-change-notification`](/docs/reference/configuration#enable-change-notification) must be set to true.  

The following table describes the new configs added and their explanation.

Config Name | Section | Allowed Values | Default Value | Description
---|---|---|---|---
[`enable-change-notification`](/docs/reference/configuration#enable-change-notification)|xdr|true, false|false|Static config. When enabled, it spins up the machinery necessary for http communication. Also does the validation against feature keys.
[`dc-type`](/docs/reference/configuration#dc-type)|datacenter|aerospike, http|aerospike|Only skeleton DC’s type can be changed dynamically to http (from default `aerospike`)
[`http-version`](/docs/reference/configuration#http-version)|datacenter|v1, v2, v2-prior-knowledge|v2|Dynamic Config. `v1` means communicate only in http v1.1. `v2-prior-knowledge` means communicate only in http v2. `v2` means try http v2, if not fall back to v1
[`http-url`](/docs/reference/configuration#http-url)|datacenter||NA|Dynamic config. Multiple lines of this config are recommended. Each one specifies an endpoint to which XDR will ship in a load balanced fashion. (E.g., http://192.128.10.1:8080/xdrproxy). If the URL is prefixed with `http://`, clear text HTTP is used.  If the URL is prefixed with `https://`, SSL is used
[`tls-name`](/docs/reference/configuration#tls-name)|datacenter||NULL| This is not a new config. It is applicable to both aerospike and http dc types. Value is the name of the tls sub-stanza that should be used to pick ca-file or ca-path from.

Putting all things together, a sample configuration may look something like
below:

```
xdr {
        enable-xdr true
        enable-change-notification true
        xdr-digestlog-path /etc/aerospike/digestlog.log1 100G
        xdr-compression-threshold 1000
        xdr-info-port 3004

        datacenter DC1 {
                dc-node-address-port 127.0.0.1 6000
        }

        datacenter jetty {
                dc-type http
                http-version v2
                http-url https://10.0.0.1:8080/connect
                http-url https://10.0.0.2:8080/connect
        }
}

namespace test {
        enable-xdr true
        xdr-remote-datacenter jetty
        replication-factor 2
        memory-size 4G
        default-ttl 30d
        storage-engine memory
}
```

## Dynamic config changes
Keeping up with the spirit of Aerospike’s philosophy of avoiding node restart
for common config changes, we kept most of the config options dynamic.

The URLs specified with the http-url config can be dynamically added or removed.
This is inline with aerospike destination node configuration. Sample info
command is as follows:
```
asinfo -v "set-config:context=xdr;dc=XDRPROXY;http-url=http://192.128.10.1:8080/xdrproxy;action=remove"
asinfo -v "set-config:context=xdr;dc=XDRPROXY;http-url=http://192.128.10.1:8080/xdrproxy;action=add"
```

The http-version can be set dynamically like any other dc attribute. But make
sure that the webserver is capable to handle the configured protocol. The new
setting will be used for any new connections that will be made.
```
asinfo -v "set-config:context=xdr;dc=XDRPROXY;http-version=v2-prior-knowledge"
```

The dc-type cannot be dynamically changed when the DC is in use. It does not
make sense to dynamically change the type on the fly.
