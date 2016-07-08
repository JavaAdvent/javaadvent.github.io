---
id: 295
title: Reactive Development Using Vert.x
date: 2015-12-05T02:23:56+00:00
author: Deven
layout: post
guid: http://www.javaadvent.com/?p=295
permalink: /2015/12/reactive-programming-with-vert-x.html
dsq_thread_id:
  - 4962579350
image: /content/uploads/2015/11/logo-sm.png
categories:
  - 2015
  - Actors
  - Dependency Injection
  - functional programming
  - java 8
  - Java Advent
  - lambda
  - Observable
  - reactive
  - Spring
  - spring boot
  - Uncategorized
tags:
  - concurrency
  - dependency injection
  - Java 8
  - lambda
  - reactive
  - spring
  - vertx
---
Lately, it seems like we're hearing about the latest and greatest frameworks for Java. Tools like <a href="http://www.ninjaframework.org/" target="_blank">Ninja</a>, <a href="http://sparkjava.com/" target="_blank">SparkJava</a>, and <a href="https://www.playframework.com/" target="_blank">Play</a>; but each one is opinionated and make you feel like you need to redesign your entire application to make use of their wonderful features. That's why I was so relieved when I discovered <a href="http://vertx.io/" target="_blank">Vert.x</a>. Vert.x isn't a framework, it's a toolkit and it's un-opinionated and it's liberating. Vert.x doesn't want you to redesign your entire application to make use of it, it just wants to make your life easier. Can you write your entire application in Vert.x? Sure! Can you add Vert.x capabilities to your existing Spring/Guice/CDI applications? Yep! Can you use Vert.x inside of your existing JavaEE applications? Absolutely! And that's what makes it amazing.
<h2>Background</h2>
Vert.x was born when Tim Fox decided that he liked a lot of what was being developed in the NodeJS ecosystem, but he&nbsp;didn't like some of the trade-offs of working in V8: Single-threadedness, limited library support, and JavaScript itself. Tim set out to write a toolkit which was unopinionated about how and where it is used, and he decided that the best place to implement it was on the JVM. So, Tim and the community set out to create an event-driven, non-blocking, reactive toolkit which in many ways mirrored what could be done in NodeJS, but also took advantage of the power available inside of the JVM. Node.x was born and it later progressed to become Vert.x.
<h2>Overview</h2>
Vert.x is designed to implement an event bus which is how different parts of the application can communicate in a non-blocking/thread safe manner. Parts of it were modeled after the Actor methodology exhibited&nbsp;by&nbsp;Eralng&nbsp;and Akka. It is also designed to take full advantage of today's multi-core processors and highly concurrent programming demands. As such, by default, all Vert.x <strong>VERTICLES</strong> are implemented as single-threaded by default. Unlike NodeJS though, Vert.x can run MANY verticles in MANY&nbsp;threads. Additionally, you can specify that some verticles are “worker” verticles and CAN be multi-threaded. And to really add some icing on the cake, Vert.x has low level support for multi-node clustering of the event bus via the use of Hazelcast. It has gone on to include many other amazing features which are too numerous to list here, but you can read more in the official Vert.x <a href="http://vertx.io/docs/" target="_blank">docs</a>.

The first thing you need to know about Vert.x is, similar to NodeJS, never block the current thread. Everything in Vert.x is set up, by default, to use callbacks/futures/promises. Instead of doing synchronous operations, Vert.x provides async methods for doing most I/O and processor intensive operations which might block the current thread. Now, callbacks can be ugly and painful to work with, so Vert.x optionally&nbsp;provides an <a href="http://vertx.io/docs/vertx-rx/java/" target="_blank">API based on RxJava</a> which implements the same functionality using the <a href="https://en.wikipedia.org/wiki/Observer_pattern" target="_blank">Observer pattern</a>. Finally, Vert.x makes it easy to use your existing classes and methods by providing the <b>executeBlocking(Function f) </b>method on many of it's asynchronous APIs. This means you can choose how you prefer to work with Vert.x instead of the toolkit dictating to you how it must be used.

