---
id: 287
title: 'Quick Web App Prototyping with Spring Boot &#038; MongoDB'
date: 2015-12-16T01:00:23+00:00
author: Holger Steinhauer
excerpt: This article will show you how quick and easy you can create a fully fledged web application based on Spring Framework, Spring Boot, Spring Data and MongoDB. Thanks to Convention over Configuration, it is sometimes even enough to just add a dependency to tick off another requirement. So, come by and have a look.
layout: post
guid: http://www.javaadvent.com/?p=287
permalink: /2015/12/quick-web-app-prototyping-with-spring-boot-and-mongodb.html
categories:
  - 2015
  - java
  - Java Advent
  - lightweight
  - mongodb
  - Programming
  - REST
  - Spring
  - spring boot
---
Back in one of my previous projects I was asked to produce a little contingency application. The schedule was tight and the scope simple. The in-house coding standard is PHP, so trying to get a classic Java EE stack in place would have been a real challenge. And, to be really honest, completely oversized. So, what then? I took the chance and gave Spring a try. I used it before, but in old versions, hidden away in the tech stack of the portal software I was plagued with at this time.

My goal was to have something the WebOps can simply put on a server with Java installed and run it. No fiddling with dozens of XML configurations and memory fine tuning. Just as easy as <code>java -jar application.jar</code>.
It was the perfect call for "Spring Boot". This Spring project is all about making it easy to bring you, the developer, up to speed and take away the need of loads of configuration and boilerplate coding.

Another thing my project was crying for was a document-oriented data storage. I mean, the main purpose of the application was to offer a digital version of a real-world paper form. So why create a relational mess if we can represent the document as a document?! I used MongoDB in a couple of small projects before, so I decided to go with it.

What has this got to do with this article? Well, I will show you how quickly you can bring together all the bits and pieces needed for a web application. Spring Boot will make a lot of things fairly easy and will keep the code minimal. And at the end you will have a JAR file, which is executable and can be deployed by just dropping it onto a server. Your WebOps will love you for it.

Let's imagine we are about to create the next big product administration web application. As it is the next big thing, it needs a big name: <i>Productr</i> (this is the reason I am a software engineer and not in sales or marketing...).
Productr will do amazing things and this article will show you its early stages, which are:

<ul>
<li>providing a simple REST interface to query all available products</li>
<li>loading these products from a MongoDB</li>
<li>providing a production-ready monitoring facility</li>
<li>displaying all products by using a JavaScript UI</li>
</ul>

All you need to start is:

<ul>
<li>Java 8</li>
<li>Maven</li>
<li>Your favourite IDE (IntelliJ, Eclipse, vi, edlin, a butterfly...)</li>
<li>A browser (ok, or Internet Explorer / MS Edge, but who would really want this?!)</li>
</ul>

And for the impatient, the code is also <a href="https://github.com/daincredibleholg/quick-web-app-prototype">available on GitHub</a>.

<h2>Let's get started</h2>
Create a pom.xml with the following content:

<pre>
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"&gt;

	&lt;parent&gt;
	    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
	    &lt;artifactId&gt;spring-boot-starter-parent&lt;/artifactId&gt;
	    &lt;version&gt;1.3.0.RELEASE&lt;/version&gt;
	&lt;/parent&gt;

	&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
	&lt;groupId&gt;net.h0lg.tutorials.rapid&lt;/groupId&gt;
	&lt;artifactId&gt;rapid-resting&lt;/artifactId&gt;
	&lt;version&gt;1.0&lt;/version&gt;


	&lt;dependencies&gt;
	    &lt;dependency&gt;
	        &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
	        &lt;artifactId&gt;spring-boot-starter-web&lt;/artifactId&gt;
	    &lt;/dependency&gt;
	&lt;/dependencies&gt;


    &lt;build&gt;
        &lt;plugins&gt;
            &lt;plugin&gt;
                &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
                &lt;artifactId&gt;spring-boot-maven-plugin&lt;/artifactId&gt;
            &lt;/plugin&gt;
        &lt;/plugins&gt;
    &lt;/build&gt;
