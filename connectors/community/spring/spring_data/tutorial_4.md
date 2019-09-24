---
title: Spring Data for Aerospike Tutorial
description: Spring Data for Aerospike Tutorial - Adding Spring MVC
---
## Part 4 : Adding Spring MVC

In this fourth and final part of the series, we tie everything together to provide a working Spring Boot web application. This application will display data from Aerospike, and allow you to create new records, update existing records, and delete selected records.

## Spring MVC
Now we are going to look at setting up a Spring MVC controller to support CRUD operations against the database.

MVC stands for Model, View, Controller. The MVC design pattern is probably the most popular design pattern used 
when writing code to generate dynamic web content. This design pattern is not limited to Java nor Spring. 
The MVC design pattern has been applied in Javascript, PHP, .NET, Python, and many other programming languages. 
The MVC pattern is popular because it does a great job of separating concerns, and leads you to a clean, maintainable, 
and easy to understand code base.

## MVC overview
### Model
Model refers to a data model, or some type of data structure. For example a web page showing a list of products, 
the ‘model’ would contain a list of product data.

### View
The view layer, in Java frequently a JSP. This will take data from the Model and render the view.

### Controller
The controller is often described as a traffic cop. It will take an incoming request, decide what to do with it, 
then direct the resulting action. For example the controller could get a view product request. 
It will direct a service to get the product data, then direct to the product view and provide the ‘model’ (product data) 
to the view.

### Single Responsibility Principle Applied to MVC
In an MVC application each component has a specific function in life. You should be able to unit test your controllers. 
Using Mocks, you should be able to unit test that your controller returns the proper model, and makes the proper decisions.

## CRUD Operations with Spring MVC
CRUD is a common acronym for Create, Read, Update, and Delete. In the last part of this series, 
we looked at creating a CRUD repository using Spring Data JPA. In this part, we’ll look at setting up the Spring 
MVC controller for the corresponding CRUD operations. We’ll continue using the Product class we previously used.

### Create
The Create operation is a two step operation. The first step needs to display the create form, the second needs to handle saving from the form post.

Here is the controller code for displaying the create product form.
```java
@RequestMapping(value = "product/new", method = RequestMethod.GET)  (1)
public String newProduct(Model model){
    model.addAttribute("product", new Product());
    return "productform";
}
```
The `@RequestMapping` annotation maps the url `product/new` to this controller action. Our controller method is taking in the model attribute. This is the ‘model’ being returned to the view layer.

Our create view is going to have a form post. We need a controller action to handle this.
```java
@RequestMapping(value = "product", method = RequestMethod.POST)  (1)
public String saveProduct(Product product){
    productService.saveProduct(product);       (2)
    return "redirect:/product/" + product.getId();  (3)
}
```
In this controller method, we’re handing the form post. The @RequestMapping annotation says to take the ‘url’ ‘product’ and 
the HTTP request method of POST to map it to this controller method. You can see how we’re asking for a Product object as 
input to the controller method. One of the cool things about Spring MVC is that it will take your form parameters and 
automatically bind them to a Product object. The object is automatically created and passed into your controller method. 
The Spring Framework saves you from the mundane work of parsing out HTTP request parameters.

You can see how we’re using a product service to handle the persistence. This is just a facade to the Spring Data for Aerospike 
repository we created in the last post. We are going to skip over the persistence code here. You’ll be able to find it in github. I want you to notice how I’m writing to an interface. The controller doesn’t know about persistence. It does not need to. Storing data is not the job of the controller. The controller code does not need to care.

On the last line of the `saveProduct` method, you can see we are returning a string with “redirect”. 
This tells Spring after the save action to redirect to the view to show the created item.

### Read
In Read operations, the client is going to tell you what it wants. In our case, the client will give us an Id value, 
and we’ll return the corresponding Product.

