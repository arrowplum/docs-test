---
title: Demo Application
assets: /docs/guide/assets
---

A demo application is provided along with the source code in the downloaded
package to demonstrate the end-to-end usage of the library. It launches a Jetty
webserver instance which runs the demo http servlet. Note that this example can
handle only plain HTTPv1.1 traffic. Do not send SSL or HTTP v2 traffic to this
demo application. In other words, do not configure the aerospike server to use
HTTP v2 or use ‘https’ in the URL.

This demo application will print the metadata of the delivered message to
standard output. The demo program can be modified trivially to also output the
record contents.

To start the demo application along with Jetty webserver, execute the
`start.sh` script in the `./bin` directory of the downloaded package. 
```
./start.sh
```
It will
start listening on port 8080 of the machine where it ran. For testing, you can
run multiple instances of the application on different machines. If you want to
run multiple instances on the same machine, you can directly invoke the java
command with different port numbers. The command is:
```
java -jar <path to jetty jar> --port <port> <path to war file>
```

If you configure the Aerospike server (as explained in Configuration section),
and insert some data into aerospike server, the demo application prints the
metadata of the record that it receives: namespace, set, digest, key, ttl,
generation, and bin names. The output will look something like this
```
$ ./bin/start.sh
2018-07-03 12:29:00.889:INFO::main: Logging initialized @176ms to org.eclipse.jetty.util.log.StdErrLog
2018-07-03 12:29:00.894:INFO:oejr.Runner:main: Runner
2018-07-03 12:29:00.975:INFO:oejs.Server:main: jetty-9.4.10.v20180503; built: 2018-05-03T15:56:21.710Z; git: daa59876e6f384329b122929e70a80934569428c; jvm 10.0.1+10
2018-07-03 12:29:01.192:WARN:oeja.AnnotationParser:main: Unrecognized runtime asm version, assuming 393216
2018-07-03 12:29:01.473:INFO:oeja.AnnotationConfiguration:main: Scanning elapsed time=280ms
2018-07-03 12:29:01.600:INFO:oejs.session:main: DefaultSessionIdManager workerName=node0
2018-07-03 12:29:01.600:INFO:oejs.session:main: No SessionScavenger set, using defaults
2018-07-03 12:29:01.602:INFO:oejs.session:main: node0 Scavenging every 660000ms
2018-07-03 12:29:01.626:INFO:oejsh.ContextHandler:main: Started o.e.j.w.WebAppContext@5906ebcb{/,file:///private/var/folders/lt/f66l1lv52n51xxdgsc_p18cr0000gn/T/jetty-0.0.0.0-8080-aerospike-connect-sdk-0.99.1-SNAPSHOT.war-_-any-12970927407087852248.dir/webapp/,AVAILABLE}{file:///Users/jhecking/aerospike/citrusleaf/aerospike-connect/target/packages/aerospike-connect-sdk-0.99.1-SNAPSHOT/lib/aerospike-connect-sdk-0.99.1-SNAPSHOT.war}
2018-07-03 12:29:01.642:INFO:oejs.AbstractConnector:main: Started ServerConnector@4f49f6af{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
2018-07-03 12:29:01.643:INFO:oejs.Server:main: Started @935ms
ERROR StatusLogger Log4j2 could not find a logging implementation. Please add log4j-core to the classpath. Using SimpleLogger to log to the console...
2018-07-03 12:29:35.197:INFO:oejshC.ROOT:qtp1585787493-13: com.aerospike.connect.demo.AerospikeConnectServlet: {"msg":"write","key":["test","tests","i2Ejrq8uPFTLpwAn2TI2YcaybfQ=","key"],"gen":0,"exp":0,"lut":0,"bins":[{"name":"i","type":"int","value":42},{"name":"s","type":"str","value":"foo"},{"name":"f","type":"float","value":1.99}]}

```