The second thing to know about Vert.x is that it composed of verticles, modules, and nodes. Verticles are the smallest unit of logic in Vert.x, and are usually represented by a single class. Verticles should be simple and single-purpose following the <a href="https://en.wikipedia.org/wiki/Unix_philosophy" target="_blank">UNIX Philosophy</a>. A group of verticles can be put together into a module, which is usually packaged as a single JAR file. A module represents a group of related functionality which when taken together could represent an entire application or just a portion of a larger distributed application. Lastly, nodes are single instances of the JVM which are running one or more modules/verticles. Because Vert.x has clustering built-in from the ground up, Vert.x applications can span nodes either on a single machine or across multiple machines in multiple geographic locations (though latency can hider performance).
<h2>Example Project</h2>
Now, I've been to a number of Meetups and conferences lately where the first thing they show you when talking about reactive programming is to build a chat room application. That's all well and good, but it doesn't really help you to completely understand the power of reactive development. Chat room apps are simple and simplistic. We can do better. In this tutorial, we're going to take a legacy Spring application and convert it to take advantage of Vert.x. This has multiple purposes: It shows that the toolkit is easy to integrate with existing Java projects, it allows us to take advantage of existing tools which may be entrenched parts of our ecosystem, and it also lets us follow the <a href="https://en.wikipedia.org/wiki/Don%27t_repeat_yourself" target="_blank">DRY principle</a> in that we don't have to rewrite large swathes of code to get the benefits of Vert.x.

Our legacy Spring application is a contrived simple example of a REST API using Spring Boot, Spring Data JPA, and Spring REST. The source code can be found in the "master" branch <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example" target="_blank">HERE</a>. There are other branches which we will use to demonstrate the progression as we go, so it should be simple for anyone with a little experience with <strong>git</strong> and <strong>Java 8</strong> to follow along. Let's start by examining the Spring Configuration class for the stock Spring application.

<pre><tt>
@SpringBootApplication
@EnableJpaRepositories
@EnableTransactionManagement
@Slf4j
public class Application {
    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(Application.class, args);

        System.out.println("Let's inspect the beans provided by Spring Boot:");

        String[] beanNames = ctx.getBeanDefinitionNames();
        Arrays.sort(beanNames);
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    }

    @Bean
    public DataSource dataSource() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        return builder.setType(EmbeddedDatabaseType.HSQL).build();
    }

    @Bean
    public EntityManagerFactory entityManagerFactory() {
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setGenerateDdl(true);

        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setJpaVendorAdapter(vendorAdapter);
        factory.setPackagesToScan("com.zanclus.data.entities");
        factory.setDataSource(dataSource());
        factory.afterPropertiesSet();

        return factory.getObject();
    }

    @Bean
    public PlatformTransactionManager transactionManager(final EntityManagerFactory emf) {
        final JpaTransactionManager txManager = new JpaTransactionManager();
        txManager.setEntityManagerFactory(emf);
        return txManager;
    }
}
</tt></pre>

As you can see at the top of the class, we have some pretty standard Spring Boot annotations. You'll also see an @Slf4j annotation which is part of the <a href="https://projectlombok.org/" target="_blank">lombok</a> library, which is designed to help reduce boiler-plate code. We also have <strong>@Bean</strong> annotated methods for providing access to the JPA EntityManager, the TransactionManager, and DataSource. Each of these items provide injectable objects for the other classes to use. The remaining classes in the project are similarly simplistic. There is a <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/blob/master/src/main/java/com/zanclus/data/access/CustomerDAO.java" target="_blank">Customer</a> POJO which is the Entity type used in the service. There is a <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/blob/master/src/main/java/com/zanclus/data/entities/Customer.java" target="_blank">CustomerDAO</a> which is created via Spring Data. Finally, there is a <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/blob/master/src/main/java/com/zanclus/api/CustomerEndpoints.java" target="_blank">CustomerEndpoints</a> class which is the JAX-RS annotated REST controller.