&lt;/project&gt;
</pre>

In these few lines a lot of stuff is already happening. Most important is the defined parent project. This will bring us a lot of useful and needed dependencies like logging, the Tomcat runtime and lots more. Thanks to Spring's modularity, everything is re-configurable via pom.xml or dependency injection. For getting everything up quickly the defaults are absolutely fine. (Convention over configuration, anybody?)

Now, create the obligatory Maven folder structure:
<code>
mkdir -p src/main/java src/main/resources src/test/java src/test/resources
</code>

And we are settled.

<h2>Start the engines</h2>
Let's get to work. We want to offer a REST interface to get access to our huge amount of products. So let's start with creating a REST collection available under <i>/api/products</i>. To do so we have to do a few things:

<ol>
<li>Our "data model" holding all information about our incredible products needs to be created</li>
<li>We need a controller offering a method which does everything necessary to answer a GET request</li>
<li>Create the main entry point for our application</li>
</ol>

The data model is pretty simple and done quickly. Just create a package called <i>demo.model</i> and a class called <i>Product</i> in it. The Product class is very straightforward:

<pre class="lang:java decode:true">
package demo.model;

import java.io.Serializable;

/**
 * Our very important and sophisticated data model
 */
public class Product implements Serializable {

    String productId;
    String name;
    String vendor;

    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getVendor() {
        return vendor;
    }

    public void setVendor(String vendor) {
        this.vendor = vendor;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Product product = (Product) o;

        if (getProductId() != null ? !getProductId().equals(product.getProductId()) : product.getProductId() != null)
            return false;
        if (getName() != null ? !getName().equals(product.getName()) : product.getName() != null) return false;
        return !(getVendor() != null ? !getVendor().equals(product.getVendor()) : product.getVendor() != null);

    }

    @Override
    public int hashCode() {
        int result = getProductId() != null ? getProductId().hashCode() : 0;
        result = 31 * result + (getName() != null ? getName().hashCode() : 0);
        result = 31 * result + (getVendor() != null ? getVendor().hashCode() : 0);
        return result;
    }
}</pre>

Our product has the incredible amount of 3 properties: an alphanumeric product ID, a name and a vendor (just the name, to keep things simple). It is serialisable and the getters, setters and the methods <i>equals()</i> &amp; <i>hashCode()</i> are implemented by using my IDE's code generation.

Alright, so creating a controller with a method to offer the GET listener it is now. Go back to your favourite IDE and create the package <i>demo.controller</i> and a class called <i>ProductsController</i> with the following content:

<pre class="lang:java decode:true">
package demo.controller;

import demo.model.Product;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * This controller provides the REST methods
 */
@RestController
@RequestMapping(value = "/", method = RequestMethod.GET)
public class ProductsController {

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public List<Product> getProducts() {
        List<Product> products = new ArrayList<Product>();

        return products;
    }

}</pre>

This is really everything you need to provide a REST interface. Ok, at the moment, an empty list is returned, but it is that easy to define.

The last thing missing is an entry point for our application. Just create a class called <i>Productr</i> in the package <i>demo</i> and give it the following content:

<pre class="lang:java decode:true">
package demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * This is the entry point of our application
 */
@SpringBootApplication
public class ProductrApplication {

    public static void main (String... opts) {
        SpringApplication.run(ProductrApplication.class, opts);
    }

}</pre>

Spring Boot saves us a lot of keystrokes. <i>@SpringBootApplication</i> does a few things we would need for every web application anyway. This annotation is shorthand for the following ones:

<ul>
  <li>@Configuration</li>
  <li>@EnableAutoConfiguration</li>
  <li>@ComponentScan</li>
</ul>

Now it is time to start our application for the first time. Thanks to Spring Boot's maven plugin, which we configured in our pom.xml, starting the application is as easy as: <code>mvn spring-boot:run</code>. Just run this command in your project root directory. You prefer the lazy point-n-click way provided by your IDE? Alright, just instruct your favourite IDE to run <i>ProductrApplication</i>.

