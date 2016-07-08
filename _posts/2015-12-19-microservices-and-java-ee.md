---
id: 723
title: Microservices and Java EE
date: 2015-12-19T01:01:23+00:00
author: Markus Eisele
layout: post
guid: http://www.javaadvent.com/?p=723
permalink: /2015/12/microservices-and-java-ee.html
categories:
  - 2015
  - JavaEE
  - microservices
tags:
  - Java EE
  - Microservices
  - WildFly Swarm
---
Microservices based architectures are everywhere these days. We learn so much about how today's innovators, like Netflix and Amazon use these to be even more successful in generating more business. But what about all of us, who are using Java EE application servers and write classical systems? Are we all doing it wrong? And how could we make our technical designs fit for the future?

<strong>Monoliths</strong>
First of all, let’s look into those classical systems. Or called monolithic applications. Even if the word has a bad smell these days, this is the way we built software for a very long time. It basically describes the fact, that we build individual applications to fulfill a certain functionality.
And monolithic does refer to exactly what Java EE or better the initial Java 2 Enterprise Edition was designed for. Centralized applications which could be scaled and clustered but not necessary build to be resilient by design. They relied on infrastructure and operations in failure scenarios most of the time.

Traditionally, Java EE applications followed some core patterns and were separated into three main layers: presentation, business, and integration. The presentation layer was packaged in Web Application Archives (WARs) while business and integration logic went into separate Java Archives (JARs). Bundled together as one deployment unit, a so-called Enterprise Archive (EAR) was created. The technology and best practices around Java EE have always been sufficient to build a well-designed monolith application. But most enterprise-grade projects tend to lose a close focus on architecture.  This is why sometimes a well designed spaghetti ball was the best way to visualize the project dependencies and internal structures. And when this happened, we’ve quickly experienced some significant drawbacks. Because everything was too coupled and integrated even making small changes required a lot of work (or sometimes major refactorings) and before putting the reworked parts into production, the applications also had to be tested with great care and from beginning to end.
The whole application was a lot more than just programmed artifacts: it also consisted of uncountable deployment descriptors and server configuration files, in addition to properties for relevant third-party environments.

The high risks in changes and the complexity of bringing new configurations into productions lead to less and less releases. A new release saw the light of day once or twice a year. Even the team structures were heavily influenced by these monolithic software architectures. The multi-month test cycle might have been the most visible proof. But besides that, projects with lifespans longer than five years tended to have huge bugs and feature databases. And if this wasn’t hard enough, the testing was barely qualified--no acceptance tests, and hardly any written business requirements or identifiable domains in design and usability.

Handling these kinds of enterprise projects was a multiple team effort and required a lot of people to oversee the entire project. From a software design perspective, the resulting applications had a very technical layering. Business components or domains were mostly driven by existing database designs or dated business object definitions. Our industry had to learn those lessons and we managed not only to keep these enterprise monoliths under control, but also invented new paradigms and methodologies to manage them even better.

<a href="http://www.javaadvent.com/content/uploads/2015/12/spaghetti.png" rel="attachment wp-att-726"><img class="size-medium wp-image-726 alignright" src="http://www.javaadvent.com/content/uploads/2015/12/spaghetti-300x218.png" alt="spaghetti" width="300" height="218" /></a>So, even if the word „monolith“ is considered synonym for a badly designed piece of software today, those architectures had a number of benefits. Monolithic applications are simple to develop since IDEs and other development tools are oriented around developing a single application. Its a single archive that can be shared with different teams and encapsulates all the functionality in there. Plus, the industry standard around Java EE gave enterprises solid access to the resources needed to not only build but also operate those applications. Software vendors have build a solid knowledge base around Java EE and sourcing isn't a big issue in general. And having worked with them since more than 15 years by now, the industry is finally able to manufacture these applications in a more or less productized and standardized way. We know which build tools to use, which processes scale in large teams and how to scale those applications. And even integration testing got a lot easier since tool like Arquillian emerged. We still are paying a price for the convenience of a mature solution like Java EE. Code-bases can grow very large. When applications stay in business for longer, they get more and more complex and harder to understand for the development teams. And even if we know how to configure application servers, the one or two special settings in each project still cause major headaches in operations.

<strong>Microservices
</strong><a href="http://www.javaadvent.com/content/uploads/2015/12/microservices.png" rel="attachment wp-att-728"><img class="size-medium wp-image-728 alignright" src="http://www.javaadvent.com/content/uploads/2015/12/microservices-300x254.png" alt="microservices" width="300" height="254" /></a>But our industry doesn’t stand still. And the next evolution of system architecture and design just saw the light of day a couple of years ago. With the growing complexity of centralized integration components and the additional overhead in the connected applications the search for something more lightweight and more resilient began. And the whole theory finally shifted away from big and heavyweight infrastructures and designs. Alongside with this IT departments started to revisit application servers alongside with wordy protocols and interface technologies.