As explained earlier, this is all standard fare in a Spring Boot application. The problem with this application is that for the most part, it has limited scalability. You would either run this application inside of a Servlet&nbsp;container, or with an embedded server like Jetty or Undertow. Either way, each requests ties up a thread and is thus wasting resources when it waits for I/O operations.

Switching over to the&nbsp;<a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/tree/Convert-To-Vert.x-Web" target="_blank">Convert-To-Vert.x-Web</a>&nbsp;branch, we can see that the Application class has changed a little. We now have some new @Bean annotated methods for injecting the <strong>Vertx</strong> instance itself, as well as an instance of <strong>ObjectMapper</strong> (part of the Jackson JSON library). We have also replaced the <strong>CustomerEnpoints</strong> class with a new <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/blob/Convert-To-Vert.x-Web/src/main/java/com/zanclus/api/CustomerVerticle.java">CustomerVerticle</a>. Pretty much everything else is the same.

The <strong>CustomerVerticle</strong> class&nbsp;is annotated with @Component, which means that Spring will instantiate&nbsp;that class on startup. It also has it's <strong>start</strong> method annotated with @PostConstruct so that the Verticle is launched on startup. Looking at the actual content of the code, we see our first bits of Vert.x code: <strong>Router</strong>.

The Router class is part of the vertx-web library and allows us to use a fluent API to define&nbsp;HTTP URLs, methods, and header filters for our request handling. Adding the <strong>BodyHandler</strong> instance to the default route allows a POST/PUT body to be processed and converted to a JSON object which Vert.x can then process as part of the RoutingContext. The order of routes in Vert.x CAN be significant. If you define a route which has some sort of glob matching (* or regex), it can swallow requests for routes defined after it unless you implement <a href="http://vertx.io/docs/vertx-web/java/#_handling_requests_and_calling_the_next_handler" target="_blank">chaining</a>. Our example shows 3 routes initially.
<pre><tt>
    @PostConstruct
    public void start() throws Exception {
        Router router = Router.router(vertx);
        router.route().handler(BodyHandler.create());
        router.get("/v1/customer/:id")
                .produces("application/json")
                .blockingHandler(this::getCustomerById);
        router.put("/v1/customer")
                .consumes("application/json")
                .produces("application/json")
                .blockingHandler(this::addCustomer);
        router.get("/v1/customer")
                .produces("application/json")
                .blockingHandler(this::getAllCustomers);
        vertx.createHttpServer().requestHandler(router::accept).listen(8080);
    }
