---
title: Installation
description: Install the Aerospike REST Client to enable RESTful communication with an Aerospike cluster.
---

The Aerospike REST Client is a Spring Boot Application. It can either be run directly from the git repo with `./gradlew bootRun`, or built into a `.war` file and deployed to a WAR container such as Tomcat.

## 1- Dependencies

- The REST Client requires Java 8.
- The REST Client requires an Aerospike Server to be installed and reachable.

## 2 - Installation

### Downloading a WAR file
The latest version of the REST Client `.war` file can be found at: [REST Client Releases](/download/client/rest)

### Running From Source
For development purposes it is possible to

- Clone the [Aerospike REST Client](https://github.com/aerospike/aerospike-client-rest)
- From the root directory run `./gradlew bootRun`

### Building a WAR file

To build a copy of the `.war` file for the REST Client:

- Clone the [Aerospike REST Client](https://github.com/aerospike/aerospike-client-rest)
- From the root directory run `./gradlew bootWar`
- The `.war` will be located in the `build/libs` directory

### Running on Tomcat

- If not already installed, download and install [Tomcat](https://tomcat.apache.org) . We recommend the Core distribution of Tomcat 9, found under the Binary Distributions section.

This will create a root installation folder which looks something like

    ./bin/
    ./conf/
    ./logs/
    ./webapps/

- Place the REST Client `.war` file in your tomcat installation's `webapps` folder.
- For more detailed server configurations, refer to the Documentation for the version of Tomcat which you are using. For Tomcat 9 these are located at: <https://tomcat.apache.org/tomcat-9.0-doc/introduction.html>
- Start tomcat. One way to do this is by running `bin/catalina.sh run` or `bin/catalina.sh start` from the root folder of your Tomcat installation.

## 3- Configuration

By default the REST Client looks for an Aerospike Server available at `localhost:3000` . The following environment variables allow specification of a different host/port.

- `aerospike_restclient_hostname` This is the IP address or Hostname of a single node in the cluster. It defaults to `localhost`
- `aerospike_restclient_port` The port to communicate with the Aerospike cluster over. Defaults to `3000`
- `aerospike_restclient_hostlist` A comma separated list of cluster hostnames and ports. If this is specified, it overrides the previous two environment variables. The format is described below:

``` Bash
    The string format is : hostname1[:tlsname1][:port1],...
    * Hostname may also be an IP address in the following formats.
    *
    * IPv4: xxx.xxx.xxx.xxx
    * IPv6: [xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx]
    * IPv6: [xxxx::xxxx]
    *
    * IPv6 addresses must be enclosed by brackets.
    * tlsname and port are optional.
    */
```

The REST Client also allows authentication to an Aerospike Enterprise edition server with security enabled. The following environment variables are used to find authentication information. **The Aerospike REST Client supports a single user only.**

- `aerospike_restclient_clientpolicy_user` This is the name of a user registered with the Aerospike database. This variable is only needed when the Aerospike cluster is running with security enabled.
- `aerospike_restclient_clientpolicy_password` This is the password for the previously specified user. This variable is only needed when the Aerospike cluster is running with security enabled.

### TLS Configuration

Beginning with version `1.1.0` the Aerospike REST Client supports TLS communication between the client and the Aerospike Server. (This feature requires an Enterprise Edition Aerospike Server). If utilizing TLS, the `aerospike_restclient_hostlist` variable should be set to include appropriate TLS Names for each of the Aerospike Nodes. For example: `localhost:cluster-tls-name:4333` The following environment variables allow configuration of this connection:

* `aerospike_restclient_ssl_enabled` boolean, set to to `true` to enable a TLS connection with the Aerospike Server. If no other SSL environment variables are provided, the REST client will attempt to establish a secure connection utilizing the default Java SSL trust and keystore settings. Default: `false`
* `aerospike_restclient_ssl_keystorepath` The path to a Java KeyStore to be used to interact with the Aerospike Server. If omitted the default Java KeyStore location and password will be used.
* `aerospike_restclient_ssl_keystorepassword` The password to the keystore. If a keystore path is specified, this must be specified as well.
* `aerospike_restclient_ssl_keypassword` The password for the key to be used when communicating with Aerospike. If omitted, and `aerospike_restclient_ssl_keystorepassword` is provided,  the value of `aerospike_restclient_ssl_keystorepassword` will be used as the key password.
* `aerospike_restclient_ssl_truststorepath` The path to a Java TrustStore to be used to interact with the Aerospike Server. If omitted the default Java TrustStore location and password will be used.
* `aerospike_restclient_ssl_truststorepassword` The password for the truststore. May be omitted if the TrustStore is not password protected.
* `aerospike_restclient_ssl_forloginonly` Boolean indicating that SSL should only be used for the initial login connection to Aerospike. Default: `false`
* `aerospike_restclient_ssl_allowedciphers` An optional comma separated list of ciphers that are permitted to be used in communication with Aerospike. Available cipher names can be obtained by `SSLSocket.getSupportedCipherSuites()`.
* `aerospike_restclient_ssl_allowedprotocols` An optional comma separated list of protocols that are permitted to be used in communication with Aerospike. Available values can be acquired using `SSLSocket.getSupportedProtocols()`. By Default only `TLSv1.2` is allowed.