#### Read by Id
```java
@RequestMapping("product/{id}")   (1)
public String showProduct(@PathVariable Integer id, Model model){  (2)
  model.addAttribute("product", productService.getProductById(id));
  return "productshow";
}
```
1. You can see in our controller method, the Request Mapping is using ‘product’ with an id value in braces. This identifies that portion of the url path as an {id} value.
2. Now, we’re using a new annotation `@Pathvariable` to inject the id value from the url path into our controller as the ID variable. Again, we’re accepting the model variable into our controller. We’re asking the product service to get the product, and the result is appended to the model object, which is returned to the view. The controller method returns a string to indicate which view to render.

### List All
A common method is also to provide a list view. Normally, you’re going to want to add paging or some type of filter. 
However, in this example, we just want to show a simple example of listing products from Aerospike.
```java
@RequestMapping(value = "/products", method = RequestMethod.GET) (1)
public String list(Model model){
   model.addAttribute("products", productService.listAllProducts());
   return "products";
}
```
1. We’ve mapped this controller method to the URL ‘/products’. We ask the product service for a list of all products and append it to the model attribute “products”. The controller method returns the string ‘products’ to tell Spring MVC to render the products view.

### Update
Updates are actions against existing entities. Updates are similar to create actions, where we have two controller actions involved. 
With a create, we’re showing a form for a new item, while an update is going to be populated with data from an existing item. 
While this is very similar to the create action, we typically will want a separate controller action to show the edit 
form to capture the data for the update.
```java
@RequestMapping(value = "product/edit/{id}", method = RequestMethod.GET)
public String edit(@PathVariable Integer id, Model model){
    model.addAttribute("product", productService.getProductById(id));
    return "productform";
```
### Delete
There’s a few different ways to implement a delete action. One of the easiest is to use a url with the ID for the delete action. This can then be implemented on the web forms as a simple URL to click on. Below is the controller action for the delete action.
```java
@RequestMapping(value = "product/delete/{id}", method = RequestMethod.GET)
public String delete(@PathVariable Integer id){  (1)
    productService.deleteProduct(id);
    return "redirect:/products";
}
```
1. This method will take in the id value from the URL and pass it to the delete method of the product service. Since we’re not creating or updating a product, a typical course of action is to return to the list view. In this example, we redirect to the products view to show the user a list of products.

### Summary of CRUD Operations
At this point we’ve covered the necessary controller actions to support CRUD operations on an entity. 
You can see these operations work in conjunction with the Spring Data for Aerospike methods we 
looked at in the previous post on Spring Data for Aerospike. We are using a Facade Service to mask the 
Spring Data for Aerospike implementation. We’ll take a look at the Facade in the next section.

### Spring Facade Service

You can see in the controller methods above, there is no dependency on the persistence layer. 
The controller is completely unaware of how data is being persisted. This is exactly as it should be. 
Too often I see legacy code where the controller is interacting with the database directly. 
This is a very poor coding practice. It makes your code tightly coupled and hard to maintain.

### Code to an Interface
When using Spring to develop applications it is always best to develop to an interface, especially when leveraging the benefits of dependency injection. To support our controller actions, I wrote the following interface.

#### ProductService.java
```java

package com.aerospike.spring.boot.example.service;

import com.aerospike.spring.boot.example.domain.Product;

public interface ProductService {
    Iterable<Product> listAllProducts();

    Product getProductById(Integer id);

    Product saveProduct(Product product);

    void deleteProduct(Integer id);
}
```
At this point you cannot determine how it’s being persisted, which is the beauty of interfaces.

## Spring Data for Aerospike Product Service Implementation
In the previous section, we looked at using Spring Data for Aerospike. Now we need an implementation of the Product Service
which will use the Spring Data for Aerospike repositories.