</tt></pre>
Notice that the HTTP method is defined, the "Accept" header is defined (via consumes), and the "Content-Type" header is defined (via produces). We also see that we are passing the handling of the request off via a call to the <strong>blockingHandler</strong> method. A blocking handler for a Vert.x route accepts a RoutingContext object as it's only parameter. The RoutingContext holds the Vert.x Request object, Response object, and any parameters/POST body data (like ":id"). You'll also see that I used <a href="https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html" target="_blank">method references</a> rather than lambdas to insert the logic into the blockingHandler (I find it more readable). Each handler for the 3 request routes is defined in a separate method further down in the class. These methods basically just call the methods on the DAO, serialize or deserialize as needed, set some response headers, and end() the request by sending a response. Overall, pretty simple and straightforward.
<pre><tt>
    private void addCustomer(RoutingContext rc) {
        try {
            String body = rc.getBodyAsString();
            Customer customer = mapper.readValue(body, Customer.class);
            Customer saved = dao.save(customer);
            if (saved!=null) {
                rc.response().setStatusMessage("Accepted").setStatusCode(202).end(mapper.writeValueAsString(saved));
            } else {
                rc.response().setStatusMessage("Bad Request").setStatusCode(400).end("Bad Request");
            }
        } catch (IOException e) {
            rc.response().setStatusMessage("Server Error").setStatusCode(500).end("Server Error");
            log.error("Server error", e);
        }
    }

    private void getCustomerById(RoutingContext rc) {
        log.info("Request for single customer");
        Long id = Long.parseLong(rc.request().getParam("id"));
        try {
            Customer customer = dao.findOne(id);
            if (customer==null) {
                rc.response().setStatusMessage("Not Found").setStatusCode(404).end("Not Found");
            } else {
                rc.response().setStatusMessage("OK").setStatusCode(200).end(mapper.writeValueAsString(dao.findOne(id)));
            }
        } catch (JsonProcessingException jpe) {
            rc.response().setStatusMessage("Server Error").setStatusCode(500).end("Server Error");
            log.error("Server error", jpe);
        }
    }

    private void getAllCustomers(RoutingContext rc) {
        log.info("Request for all customers");
        List<Customer> customers = StreamSupport.stream(dao.findAll().spliterator(), false).collect(Collectors.toList());
        try {
            rc.response().setStatusMessage("OK").setStatusCode(200).end(mapper.writeValueAsString(customers));
        } catch (JsonProcessingException jpe) {
            rc.response().setStatusMessage("Server Error").setStatusCode(500).end("Server Error");
            log.error("Server error", jpe);
        }
    }
</tt></pre>
"But this is more code and messier than my Spring annotations and classes", you might say. That CAN be true, but it really depends on how you implement the code. This is meant to be an introductory example, so I left the code very simple and easy to follow. I COULD use an <a href="https://github.com/aesteve/nubes">annotation library</a> for Vert.x to implement the endpoints in a manner similar to JAX-RS. In addition, we have gained a massive scalability improvement. Under the hood, Vert.x Web uses <a href="http://netty.io/">Netty</a> for low-level asynchronous I/O operations, thus providing us the ability to handle MANY more concurrent requests (limited by the size of the database connection pool).

We've already made some improvement to the scalability and concurrency of this application by using the Vert.x Web library, but we can improve things a little more by implementing the Vert.x <a href="http://vertx.io/docs/vertx-core/java/#event_bus">EventBus</a>. By separating the database operations into <a href="http://vertx.io/docs/vertx-core/java/#worker_verticles">Worker Verticles</a> instead of using blockingHandler, we can handle request processing more efficiently. This is show in the <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/tree/Convert-To-Worker-Verticles">Convert-To-Worker-Verticles</a> branch. The application class has remained the same, but we have changed the <strong>CustomerEndpoints</strong> class and added a new class called <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/blob/Convert-To-Worker-Verticles/src/main/java/com/zanclus/verticles/CustomerWorker.java">CustomerWorker</a>. In addition, we added a new library called <a href="https://github.com/amoAHCP/spring-vertx-ext">Spring Vert.x Extension</a> which provides Spring Dependency Injections support to Vert.x Verticles. Start off by looking at the new <strong>CustomerEndpoints</strong> class.
<pre><tt>
    @PostConstruct
    public void start() throws Exception {
        log.info("Successfully create CustomerVerticle");
        DeploymentOptions deployOpts = new DeploymentOptions().setWorker(true).setMultiThreaded(true).setInstances(4);
        vertx.deployVerticle("java-spring:com.zanclus.verticles.CustomerWorker", deployOpts, res -> {
            if (res.succeeded()) {
                Router router = Router.router(vertx);
                router.route().handler(BodyHandler.create());
                final DeliveryOptions opts = new DeliveryOptions()
                        .setSendTimeout(2000);
                router.get("/v1/customer/:id")
                        .produces("application/json")
                        .handler(rc -> {
                            opts.addHeader("method", "getCustomer")
                                    .addHeader("id", rc.request().getParam("id"));
                            vertx.eventBus().send("com.zanclus.customer", null, opts, reply -> handleReply(reply, rc));
                        });
                router.put("/v1/customer")
                        .consumes("application/json")
                        .produces("application/json")
                        .handler(rc -> {
                            opts.addHeader("method", "addCustomer");
                            vertx.eventBus().send("com.zanclus.customer", rc.getBodyAsJson(), opts, reply -> handleReply(reply, rc));
                        });
                router.get("/v1/customer")
                        .produces("application/json")
                        .handler(rc -> {
                            opts.addHeader("method", "getAllCustomers");
                            vertx.eventBus().send("com.zanclus.customer", null, opts, reply -> handleReply(reply, rc));
                        });
                vertx.createHttpServer().requestHandler(router::accept).listen(8080);
            } else {
                log.error("Failed to deploy worker verticles.", res.cause());
            }
        });
    }
