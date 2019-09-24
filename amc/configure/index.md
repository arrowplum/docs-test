---
title: Configure
description: Learn how to Configure port number and ssl certificate.
---

### Configuration for version 4.0 and later

AMC uses a single configuration file located at `/etc/amc/amc.conf` on Linux, or `/Library/amc/amc.conf` on MacOS.
The configuration file follows the  [TOML](https://github.com/toml-lang/toml) syntax.

The configuration is divided into the following contexts.
```
[AMC]              # (Required) Configuration related to the runtime behaviour of AMC

[amc.clusters]     # (Optional) List of clusters that will always be monitored by AMC on start up.

[mailer]           # (Optional) Configuration used by AMC to send out alert emails

[basic_auth]       # (Optional) The HTTP Basic Authentication credentials for AMC to use

[TLS]              # (Optional) The set of root certificate authorities that AMC uses when verifying server certificates

```

### Runtime Configurations

These configurations define the runtime behaviour of AMC.
```
[AMC]
update_interval                 = 5
certfile                        = "/home/amc/cert.pem"  # optional
keyfile                         = "/home/amc/key.pem"   # optional
database                        = "/home/amc/amc.db"
bind                            = ":8081"
loglevel                        = "info"
errorlog                        = "/home/amc/amc.log"
chdir                           = "/home/amc"
static_dir                      = "/home/amc/static"
cluster_inactive_before_removal = 1800
```

`update_interval` - the default time interval (in seconds) in which AMC should capture statistics for the clusters that AMC is monitoring
```
update_interval = 5
```

`certfile`, `keyfile` (optional)  - the public/private key pair to run AMC app server in https mode. The files must contain PEM encoded data. The certificate file may contain intermediate certificates following the leaf certificate to form a certificate chain.
```
certfile = "/home/amc/cert.pem"
keyfile  = "/home/amc/key.pem"
```

`database`  - the file which will be used to persist AMC notifications, backup/restore job history and other internal state. You can manually backup this file by making a copy of it if you would like to ensure that this data is maintained. This file should typically not grow beyond approximately 100MB in size.
```
database = "/home/amc/amc.db"
```

`bind` - the ip and port which AMC should bind to
```
bind = ":8081"
```

`loglevel`  - the level of detail at which AMC should log messages. One of
  debug, warn, error, info.
```
loglevel = "info" # one of debug, warn, error, info
```

`errorlog` - the file to which AMC will write the logs to
```
errorlog = "/home/amc/amc.log"
```

`chdir` -  the working directory of AMC
```
chdir = "/home/amc"
```

`static_dir`  - the directory which contains the static resources like css, html, js files.
```
static_dir = "/home/amc/static"
```

`cluster_inactive_before_removal` - if the user has not requested any statistics for a cluster for more than           `cluster_inactive_before_removal` seconds  then AMC stops monitoring the cluster. A value <= 0 implies the clusters will  never be removed. Keep in mind that clusters defined in the config file to monitor will never be removed, regardless of this setting.
```
cluster_inactive_before_removal = 1800
```

### Cluster Configuration
This configuration is *optional* and allows for a cluster to be loaded automatically in the MultiCluster view.

List of Aerospike clusters that are always monitored by AMC.
```
[amc.clusters]

[amc.clusters.clusterone] # unused
host     = "192.168.121.121"
port     = 3000
show_in_ui = true            # optional (needed to be visible in MultiCluster view)
tls_name = "clusteronetls"   # optional
user     = "admin"           # optional (needed for loading ACL protected cluster in MultiCLuster view)
password = "admin123"        # optional (needed for loading ACL protected cluster in MultiCluster view)
alias    = "clusterone"      # optional

[amc.clusters.clustertwo] # unused
host     = "192.168.121.122"
port     = 3000
show_in_ui = true            # optional (needed to be visible in MultiCluster view)
tls_name = "clustertwotls"   # optional
user     = "admin"           # optional (needed for loading ACL protected cluster in multiCluster view)
password = "admin123"        # optional (needed for loading ACL protected cluster in multiCluster view)
alias    = "clustertwo"      # optional
```

Each cluster has the following configurations

`host`, `port` - the IP, port of a node in the Aerospike cluster
```
host = "192.168.121.121"
port = 3000
```

`user`, `password` (optional) - the user name, password for an *access
controlled* Aerospike cluster
```
user     = "admin"
password = "admin123"
```

`alias` (optional) - the alias of the Aeorspike cluster in AMC
```
alias = "clusterone"
```

`tls_name` (optional) - the advertised name of the target node used during authentication by amc.
Warning - the certificate needs to be part of the system cert pool or needs
to be specified as a configuration to AMC
```
tls_name = "clusteronetls"
```

`show_in_ui` (optional) - display the cluster in the ui MultiCluster view when AMC is loaded into the browser.
```
show_in_ui = true
```

`use_services_alternate` (optional) - allows the use of `services_alternate` on the server to be able to connect from a public network to the cluster.
```
use_services_alternate = true
```


### Mail Configuration
This configuration is *optional* and available only in the enterprise edition.

Configuration used by AMC to send out email alerts for the monitored clusters. The email server should accept user/pass access (extra configuration may be needed on the mail server to allow this)

```
[mailer]
template_path 		= "/home/amc/mailer/templates"
host          		= "smtp.outlook.com"
port          		= 587
user          		= "user"
password      		= "user123"
send_to       		= ["monitorone@gmail.com", "monitortwo@yahoo.com"]
accept_invalid_cert	= false
```


`template_path` - the directory containing the templates for the emails
```
template_path = "/home/amc/mailer/templates"
```

`host`, `port` - the host name and port of the smtp server
```
host = "smtp.outlook.com"
port = 587
```

`user`, `password` - the user name and password at the smtp server. Please note that the user name must be in an email address format.
```
user     = "user@example.com"
password = "user123"
```

`send_to` - the list of emails to send out alerts to
```
send_to = ["monitorone@gmail.com", "monitortwo@yahoo.com"]
```

`accept_invalid_cert` - if true, continue sending mail even if the certificate is invalid
```
accept_invalid_cert = true
```

{{#note}}The `user` field in the `[mailer]` configuration is used as the from address when sending notification alerts via e-mail. This field must populated in order for e-mail notifications to be successfully sent.{{/note}}

### HTTP Basic Authentication
This configuration is *optional*.

The basic authentication credentials that AMC should use
```
[basic_auth]
user     = "user"
password = "user123"
```

`user`, `password` - the user name, password to use for HTTP Basic Authentication
```
[basic_auth]
user     = "user"
password = "user123"
```

### TLS Configuration
This configuration is *optional* and available only in the enterprise edition.

AMC will connect to the TLS enabled cluster just like any other client.

For standard authentication (server only), on the server side configuration, set [`tls-authenticate-client`](/docs/reference/configuration/#tls-authenticate-client) to `false`, which will cause clients to authenticate the server. 
In case of standard-authentication, it is sufficient to specify only the CA file path in `server_cert_pool` configuration on AMC.

`server_cert_pool` - the list of root certificate authorities that AMC uses when verifying server certificates.

For example,
here's the configuration on AMC when using standard-authentication.

```
[amc.clusters]

        [amc.clusters.AeroCluster]
        host = "172.17.0.5"
        tls_name = "AeroCluster"
        port = 4333
        show_in_ui = true


[TLS]

server_cert_pool = [
        "/app/TLS-Cert-Gen-Tool/AeroClusterCA.pem",
]
```

For mutual authentication, on the server side configuration, set [`tls-authenticate-client`](/docs/reference/configuration/#tls-authenticate-client) to `any` or `<SubjectName>`, where both client and server needs to be authenticated.
In this case, we need to specify the certificates that AMC will present to the server under `tls.client_certs` configuration on AMC.

For example,
here's the configuration on AMC when using mutual-authentication.

```
[amc.clusters]

        [amc.clusters.AeroCluster]
        host = "172.17.0.5"
        tls_name = "AeroCluster"
        port = 4333
        show_in_ui = true

[TLS]

server_cert_pool = [
        "/app/TLS-Cert-Gen-Tool/AeroClusterCA.pem",
]

[tls.client_certs]

        [tls.client_certs.AeroCluster]
        cert_file="/app/TLS-Cert-Gen-Tool/AeroClientCert.pem"
        key_file="/app/TLS-Cert-Gen-Tool/AeroClientKey.pem"
```

### Configuration prior to version 4.0

AMC Port Configuration:

By Default AMC runs on port number 8081. To change AMC port number user needs to update /etc/amc/config/gunicorn_config.py for example modify, from "0.0.0.0:8081" to "0.0.0.0:XXXX", where XXXX is the desired port number. After this change, AMC service needs to be restarted in order for the changes to take effect.

SSL Configuration:
This feature is available for Enterprise AMC users only.
To run AMC on HTTPS, please provision the ssl-certificate and ssl-key file paths within the /etc/amc/config/amc.cfg file. If you do not provide ssl certificate and key file, AMC will default to HTTP.
