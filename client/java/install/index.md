---
title: Installation
description: Get started using Aerospike’s Java client with the Aerospike database for your real-time, big data application.
categories:
  - aerospike-client-java
tags:
  - aerospike-client-java
  - install
---

### Prerequisites

- [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 8 and above.

- One of the following build managers:
    - [Maven](https://maven.apache.org/download.cgi)
    - [Ivy](http://ant.apache.org/ivy)
    - [Gradle](http://gradle.org/gradle-download)
    - [SBT](http://www.scala-sbt.org/download.html)
    - [Leiningen](http://leiningen.org)

### Build from Source Code (Optional)

The client library can optionally be built from source code.  This step requires Maven.  The source code is available from:

- [Aerospike Website](/download/client/java).  Download source code package and build library.   
    ```bash
    tar xvf aerospike-client-java-{VERSION}.tgz
    cd aerospike-client-java-{VERSION}
    ./build_all
    ```

- [Github](http://github.com/aerospike/aerospike-client-java).  Clone source code repository and build library.
    
    ```bash
    git clone git@github.com:aerospike/aerospike-client-java.git
    cd aerospike-client-java
    ./build_all
    ```

The build installs the client library into your local Maven repository.  The client package includes the following projects:

Folder | Contents 
--- | --- 
client | APIs and resources for building an application to communicate with the Aerospike cluster.
examples | Example programs to build and test.
benchmarks | Benchmark program for testing performance your Java client on the Aerospike cluster.
servlets | Servlet demonstration using Aerospike with HTTP (REST).

### Reference Client Library

Your project needs to reference the client library as a dependency.  If the client library is found on your local Maven repository, that library is used.  Otherwise, the client library on a remote [Maven Repository](http://search.maven.org/#search%7Cga%7C1%7Caerospike) is used.

Use your favorite build manager to define this dependency on the Aerospike Java client.

- Maven — add the following dependency to `pom.xml`:
```xml
    <dependencies>
      <dependency>
        <groupId>com.aerospike</groupId>
        <artifactId>aerospike-client</artifactId>
        <version>4.1.11</version>
      </dependency>
    </dependencies>
```
- Ivy — add the following to `ivy.xml`:
```xml
    <dependencies>
      <dependency org="com.aerospike" name="aerospike-client" revision="4.1.11" />
    </dependencies>
```
- Gradle — add the following to `build.gradle`:
```xml 
    repositories {
      mavenCentral()
    }
            
    dependencies {
      compile "com.aerospike:aerospike-client:4.1.11"
    }
```
- SBT — add the following to `build.sbt`:
```xml
    libraryDependencies += "com.aerospike" % "aerospike-client" % "latest.integration"
```
- Leiningen — add the following to `project.clj`:
```xml
    :dependencies [[com.aerospike/aerospike-client "LATEST"]]
```

#### Alternate Crypto Library

The default crypto library used for aerospike-client is GNU Crypto.  To use the alternate Bouncy Castle crypto library, the artifactId is aerospike-client-bc instead of aerospike-client.

```xml
    <dependencies>
      <dependency>
        <groupId>com.aerospike</groupId>
        <artifactId>aerospike-client-bc</artifactId>
        <version>4.1.11</version>
      </dependency>
    </dependencies>
```

#### Netty

AerospikeClient asynchronous methods now support Netty event loops in addition to direct NIO event loops.  Netty is optional.  To use Netty with AerospikeClient, declare netty library dependencies in your build file.

- Maven — add the following dependency to `pom.xml`:
```xml
    <dependencies>
      <dependency>
        <groupId>com.aerospike</groupId>
        <artifactId>aerospike-client</artifactId>
        <version>4.1.11</version>
      </dependency>

      <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-handler</artifactId>
        <version>4.1.28.Final</version>
      </dependency>

      <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-transport</artifactId>
        <version>4.1.28.Final</version>
      </dependency>

      <!-- Only needed when using epoll event loops on linux -->
      <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-transport-native-epoll</artifactId>
        <classifier>linux-x86_64</classifier>
        <version>4.1.28.Final</version>
      </dependency>
    </dependencies>
```

### Next Steps
- Run [Examples](https://github.com/aerospike/aerospike-client-java/tree/master/examples)
- Try the [Benchmark Tool](/docs/client/java/benchmarks.html)