</tt></pre>
The routes are the same, but the implementation code is not. Instead of using calls to blockingHandler, we have now implemented proper async handlers which send out events on the event bus. None of the database processing is happening in this Verticle anymore. We have moved the database processing to a Worker Verticle which has multiple instances to handle multiple requests in parallel in a thread-safe manner. We are also registering a callback for when those events are replied to so that we can send the appropriate response to the client making the request. Now, in the <a href="https://github.com/JUGGL/Spring-Vert.x-Integration-Example/blob/Convert-To-Worker-Verticles/src/main/java/com/zanclus/verticles/CustomerWorker.java">CustomerWorker</a> Verticle we have implemented the database logic and error handling.

    @Override
    public void start() throws Exception {
        vertx.eventBus().consumer("com.zanclus.customer").handler(this::handleDatabaseRequest);
    }

    public void handleDatabaseRequest(Message<Object> msg) {
        String method = msg.headers().get("method");

        DeliveryOptions opts = new DeliveryOptions();
        try {
            String retVal;
            switch (method) {
                case "getAllCustomers":
                    retVal = mapper.writeValueAsString(dao.findAll());
                    msg.reply(retVal, opts);
                    break;
                case "getCustomer":
                    Long id = Long.parseLong(msg.headers().get("id"));
                    retVal = mapper.writeValueAsString(dao.findOne(id));
                    msg.reply(retVal);
                    break;
                case "addCustomer":
                    retVal = mapper.writeValueAsString(
                                        dao.save(
                                                mapper.readValue(
                                                        ((JsonObject)msg.body()).encode(), Customer.class)));
                    msg.reply(retVal);
                    break;
                default:
                    log.error("Invalid method '" + method + "'");
                    opts.addHeader("error", "Invalid method '" + method + "'");
                    msg.fail(1, "Invalid method");
            }
        } catch (IOException | NullPointerException e) {
            log.error("Problem parsing JSON data.", e);
            msg.fail(2, e.getLocalizedMessage());
        }
    }


The <strong>CustomerWorker</strong> worker verticles register a consumer for messages on the event bus. The string which represents the address on the event bus is arbitrary, but it is recommended to use a reverse-tld style naming structure so that it is simple to ensure that the addresses are unique ("com.zanclus.customer"). Whenever a new message is sent to that address, it will be delivered to one, and only one, of the worker verticles. The worker verticle then calls <strong>handleDatabaseRequest</strong> to do the database work, JSON serialization, and error handling.

There you have it. You've seen that Vert.x can be integrated into your legacy applications to improve concurrency and efficiency without having to rewrite the entire application. We could have done something similar with an existing Google Guice or JavaEE CDI application. All of the business logic could remain relatively untouched while we tried in Vert.x to add reactive capabilities. The next steps are up to you. Some ideas for where to go next include <a href="http://vertx.io/docs/vertx-core/java/#_cluster_managers">Clustering</a>, <a href="http://vertx.io/docs/vertx-core/java/#_websockets">WebSockets</a>, and <a href="http://vertx.io/docs/vertx-rx/java/">VertxRx</a> for ReactiveX sugar.