Once it is started, use a browser, a REST client (you should check out <a href="http://www.getpostman.com/">Postman</a>, I love this tool) or a command line tool like <i>curl</i>. The address you are looking for is: <a href="http://localhost:8080/api/products/">http://localhost:8080/api/products/</a>. So, with <i>curl</i>, the command looks like this:

<code>
curl http://localhost:8080/api/products/
</code>

<h2>Data please</h2>
Ok, returning an empty list isn't that shiny, is it? So let's bring in data. 
In many projects a classic relational database is usually overkill (and painful if you have to use it AND scale out). This may be one reason for the hype around NoSQL databases. One (in my opinion good) example is MongoDB. 

Getting MongoDB up and running is pretty easy. On Linux you can use your package manager to install it. For Debian / Ubuntu, for example, simply do: <code>sudo apt-get install mongodb</code>.

For Mac, the easiest way is <i>homebrew</i>: <code>brew install mongodb</code> and follow the instructions in the "Caveats" section.

Windows users should go with the MongoDB installer (and toi toi toi).

Alright, we just got our data store sorted. It is about time to use it.
There is one particular Spring project dealing with data - called <a href="http://projects.spring.io/spring-data/">Spring Data</a>. And by sheer coincidence a sub-project called <a href="http://projects.spring.io/spring-data-mongodb/">Spring Data MongoDB</a> is just waiting for us. Even more, Spring Boot provides a dependency package to get up to speed instantly. No wonder that the following few lines in the <i>pom.xml</i>'s <code>&lt;dependencies&gt;</code> section are enough to bring in everything we need:

<code>
&nbsp;&nbsp;&lt;dependency&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;spring-boot-starter-data-mongodb&lt;/artifactId&gt;
&nbsp;&nbsp;&lt;/dependency&gt;
</code>

Now, create a new package called <i>demo.domain</i> and put in a new interface called <i>ProductRepository</i>. Spring provides a pretty neat way to get rid of writing code which is usually needed to interact with a data source. Most of the basic queries are generated by Spring Data - all you need is to define an interface. A couple of query methods are available without even specifying method headers. One example is the <code>findAll()</code> method, which will return all entries in the collection.
But hey, let's see it in action instead of talking about it. The bespoke <i>ProductRepository</i> interface should look like this:

<pre class="lang:java decode:true">
package demo.domain;

import demo.model.Product;
import org.springframework.data.mongodb.repository.MongoRepository;

/**
 * This interface lets Spring generate a whole Repository implementation for
 * Products.
 */
public interface ProductRepository extends MongoRepository<Product, String> {

}</pre>

Next, create a class called <i>ProductService</i> in the same package. Purpose of this class is to actually provide some useful methods to query products. For now, the code is as easy as this:

<pre class="lang:java decode:true">
package demo.domain;

import demo.model.Product;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * This is a little service class we will let Spring inject later.
 */
@Service
public class ProductService {

    @Autowired
    private ProductRepository repository;

    public List<Product> getProducts() {
        return repository.findAll();
    }

}</pre>

See how we can use <code>repository.findAll()</code> without even defining it in the interface? Pretty slick, isn't it? Especially if you are in a hurry and need to get things up quickly.

Alright, so far we prepared the foundation for the data access. I think it is time to wire it together. To do so, simply head back to our class <code>demo.controller.ProductsController</code> and modify it slightly. All we have to do is to inject our shiny new <i>ProductService</i> service and call its <code>getProducts()</code> method. The class will look like this afterwards:

<pre class="lang:java decode:true">
package demo.controller;

import demo.domain.ProductService;
import demo.model.Product;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * This controller provides the REST methods
 */
@RestController
@RequestMapping("/api/products/")
public class ProductsController {

    // Let Spring DI inject the service for us
    @Autowired
    private ProductService productService;

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public List<Product> getProducts() {
        // Ask the data store for a list of products
        return productService.getProducts();
    }

}</pre>

That's it. Start MongoDB (if not already running), start our application again (remember the <code>mvn spring-boot:run</code> thingy?!) and start another GET request to <a href="http://localhost:8080/api/products/">http://localhost:8080/api/products/</a>:

<code>
$ curl http://localhost:8080/api/products/
[]
</code>