### Spring Data for Aerospike Repository
We’ll need to inject an instance of the Spring Data for Aerospike repository into the implementation of our product service. 
You can do so by declaring a property for the repository and annotating the property with the @Autowired annotation.
```java
@Autowired
private ProductRepository productRepository;
}
```
#### List Products
Using Spring Data for Aerospike, it becomes trivial to list all the products for our application. 
While we did not actually create a `findAll()` method on the repository we defined, 
we inherited by extending the CrudRepository in Spring Data for Aerospike. 
This is one of many handy features of Spring Data for Aerospike. It’s going to provide us an implementation of the `findAll()` 
method, which we do not need to write code for.
```java
@Override
public Iterable<Product> listAllProducts() {
    return productRepository.findAll();
}
```
#### Get Product
To fetch a product by its id value, again, we can leverage a method implemented for us by Spring Data Aerospike.
```java
@Override
public Product getProductById(Integer id) {
     return productRepository.findOne(id);
}
```
#### Save Product
Spring Data Aerospike also provides us an implementation of a save method for saving entities. 
We use this method in creating and updating products in our web application.
```java
@Override
public Product saveProduct(Product product) {
      return productRepository.save(product);
}
```
#### Delete Product
Finally in our CRUD operations, Spring Data Aerospike provides us an implementation of a delete method. 
Spring Data Aerospike overloads the delete method, accepting just the ID value, or the entity itself. 
For our purposes, we are using the ID value to delete the desired entity.
```java
@Override
public void deleteProduct(Integer id) {
       productRepository.delete(id);
}
```
### Summary
In this example we implemented the CRUD operations using a CrudRepository supplied by Spring Data Aerospike. 
If you look at the code you will see all we did was extend the Spring Data for Aerospike CrudRepository to create our Product Repository. We did not define, nor implement an additional methods. We’re not declaring transactions. We’re not writing any SQL. I hope you can see the simplicity and time saving using tools like Spring Data for Aerospike can bring you.

## Thymeleaf Fragments
Thymeleaf fragments are a very powerful feature of Thymeleaf. They allow you to define repeatable chunks of code for your website. Once you define a Thymeleaf fragment, you can reuse it in other Thymeleaf templates. This works great for components you wish to reuse across your web pages.

In developing the Spring Boot Web Application, I found two uses for Thymeleaf templates. The first was common includes of the CSS, Javascript. The second was for a common menu I wanted to display at the top of each web page.

### Includes
Below is the Thymeleaf Fragment I’m using for the HTML header includes. You can see its a normal HTML document, using Thymeleaf tags to define the resources for a page.

#### headerinc.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en" th:fragment="head">
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<link
        href="http://cdn.jsdelivr.net/webjars/bootstrap/3.3.7/css/bootstrap.min.css"
        th:href="@{/webjars/bootstrap/3.3.7/css/bootstrap.min.css}"
        rel="stylesheet" media="screen" />

<script src="http://cdn.jsdelivr.net/webjars/jquery/2.1.4/jquery.min.js"
        th:src="@{/webjars/jquery/2.1.4/jquery.min.js}"></script>

<link href="../static/css/aerospike.css" th:href="@{/css/aerospike.css}"
        rel="stylesheet" media="screen" />
</head>
<body>
</body>
</html>
```
### Menu
For our Spring Boot Web Application, I chose to use the Bootstrap CSS framework. I’m big fan of Bootstrap. It’s easy to use, and its components look great. Bootstrap CSS has a menu component which I chose to use for the menu system.

In this Thymeleaf fragment, I’m providing the Bootstrap CSS menu I want to place at the top of all my pages. I also have a section to show my Spring Boot logo on each page.

#### header.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
</head>
<body>
    <div class="container">
        <div th:fragment="header">
            <nav class="navbar navbar-default">
                <div class="container-fluid">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="#" th:href="@{/}">Home</a>
                        <ul class="nav navbar-nav">
                            <li><a href="#" th:href="@{/products}">Products</a></li>
                            <li><a href="#" th:href="@{/product/new}">Create Product</a></li>
                        </ul>
                    </div>
                </div>
            </nav>
            <div class="jumbotron">
                <div class="row text-center">
                    <div class="">
                        <h2>Spring Framework Aerospike</h2>
                        <h3>Spring Boot Web App</h3>
                    </div>
                </div>
                <div class="row text-center">
                    <img src="../../static/images/Aerospike_Wallpaper_tdspb.jpg"
                            width="400" th:src="@{/images/Aerospike_Wallpaper_tdspb.jpg}" />
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```
## Including Thymeleaf Fragments
### Example
Previously, we defined an index page for our Spring Boot web application. You can apply Thymeleaf templates through the 
use of HTML comments. By doing this, you preserve the ability of the document to be viewed in the browser. 
You will be able to see the document okay in your browser, but the fragment portions will be omitted. 
The fragments are only included when the Thymeleaf template is rendered by Spring.

