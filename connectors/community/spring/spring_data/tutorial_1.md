---
title: Spring Data for Aerospike Tutorial
description: Spring Data for Aerospike Tutorial - Setup
---
## Part 1 : Setup

## Spring Starter using Spring Boot

The cool part about Spring Boot is it makes a lot of common sense guesses for you. 
Like if you add Spring Data to your build, it guesses you’re going to want a transaction manager. 
The transaction manager is just one example of a Spring component you’d normally need to wire up that Spring Boot will automatically do for you. 
Spring Boot actually has over 200 default choices which it makes for you.

Spring Boot takes a lot of the mundane pain out of building Spring Applications. It really is making Spring Fun again. 
Spring Boot is still relatively new in the family of Spring Projects.

##Building a Spring Boot Web Application
To build a simple web application we are going to use the following technologies.

- The spring framework
- Spring MVC for the web part.
- Thymeleaf for the template engine
- Aerospike for the persistence layer
- Spring Data to make persistence easier with Aerospike
- Tomcat as your application server

In this Spring Boot tutorial we are going to walk you through step by step in developing a web application using Spring Boot and the technologies listed above.

##Getting Started with Spring Boot
###Creating the Spring Boot Project Using Spring Tool Suite
Start up [Spring Tool Suite](https://spring.io/tools) and create a new Spring Starter Project. 

![New Project](/docs/connectors/assets/images/new_project.png)

Use "spring-boot-example" for the name of the poject and "com.aerospike" for the group. Select "Gradle (Buildship)" for the type of project.

![STS Create Project](/docs/connectors/assets/images/create_project_1.png)

Select Next

From the "Core" section check Security. From the "Template Engines" section check Thymeleaf. From the "Web" section check Web.

![STS Create Project](/docs/connectors/assets/images/create_project_2.png)

Select Finish

This will create a project with this structure:

![Build Structure](/docs/connectors/assets/images/project_structure.png)

#### build.gradle
The build.gradle file is below. This was customized for us based on the options we selected in the Spring Starter configuration. The presence of these dependencies is important because Spring Boot will make decisions on what to create automatically when certain things are found on the classpath.
```java
buildscript {
    ext {
        springBootVersion = '1.5.2.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-security')
    compile('org.springframework.boot:spring-boot-starter-thymeleaf')
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

#### SpringBootWebApplication.java
Create us a very basic Spring Boot application class. Technically, this is a Spring Configuration class. The annotation @SpringBootApplication enables the Spring Context and all the startup magic of Spring Boot.

```java
package com.aerospike.spring.boot.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootExampleApplication.class, args);
    }
}

```
#### SpringBootWebApplicationTests.java
Here’s a sample of a JUnit Integration test.
```java
package com.aerospike.spring.boot.example;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootExampleApplicationTests {

    @Test
    public void contextLoads() {
    }

}
```
## Conclusion
In this section we’ve taken a look at using gradle with Spring boot inside an STS application. In the [next section](/docs/connectors/community/spring/spring_data/tutorial_2.html) we will cover building a Spring Boot Web Application using ThymeLeaf as the UI.