Wait, still an empty list? Yes, or do you remember us putting anything into the database? Let's change this by using the following command:

<code>
mongo localhost/test --eval "db.product.insert({productId: 'a1234', name: 'Our First Product', vendor: 'ACME'})"
</code>

This adds one product called "Our First Product" to our database. Ok, so what is our service returning now? This:

<pre>
$ curl http://localhost:8080/api/products/
[{"productId":"5657654426ed9d921affc3c0","name":"Our First Product","vendor":"ACME"}]
</pre>

Easy, wasn't it?!

Looking for a little more data but no time to create it yourself? Alright, it's nearly Christmas, so take my little test selection:

<pre class="lang:bash decode:true">
curl https://gist.githubusercontent.com/daincredibleholg/f8667a26ce2f17776903/raw/ed9b4c8ec6c9c455dc063e833af2418648928ba6/quick-web-app-product-example.json | mongoimport -d test -c product --jsonArray
</pre>

<h2>Basic requirements at your fingertips</h2>
In today's hectic days and with "microservice" culture spreading, it is getting harder and harder to keep an eye on what is really running on your servers or cloud environments. So in nearly all environments I was working on over the last years monitoring was a big thing. One common pattern is to provide health check endpoints. One can find everything from simple ping endpoints to health metrics, returning a detailed overview of business relevant metrics.
All of this is most of the times a copy-n-paste adventure and involves tackling a lot of boilerplate code. Here is what we have to do - simply add the following dependency to your pom.xml:

<code>
&nbsp;&nbsp;&lt;dependency&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;spring-boot-starter-actuator&lt;/artifactId&gt;
&nbsp;&nbsp;&lt;/dependency&gt;
</code>

and restart the service. Let's have a look what happens if we query http://localhost:8080/health:

<code>
$ curl http://localhost:8080/health
{"status":"UP","diskSpace":{"status":"UP","total":499088621568,"free":83261571072,"threshold":10485760},"mongo":{"status":"UP","version":"3.0.7"}}
</code>

This should provide sufficient data for a basic health check. If you follow the startup log messages, you'll probably spotted a number of other endpoints. Experiment a bit and check the <a href="https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html">Actuator documentation</a> for more information.

<h2>Show it to me</h2>
Ok, we got ourselves a REST service and some data. But we want to show this data to our users. So let's go on and provide a page with an overview of our awesome products.

Thank Santa that there is a really active web UI community working on loads of nice and easy usable frontend frameworks and libraries. One pretty popular example is <a href="https://getbootstrap.com/">Bootstrap</a>. It is easy to use and all the needed bits and pieces are provided via open CDNs.

We want to have a short overview of our products, so a table view would be nice. <a href="http://bootstrap-table.wenzhixin.net.cn/">Bootstrap Table</a> will help us with that. It is built on top of Bootstrap and also available via CDNs. What a world we live in...

But wait, where to put our HTML file? Spring Boot makes it easy, again. Just create a folder called <i>src/main/resources/static</i> and create a new HTML file called <i>index.html</i> with the following content:

<pre>
&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
	&lt;meta charset="utf-8"&gt;
    &lt;meta http-equiv="X-UA-Compatible" content="IE=edge"&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1"&gt;

	&lt;title&gt;Productr&lt;/title&gt;

	&lt;!-- Import Bootstrap CSS from CDNs --&gt;
	&lt;link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"&gt;
	&lt;link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/bootstrap-table/1.9.1/bootstrap-table.min.css"&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;nav class="navbar navbar-inverse"&gt;
	&lt;div class="container"&gt;
		&lt;div class="navbar-header"&gt;
			&lt;button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar"&gt;
				&lt;span class="sr-only"&gt;Toggle navigation&lt;/span&gt;
				&lt;span class="icon-bar"&gt;&lt;/span&gt;
				&lt;span class="icon-bar"&gt;&lt;/span&gt;
				&lt;span class="icon-bar"&gt;&lt;/span&gt;
			&lt;/button&gt;
			&lt;a class="navbar-brand" href="#"&gt;Productr&lt;/a&gt;
		&lt;/div&gt;
		&lt;div id="navbar" class="collapse navbar-collapse"&gt;
			&lt;ul class="nav navbar-nav"&gt;
				&lt;li class="active"&gt;&lt;a href="#"&gt;Home&lt;/a&gt;&lt;/li&gt;
				&lt;li&gt;&lt;a href="#about"&gt;About&lt;/a&gt;&lt;/li&gt;
				&lt;li&gt;&lt;a href="#contact"&gt;Contact&lt;/a&gt;&lt;/li&gt;
			&lt;/ul&gt;
		&lt;/div&gt;&lt;!--/.nav-collapse --&gt;
	&lt;/div&gt;
