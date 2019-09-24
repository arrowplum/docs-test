---
title: Spring Data for Aerospike Tutorial
description: Spring Data for Aerospike Tutorial - Spring Data for Aerospike
---
## Part 3 : Spring Data for Aerospike

In the first part of this tutorial we created the Gradle project with Spring Boot.

In the second part, we configured Spring MVC and ThymeLeaf templates to display a basic web page via Tomcat.

In this part we’ll connect to an Aerospike Cluster via Spring Data for Aerospike. We will use these tools to persist data to Aerospike in our Spring Boot Web Application.

## Aerospike Persistence with Spring Boot
Spring Boot comes with pre-configured options for relational databases. Like other things in Spring Boot, these are enabled by simply having the dependency on your classpath.

However this tutorial will focus on the Aerospike database. Spring Data does a great job of abstracting the persistence layer. We will incorporate Spring Data for Aerospike, create some entities and persist them using a Spring Data Repository.

## Aerospike
Aerospike is the company behind the open source Aerospike distributed database, which is built to provide high performance at high scale. Aerospike is used for mission critical apps requiring blazing speed, cost efficient scaling & no downtime.

Aerospike is developed and optimized for 64-bit Linux. Aerospike supports popular Linux distributions, with .rpm packages for Red Hat variants, .deb packages for Ubuntu and Debian, binary packages and source builds.

Aerospike provides an Amazon Linux AMI, prebuilt with the required dependencies and other cloud providers are supported as well.

OS X and Windows are supported using Vagrant managed virtual machines. With the Vagrant cloud virtual machine distribution system you will be able to download and run Aerospike with a few simple commands.

Configuring the Aerospike Server is out of the scope of this tutorial. Please follow the instructions available here Install Aerospike

## Aerospike and Spring Boot
When you include the Spring Data for Aerospike dependency in your build.gradle or Maven POM, Aerospike Spring Data is included by default. As typical with Spring Boot, Spring Data for Aerospike is setup and configured with sensible default properties. We are also including the spring data commons dependency which is required for Spring Data for Aerospike. Don't forget to run a Gradle -> Refresh Gradle Project menu command after adding these dependencies to your build.gradle.

####build.gradle
```java
dependencies {
    ...
    compile('org.springframework.data:spring-data-commons')
    compile('com.aerospike:spring-data-aerospike:1.0.2.RELEASE')
}
```
### Example Entity
In our example application, we’re going to use a product for an ecommerce website. The entity classes are in a package called ‘domain’.

#### Product.Java
```java
package com.aerospike.spring.boot.example.domain;

import org.springframework.data.annotation.Id;


public class Product {
    @Id
    private Integer id;
    private String productId;
    private String description;
    private String imageUrl;
    private double price;
    
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    public String getImageUrl() {
        return imageUrl;
    }
    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }
    public double getPrice() {
        return price;
    }
    public void setPrice(double price) {
        this.price = price;
    }
}
```
## Spring Data for Aerospike

Using Spring Data for Aerospike can save you a lot of time when interacting with the Aerospike database. Spring Data for Aerospike implements the Repository Pattern. This design pattern was originally defined by Eric Evans and Martin Fowler, in their book Domain Driven Design. This is one of those time tested computer science books, over a decade old, still remains relevant today.

You don’t need to use Spring Data for Aerospike for this type of project, but using Spring Data for Aerospike will make your life as a developer easier. A common alternative to Spring Data for Aerospike would be to use the widely accepted DAO pattern, The DAO pattern is very similar to the Repository Pattern. The advantage of using Spring Data for Aerospike is that you’ll be writing a lot less code. Spring Data for Aerospike works like the Spring Integration Gateways, where you define an interface, and Spring provides the implementation at run time.

### Spring Data for Aerospike CRUD Repository
The Spring Data for Aerospike CRUD Repository is my favorite feature of Spring Data for Aerospike. Similar to coding with a Spring Integration Gateway, you can just define an interface. Spring Data for Aerospike uses generics and reflection to generate the concrete implementation of the interface we define.

Defining a repository for our Product domain class is as simple as defining an interface and extending the CrudRepository interface. You need to declare two classes in the generics for this interface. They are used for the domain class the repository is supporting, and the type of the id declared of the domain class.

For our Product domain class we can define a Spring Data for Aerospike repository as follows.

