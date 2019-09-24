---
title: Spring Session Aerospike
description: Aerospike connector for Spring Session
---

[Spring Session](http://projects.spring.io/spring-session/) is a Java framework which provides an API and implementations for managing a user's session information. 

Spring Session provides the following features:
- API and implementations for managing a user's session.
- HttpSession - allows replacing the HttpSession in an application container (i.e. Tomcat) neutral way.
- Clustered Sessions - Spring Session makes it trivial to support clustered sessions without being tied to an application container specific solution.
- Multiple Browser Sessions - Spring Session supports managing multiple users' sessions in a single browser instance (i.e. multiple authenticated accounts similar to Google).
- RESTful APIs - Spring Session allows providing session ids in headers to work with RESTful APIs.
- WebSocket - provides the ability to keep the HttpSession alive when receiving WebSocket messages.

Spring Session Aerospike extends Spring Session to add support for storing user sessions in an Aerospike database cluster.

### Get Started
This [Spring Boot sample project](https://github.com/aerospike/spring-session-example) demonstrates how to configure a Spring Boot application to use Spring Session Aerospike.