&lt;/nav&gt;
	&lt;div class="container"&gt;
		&lt;table data-toggle="table" data-url="/api/products/"&gt;
			&lt;thead&gt;
			&lt;tr&gt;
				&lt;th data-field="productId"&gt;Product Reference&lt;/th&gt;
				&lt;th data-field="name"&gt;Name&lt;/th&gt;
				&lt;th data-field="vendor"&gt;Vendor&lt;/th&gt;
			&lt;/tr&gt;
			&lt;/thead&gt;
		&lt;/table&gt;
	&lt;/div&gt;


&lt;!-- Import Bootstrap, Bootstrap Table and JQuery JS from CDNs --&gt;
    &lt;script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"&gt;&lt;/script&gt;
	&lt;script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"&gt;&lt;/script&gt;
	&lt;script src="//cdnjs.cloudflare.com/ajax/libs/bootstrap-table/1.9.1/bootstrap-table.min.js"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
</pre>

This file isn't pretty complex. It is just a HTML file, which includes the minimised CSS files from the CDNs. If you see a reference like <code>//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css</code> for the first time, it is not a bad mistake that the protocol (http or https) is missing. A resource referenced that way will be loaded via the same protocol the main page got loaded with. Say, if you use <i>http://localhost:8080/</i>, it will use <i>http:</i> to load the CSS files.

The <i>&lt;body&gt;</i> block contains a navigation bar (using the HTML5 <i>&lt;nav&gt;</i> tag) and a table. The interesting part of this table definition is the provided <i>data-url</i> attribute. It is interpreted by Bootstrap Table to load the data. Our definition points to our previously created REST endpoint.
Which part of our JSON objects is used in which column is defined via the <i>data-field</i> attributes on the <i>&lt;th&gt;</i> definitions. Can you spot the matching attribute names?

Last but not least we load the needed JavaScript libraries. All Bootstrap-related JavaScript functionality needs JQuery, so this is the first library to load. Followed straight by the main Bootstrap and the Bootstrap Table JavaScript files. Each of these library files is loaded in the minimised version, to keep download times at a minimum.

<h2>Where to go now</h2>
It is fair to say that we have a really simple web application now. Well, the main purpose of this article was to show you how to get up to speed with as little code as possible. You've seen that sometimes just a dependency in your POM file brings you a complete new feature, without the need of any additional line of code.
Take a step back, look at what we've built so far and think about the next steps needed. And just start to take a look around in the Spring universe. 

I think one of the most crucial steps needed next, beside adding the missing tests, is to bring in security. Check out <a href="http://projects.spring.io/spring-security">Spring Security</a> and its subprojects <a href="http://projects.spring.io/spring-security-oauth/">Spring Security OAuth</a>.
More interested in "classic" web pages? Check out <a href="http://projects.spring.io/spring-framework">Spring MVC</a> and how easy it is to integrate quite sophisticated template engines (e. g. by following <a href="http://spring.io/guides/gs/serving-web-content/">this guide</a>).

Hopefully, you enjoyed this article as much as I enjoyed its creation. I wish you all a merry Christmas and if the one or the other wants to get in touch, you can find me e. g. on <a href="https://twitter.com/incredibleh0lg">Twitter</a>, <a href="https://plus.google.com/u/0/+HolgerSteinhauer">G+</a> and <a href="https://uk.linkedin.com/in/holgersteinhauer">LinkedIn</a>.