The technical design went back to more handy artifacts and services with the proven impracticality of most of the service implementation in SOA- and ESB-based projects. Instead of intelligent routing and transformations, microservices use simple routes and encapsulate logic in the endpoint itself. And even if the name implies a defined size, there isn’t one.  Microservices are about having a single business purpose. And even more vexing for enterprise settings, the most effective runtime for microservices isn’t necessarily a full-blown application server. It might just be a servlet engine or that the JVM is already sufficient as an execution environment. With the growing runtime variations and the broader variety of programming language choices, this development turned into yet another operations nightmare. And even developers today are a little lost when it comes to defining microservices and how to apply this design to existing applications.

Microservices are designed to be small, stateless, in(ter)dependent &amp; fully contained applications. Ideally able to deploy them everywhere, because the deployment contains all the needed parts.

Microservices are designed to be small.  But defining "small" is subjective.  Some of the estimation techniques like lines of code, function points, use cases may be used.  But generally “small” isn’t about size.
In the book Building Microservices the author Sam Newman suggests a few techniques to define the size of microservice, they are:
<ul>
	<li>small enough to be owned by a small agile development team,</li>
	<li>re-writable within one or two agile sprints ( typically two to four weeks) or</li>
	<li>the complexity does not require to further divide the service up</li>
</ul>
A stateless application handles every request with the information contained only within it. Microservices must be stateless and it must service the request without remembering the previous communications from the external system.

Microservices must service the request independently, it may collaborate with other microservices within the eco-system.  For example, a microservice that generates a unique report after interacting with other microservices is an interdependent system. In this scenario, other microservices which only provide the necessary data to reporting microservices may be independent services. A full stack application is individually deploy-able. It has its own server, network &amp; hosting environment.  The business logic, data model and the service interface (API / UI) must be part of the entire system.  A Microservice must be a full stack application.

<strong>Why now? And why me?</strong>
"I’ve been going through enough already and the next Java EE version is already under development. We’re not even using latest Java EE 7. There are so many productivity features coming: I don’t care if I build a monolith if it just does the job and we can handle it." And I do understand these thoughts. I like Java EE as much as you probably do and I was really intrigued to find out why microservices evolved lately. The answer to those two questions might not be a simple one: But lets try:

Looking at all the problems in our industry and the still very high amount of projects failing, it is perfectly fine to understand the need to grow and eliminate problems. A big part of new hypes and revamped methodologies is the human will to grow.

And instead of “never touch a running system” our industry usually want’s to do something better than the last time.
So, to answer the second part of the question first: “You probably want to look into this, because not doing anything isn’t a solution.”

As a developer, architect or QA engineer we basically all signed up for live long learning. And I can only speak for myself at this point, but this is a very big part of why I like this job so much. The first part of the question isn’t so easy to answer.

Innovation and constant improvement are the drivers behind enterprises and enterprise-grade projects. Without innovation, there will be outdated and expensive infrastructure components (e.g., host systems) that are kept alive way longer than the software they are running was designed for. Without constant validation of the status quo, there will be implicit or explicit vendor lock-in. Aging middleware runs into extended support and only a few suppliers will still be able to provide know-how to develop for it. Platform stacks that stay behind the latest standards attempt to introduce quick and dirty solutions that ultimately produce technical debt. The most prominent and quickest moving projects in the microservices space are Open Source projects. Netflix OSS, Spring, Camel, Fabric8 and others are prominent examples. And it became a lot easier to operate polyglot full-stack applications with today’s PaaS offerings which are also backed by Open Source projects like Docker and Kubernetes. In our fast paced world the lead times for legally induced software changes or simple bug fixes are shrinking. Very few enterprises still have the luxury to work with month long production cycles and the need for software to generate real value for business emerges even more. And this is not only true for completely software driven companies like Uber, NetFlix, Amazon and others.

We need to build systems for flexibility and resiliency, not just efficiency and robustness.  And we need to start building them today with what we have.

And I really want to make sure you’re reading this statement the right way: I am not saying, that everything from today on is a microservice.
<ul>
	<li>But we should be aware of the areas where they can help and be able to</li>
	<li>change existing applications towards the new approach when it makes sense.</li>
	<li>and we want to be able to be a good consultant for those asking about the topic</li>
</ul>
And Java EE isn’t going anywhere soon. It will be complemented and the polyglot world will be moving in in various places, but we’re not going to get rid of it soon.  And this is the good news.

Learn more about how to evolve your Java EE applications into microservices by downloading my free eBook from <a href="http://developers.redhat.com/promotions/distributed-javaee-architecture/">developers.redhat.com</a>. Make sure to re-watch my O'Reilly Webcast about "<a href="http://www.oreilly.com/pub/e/3609">Java EE Microservices Architecture</a>"  and also follow my blog.eisele.net with some more technical information about <a href="http://blog.eisele.net/2015/10/wildfly-swarm-jax-rs-microservice-on-docker.html">WildFly Swarm</a>, Docker and Kubernetes with <a href="http://blog.eisele.net/2015/10/deploying-java-ee-microservices-on-openshift.html">OpenShift</a>.