Remember, Spring will be reading the Thymeleaf templates, then producing output based upon the Thymeleaf directives.

#### index.html
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <title>Spring Boot Aerospike</title>
    <!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
    <div class="container">
        <!--/*/ <th:block th:include="fragments/header :: header"></th:block> /*/-->
    </div>
</body>
</html>
```
You can see how our index page is very simple now. While this is a very lean HTML document, when Spring renders it at run time, you will see HTML looking like this:

#### Actual HTML Rendered to Browser
```html
<!DOCTYPE html>

<html>
<head lang="en">

<title>Spring Boot Aerospike</title>


<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<link href="/webjars/bootstrap/3.3.4/css/bootstrap.min.css" rel="stylesheet" media="screen" />

<script src="/webjars/jquery/2.1.4/jquery.min.js"></script>

<link href="/css/aerospike.css" rel="stylesheet" media="screen" />

</head>
<body>

    <div class="container">
        <nav class="navbar navbar-default">
            <div class="container-fluid">
                <div class="navbar-header">
                    <a class="navbar-brand" href="/">Home</a>
                    <ul class="nav navbar-nav">
                        <li><a href="/products">Products</a></li>
                        <li><a href="/product/new">Create Product</a></li>
                    </ul>
                </div>
            </div>
        </nav>

        <div class="jumbotron">
            <div class="row text-center">
                <div class="">
                    <h2>Spring Framework Aerospike</h2>
                    <h3>Spring Boot Web App</h3>
                </div>
            </div>
            <div class="row text-center">
                <img src="/images/Aerospike_Wallpaper_tdspb.jpg" width="400" />
            </div>
        </div>

    </div>
</body>
</html>
```
Notice how Thymeleaf and Spring have merged the contents of the index.html document and the two Thymeleaf fragment documents? Now you have pure HTML, and Thymeleaf tags are not rendered to the HTML content sent to the browser.

The `index.html` Thymeleaf template will show this page in your browser.

![Index html](/docs/connectors/assets/images/Index_html.png)

## Thymeleaf Views for CRUD Application
### Show Product
Showing a product is one of the simpler operations under Spring MVC and Thymeleaf. 
Our controller returned a product object to the model and bound it to the property ‘product’. 
Now we can use the typical name-dot-property syntax to access properties of the product object.

This Thymeleaf tag:
```html
  <p class="form-control-static" th:text="${product.id}">Product Id</p></div>
```
Will get text from the description property of the product object and replace the ‘description’ text in the paragraph HTML tag.

Here is the full Thymeleaf template for showing a product:

#### productshow.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <title>Spring Boot Aerospike</title>
    <!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
<div class="container">
    <!--/*/ <th:block th:include="fragments/header :: header"></th:block> /*/-->

    <h2>Product Details</h2>
        <div>
            <form class="form-horizontal">
                <div class="form-group">
                    <label class="col-sm-2 control-label">Id:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.id}">Product Id</p></div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Product Id:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.productId}">productId</p>
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Description:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.description}">description</p>
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Price:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.price}">Priceaddd</p>
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Image Url:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.imageUrl}">url....</p>
                    </div>
                </div>
            </form>
    </div>
</div>

</body>
</html>
```
The show product Thymeleaf template will show this page:

![view html](/docs/connectors/assets/images/view_html.png)

### List Products
The list view is a little tricker because now we have a list of products to iterate over. Luckily, Thymeleaf makes this very easy to do.

Here is a snippet showing how to iterate over a list of products.
```html
 <tr th:each="product : ${products}">
    <td th:text="${product.id}"><a href="/product/${product.id}">Id</a></td>
    <td th:text="${product.productId}">Product Id</td>
    <td th:text="${product.description}">descirption</td>
    <td th:text="${product.price}">price</td>
    <td><a th:href="${'/product/' + product.id}">View</a></td>
    <td><a th:href="${'/product/edit/' + product.id}">Edit</a></td>
    <td><a th:href="${'/product/delete/' + product.id}">Delete</a></td>
</tr>
```
You can see the syntax of this Thymeleaf tag is similar to a for-each loop in Java.
```html
<tr th:each="product : ${products}">
```
Our controller added a list of products to the ‘products’ property to the model, which we pass to the Thymeleaf tag. The variable name we are assigning to the iterator is ‘product’.