#### ProductRepository.java
```java
package com.aerospike.spring.boot.example.repository;

import org.springframework.data.aerospike.repository.AerospikeRepository;

import com.aerospike.spring.boot.example.domain.Product;

public interface ProductRepository extends AerospikeRepository<Product, Integer> {
}
```
### Integration Testing with Spring Data for Aerospike and JUnit
For our integration tests, we’re going to use a Spring Context to wire up beans to support our tests. If we were not using Spring Boot, we’d need to create a number of beans ourselves. Normally we would need to create:

- Aerospike Client
- Aerospike Template
- A Transaction Manager

But since we’re using Spring Boot, we don’t need to write code to create these beans. For the purposes of our integration tests for our Spring Data for Aerospike repositories, we can complete our Java configuration with just annotations.

#### RepositoryConfiguration.java
```java
package com.aerospike.spring.boot.example.configuration;

import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.aerospike.core.AerospikeTemplate;
import org.springframework.data.aerospike.repository.config.EnableAerospikeRepositories;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.aerospike.client.AerospikeClient;
import com.aerospike.client.policy.ClientPolicy;

@Configuration (1)
@EnableAerospikeRepositories(basePackages = {"com.aerospike.spring.boot.example.repository"})  (2)
@EnableAutoConfiguration
@EnableTransactionManagement (3)
public class RepositoryConfiguration {

    public @Bean(destroyMethod = "close") AerospikeClient aerospikeClient() {  (4)

        ClientPolicy policy = new ClientPolicy();
        policy.failIfNotConnected = true;

        return new AerospikeClient(policy, "localhost", 3000);  (5)
    }

    public @Bean AerospikeTemplate aerospikeTemplate() {  (6)
        return new AerospikeTemplate(aerospikeClient(), "test");
    }
}
```
1. `@Configuration` tells the Spring Framework this is a Java configuration class.
2. `@EnableAerospikeRepositories(basePackages = {"com.aerospike.spring.boot.example.repository"})` tells Spring Boot to do its auto configuration magic. This is what causes Spring Boot to automatically create the Spring Beans with sensible defaults for our tests.
3. `@EnableTransactionManagement` Enables Spring’s annotation driven transaction management
4. `@Bean` Configuration for AerospikeClient
5. Host and port of Aerospike server
6. `@Bean` Configuration for AerospikeTemplate

Through this configuration, we have everything we need to use Aerospike with Spring Data for Aerospike in JUnit tests.

### Spring Data for Aerospike JUnit Integration Test
With our Spring Java configuration done, our JUnit integration test becomes very simple to write.

In this post, I am not going to go in depth with Spring Data for Aerospike. This is a fairly large and complex project in the Spring Framework. We’re going to use the CRUD repository from Spring Data for Aerospike. CRUD stands for CReate, Update, Delete; the basic persistence operations. Simply extending Spring Data for Aerospike’s CRUD Repository interface, as we did above, for the specified Entity we will get methods which will:

- Save an entity
- Find an entity based on its ID
- Check if an entity exists based on its ID
- Get a list of all entities Get a count of all entities
- Delete an entity
- Delete all entities

We have written a simple integration test for the Spring Data for Aerospike repository defined above. In the test, we are going to do some basic operations, like creating an entity, saving an entity, and fetching an entity from Aerospike. While we have written a minimal amount of code in this example, the data is really getting saved into Aerospike. You don’t see any aerospike code happening, but it is getting generated behind the scenes for us. Once you grasp how little code you are writing, and how much is happening under the covers for you, you can appreciate what a powerful tool Spring Data for Aerospike is.

#### ProductRepositoryTest.java
```java
package com.aerospike.spring.boot.example.repository;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.aerospike.spring.boot.example.configuration.RepositoryConfiguration;
import com.aerospike.spring.boot.example.domain.Product;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = {RepositoryConfiguration.class})
public class ProductRepositoryTest {
    @Autowired
    private ProductRepository productRepository;

    @Test
    public void testSaveProduct(){
        Product product = new Product();
        product.setDescription("Spring Data for Aerospike Shirt");
        product.setPrice(18.95);
        product.setProductId("1234");
        product.setId(100);

        productRepository.deleteAll();
        productRepository.save(product);
        assertNotNull(product.getId()); //not null after save

        //fetch from DB
        Product fetchedProduct = productRepository.findOne(product.getId());
    
        //should not be null
        assertNotNull(fetchedProduct);
    
        //should equal
        assertEquals(product.getId(), fetchedProduct.getId());
        assertEquals(product.getDescription(), fetchedProduct.getDescription());
    
        //update description and save
        fetchedProduct.setDescription("New Description");
        productRepository.save(fetchedProduct);
    
        //get from Aerospike, should be updated
        Product fetchedUpdatedProduct = productRepository.findOne(fetchedProduct.getId());
        assertEquals(fetchedProduct.getDescription(), fetchedUpdatedProduct.getDescription());
    
        //verify count of products in DB
        long productCount = productRepository.count();
        assertEquals(productCount, 1);
    
        //get all products, list should only have one
        Iterable<Product> products = productRepository.findAll();
    
        int count = 0;
        for(Product p : products){
            count++;
        }
    
        assertEquals(count, 1);
    }
}
```
## Loading Data Using Spring Data on Startup
### Creating a Product Loader
The Spring Framework comes out of the box with a number of events, and you’re able to extend the event functionality for your own purposes.