The body of the each tag will be rendered once for each product in the list of products.

Here is the complete Thymeleaf template used for showing a list of products.

#### products.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <title>Spring Boot Aerospike</title>
    <!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
    <div class="container">
        <!--/*/ <th:block th:include="fragments/header :: header"></th:block> /*/-->
        <div th:if="${not #lists.isEmpty(products)}">
            <h2>Product List</h2>
            <table class="table table-striped">
                <tr>
                    <th>Id</th>
                    <th>Product Id</th>
                    <th>Description</th>
                    <th>Price</th>
                    <th>View</th>
                    <th>Edit</th>
                    <th>Delete</th>
                </tr>
                <tr th:each="product : ${products}">
                    <td th:text="${product.id}"><a href="/product/${product.id}">Id</a></td>
                    <td th:text="${product.productId}">Product Id</td>
                    <td th:text="${product.description}">descirption</td>
                    <td th:text="${product.price}">price</td>
                    <td><a th:href="${'/product/' + product.id}">View</a></td>
                    <td><a th:href="${'/product/edit/' + product.id}">Edit</a></td>
                    <td><a th:href="${'/product/delete/' + product.id}">Delete</a></td>
                </tr>
            </table>
        </div>
    </div>
</body>
</html>
```
Here is the Thymeleaf list products page:

![Products html](/docs/connectors/assets/images/Products_html.png)

### Create / Update Product
We can use the same HTML form for creating and updating products. A little trick is to have your controller method return an empty object to the view for the create option, and the existing object for the update option. By doing this you don’t need to worry about null objects on the view layer. For a new object, the null properties show up blank. For existing objects, non-null properties will get populated into the form fields.

The following line sets up the form in Thymeleaf.
```html
<form class="form-horizontal" th:object="${product}"
                                th:action="@{/product}" method="post">
```
The `th:object` tag binds the product object to the form. Thus, you only use the property names on the form fields. No need to qualify the object name too.

The “th:action” tag maps the form action to the `/product` url. And we specify to use the HTML post action for the form.

Here is the controller action this maps back to:
```java
@RequestMapping(value = "product", method = RequestMethod.POST)
public String saveProduct(Product product){
    productService.saveProduct(product);
    return "redirect:/product/" + product.getId();
}
```
Notice how we’ve assigned the url `product` and method POST in the request mapping.

#### productform.html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <title>Spring Boot Aerospike</title>
    <!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
    <div class="container">
        <!--/*/ <th:block th:include="fragments/header :: header"></th:block> /*/-->

        <h2>Product Details</h2>
        <div>
            <form class="form-horizontal" th:object="${product}"
                th:action="@{/product}" method="post">

                <div class="form-group">
                    <label class="col-sm-2 control-label">Id:</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" th:field="*{id}"
                            th:readonly="${product.id} != null" />
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Description:</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" th:field="*{description}" />
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Product Id:</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" th:field="*{productId}" />
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Price:</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" th:field="*{price}" />
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Image Url:</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" th:field="*{imageUrl}" />
                    </div>
                </div>
                <div class="row">
                    <button type="submit" class="btn btn-default">Submit</button>
                </div>
            </form>
        </div>
    </div>

</body>
</html>
```
Here is the Thymeleaf product form.

![edit html](/docs/connectors/assets/images/edit_html.png)

## Conclusion
In this section we built upon the previous sections to have a functional web application which performs CRUD operations against a single entity. At this point you can checkout the project from Github and build it using Maven. Spring Boot will create an executable JAR, which you can run to demo the application. Spring Boot will run the application in an embedded Apache Tomcat instance and you will be able to see the application running at http://localhost:8080.

### Get The Source!
The source code for this post is available on GitHub [here](https://github.com/aerospike/spring-boot-example)