For example, if we would like to do something on startup we have two events we can consider using. Traditionally under Spring Framework, we can use the `ContextRefreshedEvent`. This event has been around since the beginning of the Spring Framework.

If you’re using Spring Boot, you do have additional events to choose from. We often want to use a startup event to seed data for tests, so in this case, we need the database connection to be setup. Reading about the Spring Boot Events, we thought the event we would like to use is `ApplicationPreparedEvent`. But in testing it out, this was not the case. We ran into some issues with getting the event listeners setup properly in the Spring Boot Context. we found we had better results using `ContextRefreshedEvent`.

#### ProductLoader.java
This class implements the ApplicationListener interface, so it is called with the `ContextRefreshedEvent` on startup. We’re using Spring to inject the Spring Data for Aerospike repository into the class for our use. In this example, we are creating two entities and saving them into Aerospike.
```java
package com.aerospike.spring.boot.example.domain.bootstrap;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

import com.aerospike.spring.boot.example.domain.Product;
import com.aerospike.spring.boot.example.repository.ProductRepository;

@Component
public class ProductLoader implements ApplicationListener<ContextRefreshedEvent> {

    @Autowired
    private ProductRepository productRepository;

    private Logger log = LoggerFactory.getLogger(ProductLoader.class);

    /*
     * (non-Javadoc)
     * 
     * @see
     * org.springframework.context.ApplicationListener#onApplicationEvent(org.
     * springframework.context.ApplicationEvent)
     */
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        productRepository.deleteAll();
        Product shirt = new Product();
        shirt.setId(10001);
        shirt.setDescription("Aerospike Shirt");
        shirt.setPrice(18.95);
        shirt.setImageUrl("http://hunt4freebies.com/wp-content/uploads/2014/07/Aerospike-T-shirt.png");
        shirt.setProductId("235268845711068308");
        productRepository.save(shirt);

        log.info("Saved Shirt - id: " + shirt.getId());

        Product mug = new Product();
        mug.setId(10002);
        mug.setDescription("Aerospike Mug");
        mug.setPrice(4.99);

        mug.setImageUrl(
                "https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcR3TPM0daB-aXKfxdYqlHHgQrz67bSCPKUcpbmzPXtvo3GillfR");
        mug.setProductId("168639393495335947");
        productRepository.save(mug);

        log.info("Saved Mug - id:" + mug.getId());
    }
}
```
### Running Product Loader
We still have our Spring Boot application class which was created earlier.

#### SpringBootWebApplication.java
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
When we run this class, it will startup tomcat for us. In the console log, we can see the output of the log statements from our ProductLoader class.

```
2017-04-18 09:01:21.230  INFO 88738 --- [           main] c.a.s.b.e.d.bootstrap.ProductLoader      : Saved Shirt - id: 10001
2017-04-18 09:01:21.231  INFO 88738 --- [           main] c.a.s.b.e.d.bootstrap.ProductLoader      : Saved Mug - id:10002
2017-04-18 09:01:21.288  INFO 88738 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-04-18 09:01:21.292  INFO 88738 --- [           main] c.a.s.b.e.SpringBootExampleApplication   : Started SpringBootExampleApplication in 4.433 seconds (JVM running for 5.138)
```
To run the Spring Boot application, simply right click on the SpringBootWebApplication class and select “Run ‘SpringBootWebApplica…'”

## Conclusion
In this part of the tutorial we have setup Aerospike and Spring Data for Aerospike for use. You can see how easy it is to persist data to Aerospike using Spring Data JPA repositories.

In the [next section](/docs/connectors/community/spring/spring_data/tutorial_4.html), we will incorporate what we have learned into a full Spring MVC application.
