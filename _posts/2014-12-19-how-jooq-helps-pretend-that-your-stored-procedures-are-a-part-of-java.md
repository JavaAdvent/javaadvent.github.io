---
id: 14
title: How jOOQ Helps Pretend that Your Stored Procedures are a Part of Java
date: 2014-12-19T08:30:00+00:00
author: Lukas Eder
layout: post
permalink: /2014/12/how-jooq-helps-pretend-that-your-stored-procedures-are-a-part-of-java.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Lukas Eder
blogger_permalink:
  - /2014/12/how-jooq-helps-pretend-that-your-stored.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/6757632038019975486
image: /content/uploads/2014/12/jooq-the-best-way-to-write-sql-in-java-small.png
categories:
  - CallableStatement
  - java
  - Java Advent
  - java advent 2014
  - JDBC
  - jooq
  - PL/SQL
  - SAP HANA
  - stored procedure
  - T-SQL
---
In this year's Java Advent Calendar, we're thrilled to have been asked to feature a mini-series showing you a couple of advanced and very interesting topics that we've been working on when developing <a href="http://www.jooq.org/" title="jOOQ: The best way to write SQL in Java">jOOQ</a>.<br /><br />The series consists of:<br /><br /><ul><li><a href="http://www.javaadvent.com/2014/12/how-jooq-leverages-generic-type-safety-in-its-dsl/">Dec 17: How jOOQ Leverages Generic Type Safety in its DSL</a></li><li><a href="http://www.javaadvent.com/2014/12/how-jooq-allows-for-fluent-functional-relational-interactions-in-java-8/">Dec 18: How jOOQ Allows for Fluent Functional-Relational Interactions in Java 8</a></li><li><a href="http://www.javaadvent.com/2014/12/how-jooq-helps-pretend-that-your-stored-procedures-are-a-part-of-java/">Dec 19: How jOOQ Helps Pretend that Your Stored Procedures are a Part of Java</a></li></ul><br />Don't miss any of these!<br /><br /><h2 style="font-size: 2em">How jOOQ helps pretend that your stored procedures are a part of Java</h2><br /><sub><a href="http://blog.jooq.org/2014/11/04/painless-access-from-java-to-plsql-procedures-with-jooq/">This article was originally published fully on the jOOQ blog</a></sub><br /><br />Stored procedures are an interesting way to approach data processing. Some Java developers tend to stay clear of them for rather dogmatic reasons, such as:<br /><br /><ul><li>They think that the database is the wrong place for business logic</li><li>They think that the procedural aspect of those languages is ill-suited for their domain</li></ul><br />But in practice, stored procedures are an excellent means of handling data manipulations simply for the fact that they can execute complex logic right where the data is. This completely removes all effects that network latency and bandwidth will have on your application, otherwise. As we're looking into supporting SAP HANA for jOOQ 3.6, we can tell you that running jOOQ's 10000 integration test queries connecting from a local machine to the cloud takes a lot longer. If you absolutely want to stay in Java land, then you better also deploy your Java application into the cloud, close to that database (SAP HANA obviously offers that feature). But much better than that, move some of the logic into the database!<br /><br />If you're doing calculations on huge in-memory data sets, you should better get your code into that same memory, rather than shuffling around memory pieces between possibly separate physical memory addresses. Companies like <a href="http://hazelcast.com/">Hazelcast</a> essentially do the same, except that their in-memory database is written in Java, so you can also write your "stored procedure" in Java.<br /><br />With SQL databases, procedural SQL languages are king. And because of their tight integration with SQL, they're much superior for the job than any Java based stored procedure architecture.<br /><br /><h2 style="font-size: 2em">I knoow, but JDBC's CallableStatement... Arrrgh!</h2><br />Yes. As ever so often (and as mentioned before in our <a href="http://www.javaadvent.com/2014/12/how-jooq-leverages-generic-type-safety-in-its-dsl/">previous articles</a>, one very important reason why many Java developers don't like working with SQL is JDBC. <a href="http://www.jooq.org/hacking-jdbc">Binding to a database via JDBC is extremely tedious</a> and keeps us from working efficiently. Let's have a look at a couple of PL/SQL binding examples:<br /><br />Assume we're working on an <a href="https://github.com/jOOQ/jOOQ/tree/master/jOOQ-examples/Sakila/oracle-sakila-db">Oracle-port of the popular Sakila database</a> (<a href="http://dev.mysql.com/doc/sakila/en/">originally created for MySQL</a>). This particular Sakila/Oracle port was implemented by <a href="http://www.etl-tools.com/">DB Software Laboratory</a> and published under the BSD license.<br /><br />Here's a partial view of that Sakila database.<br /><br /><div style="clear: both; text-align: center;"><a href="http://3.bp.blogspot.com/-gTFY22YaYAk/VInCWK6W_yI/AAAAAAAABNc/aqYFxqhr6m4/s1600/sakila-film-actor-category%5B1%5D.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://3.bp.blogspot.com/-gTFY22YaYAk/VInCWK6W_yI/AAAAAAAABNc/aqYFxqhr6m4/s1600/sakila-film-actor-category%5B1%5D.png" /></a></div><br /><sub><a href="http://www.vertabelo.com/">ERD created with vertabelo.com</a> - <a title="Stop Manually Importing Your ERD Export into jOOQ" href="http://blog.jooq.org/2014/09/05/importing-your-erd-export-into-jooq/">learn how to use Vertabelo with jOOQ</a></sub><br /><br />Now, let's assume that we have an API in the database that doesn't expose the above schema, but <a href="https://github.com/jOOQ/jOOQ/blob/master/jOOQ-examples/Sakila/oracle-sakila-db/oracle-sakila-schema-pl-sql.sql">exposes a PL/SQL API instead</a>. The API might look something like this:<br /><br /><code><pre>CREATE TYPE LANGUAGE_T AS OBJECT (<br />  language_id SMALLINT,<br />  name CHAR(20),<br />  last_update DATE<br />);<br />/<br /><br />CREATE TYPE LANGUAGES_T AS TABLE OF LANGUAGE_T;<br />/<br /><br />CREATE TYPE FILM_T AS OBJECT (<br />  film_id int,<br />  title VARCHAR(255),<br />  description CLOB,<br />  release_year VARCHAR(4),<br />  language LANGUAGE_T,<br />  original_language LANGUAGE_T,<br />  rental_duration SMALLINT,<br />  rental_rate DECIMAL(4,2),<br />  length SMALLINT,<br />  replacement_cost DECIMAL(5,2),<br />  rating VARCHAR(10),<br />  special_features VARCHAR(100),<br />  last_update DATE<br />);<br />/<br /><br />CREATE TYPE FILMS_T AS TABLE OF FILM_T;<br />/<br /><br />CREATE TYPE ACTOR_T AS OBJECT (<br />  actor_id numeric,<br />  first_name VARCHAR(45),<br />  last_name VARCHAR(45),<br />  last_update DATE<br />);<br />/<br /><br />CREATE TYPE ACTORS_T AS TABLE OF ACTOR_T;<br />/<br /><br />CREATE TYPE CATEGORY_T AS OBJECT (<br />  category_id SMALLINT,<br />  name VARCHAR(25),<br />  last_update DATE<br />);<br />/<br /><br />CREATE TYPE CATEGORIES_T AS TABLE OF CATEGORY_T;<br />/<br /><br />CREATE TYPE FILM_INFO_T AS OBJECT (<br />  film FILM_T,<br />  actors ACTORS_T,<br />  categories CATEGORIES_T<br />);<br />/<br /></pre></code><br /><br />You'll notice immediately, that this is essentially just a 1:1 copy of the schema in this case modelled as Oracle SQL <code>OBJECT</code> and <code>TABLE</code> types, apart from the <code>FILM_INFO_T</code> type, which acts as an aggregate.<br /><br />Now, our <a href="https://twitter.com/JavaOOQ/status/540422773261500416">DBA</a> (or our database developer) has implemented the following API for us to access the above information:<br /><br /><code><pre>CREATE OR REPLACE PACKAGE RENTALS AS<br />  FUNCTION GET_ACTOR(p_actor_id INT) RETURN ACTOR_T;<br />  FUNCTION GET_ACTORS RETURN ACTORS_T;<br />  FUNCTION GET_FILM(p_film_id INT) RETURN FILM_T;<br />  FUNCTION GET_FILMS RETURN FILMS_T;<br />  FUNCTION GET_FILM_INFO(p_film_id INT) RETURN FILM_INFO_T;<br />  FUNCTION GET_FILM_INFO(p_film FILM_T) RETURN FILM_INFO_T;<br />END RENTALS;<br />/<br /></pre></code><br /><br />This, ladies and gentlemen, is how you can now...<br /><br /><h2 style="font-size: 2em">... tediously access the PL/SQL API with JDBC</h2><br />So, in order to avoid the awkward <a href="http://docs.oracle.com/javase/8/docs/api/java/sql/CallableStatement.html">CallableStatement</a> with its <code>OUT</code> parameter registration and JDBC escape syntax, we're going to fetch a <code>FILM_INFO_T</code> record via a SQL statement like this:<br /><br /><code><pre>try (PreparedStatement stmt = conn.prepareStatement(<br />        &quot;SELECT rentals.get_film_info(1) FROM DUAL&quot;);<br />     ResultSet rs = stmt.executeQuery()) {<br /><br />    // STRUCT unnesting here...<br />}<br /></pre></code><br /><br />So far so good. Luckily, there is Java 7's try-with-resources to help us clean up those myriad JDBC objects. Now how to proceed? What will we get back from this <code>ResultSet</code>? A <a href="http://docs.oracle.com/javase/8/docs/api/java/sql/Struct.html"><code>java.sql.Struct</code></a>:<br /><br /><code><pre>while (rs.next()) {<br />    Struct film_info_t = (Struct) rs.getObject(1);<br /><br />    // And so on...<br />}<br /></pre></code><br /><br />Now, the brave ones among you would continue downcasting the <code>java.sql.Struct</code> to an even more obscure and arcane <a href="http://docs.oracle.com/database/121/JAJDB/oracle/sql/STRUCT.html"><code>oracle.sql.STRUCT</code></a>, which contains almost no Javadoc, but tons of deprecated additional, vendor-specific methods.<br /><br />For now, let's stick with the "standard API", though. Let's continue navigating our <code>STRUCT</code><br /><br /><code><pre>while (rs.next()) {<br />    Struct film_info_t = (Struct) rs.getObject(1);<br /><br />    Struct film_t = (Struct) film_info_t.getAttributes()[0];<br />    String title = (String) film_t.getAttributes()[1];<br />    Clob description_clob = (Clob) film_t.getAttributes()[2];<br />    String description = description_clob.getSubString(1, (int) description_clob.length());<br /><br />    Struct language_t = (Struct) film_t.getAttributes()[4];<br />    String language = (String) language_t.getAttributes()[1];<br /><br />    System.out.println(&quot;Film       : &quot; + title);<br />    System.out.println(&quot;Description: &quot; + description);<br />    System.out.println(&quot;Language   : &quot; + language);<br />}<br /></pre></code><br /><br />This could go on and on. The pain has only started, we haven't even covered arrays yet. <a href="http://blog.jooq.org/2014/11/04/painless-access-from-java-to-plsql-procedures-with-jooq/">The details can be seen here in the original article</a>.<br /><br />Anyway. Now that we've finally achieved this, we can see the print output:<br /><br /><pre>Film       : ACADEMY DINOSAUR<br />Description: A Epic Drama of a Feminist And a Mad <br />             Scientist who must Battle a Teacher in<br />             The Canadian Rockies<br />Language   : English             <br />Actors     : <br />  PENELOPE GUINESS<br />  CHRISTIAN GABLE<br />  LUCILLE TRACY<br />  SANDRA PECK<br />  JOHNNY CAGE<br />  MENA TEMPLE<br />  WARREN NOLTE<br />  OPRAH KILMER<br />  ROCK DUKAKIS<br />  MARY KEITEL<br /></pre><h2 style="font-size: 2em">When will this madness stop?</h2><strong>It'll stop right here!</strong><br /><br />So far, this article read like a tutorial (or rather: medieval torture) of how to deserialise nested user-defined types from Oracle SQL to Java (don't get me started on serialising them again!)<br /><br />In the next section, we'll see how the exact same business logic (listing Film with ID=1 and its actors) can be implemented with no pain at all using <a title="jOOQ generates Java code from your database and lets you build type safe SQL queries through its fluent API." href="http://www.jooq.org/">jOOQ and its source code generator</a>. Check this out:<br /><br /><code><pre>// Simply call the packaged stored function from<br />// Java, and get a deserialised, type safe record<br />FilmInfoTRecord film_info_t = Rentals.getFilmInfo1(configuration, new BigInteger(&quot;1&quot;));<br /><br />// The generated record has getters (and setters)<br />// for type safe navigation of nested structures<br />FilmTRecord film_t = film_info_t.getFilm();<br /><br />// In fact, all these types have generated getters:<br />System.out.println(&quot;Film       : &quot; + film_t.getTitle());<br />System.out.println(&quot;Description: &quot; + film_t.getDescription());<br />System.out.println(&quot;Language   : &quot; + film_t.getLanguage().getName());<br /><br />// Simply loop nested type safe array structures<br />System.out.println(&quot;Actors     : &quot;);<br />for (ActorTRecord actor_t : film_info_t.getActors()) {<br />    System.out.println(<br />        &quot;  &quot; + actor_t.getFirstName()<br />       + &quot; &quot; + actor_t.getLastName());<br />}<br /><br />System.out.println(&quot;Categories     : &quot;);<br />for (CategoryTRecord category_t : film_info_t.getCategories()) {<br />    System.out.println(category_t.getName());<br />}<br /></pre></code><br /><br /><strong>Is that it?</strong><br /><br />Yes!<br /><br />Wow, I mean, this is just as though all those PL/SQL types and procedures / functions were actually part of Java. All the caveats that we've seen before are hidden behind those generated types and implemented in <a title="jOOQ generates Java code from your database and lets you build type safe SQL queries through its fluent API." href="http://www.jooq.org/">jOOQ</a>, so you can concentrate on what you originally wanted to do. Access the data objects and do meaningful work with them. Not serialise / deserialise them!<br /><br /><div style="clear: both; text-align: center;"><a href="http://www.jooq.org/"><img src="http://www.jooq.org/img/jooq-the-best-way-to-write-sql-in-java-small.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;" /></a></div><br /><h2 style="font-size: 2em">Not convinced yet?</h2>I told you not to get me started on serialising the types to JDBC. And I won't, but here's how to serialise the types to jOOQ, because that's a piece of cake!<br /><br />Let's consider this other aggregate type, that returns a customer's rental history:<br /><br /><code><pre>CREATE TYPE CUSTOMER_RENTAL_HISTORY_T AS OBJECT (<br />  customer CUSTOMER_T,<br />  films FILMS_T<br />);<br />/<br /></pre></code><br /><br />And the full PL/SQL package specs:<br /><br /><code><pre>CREATE OR REPLACE PACKAGE RENTALS AS<br />  FUNCTION GET_ACTOR(p_actor_id INT) RETURN ACTOR_T;<br />  FUNCTION GET_ACTORS RETURN ACTORS_T;<br />  FUNCTION GET_CUSTOMER(p_customer_id INT) RETURN CUSTOMER_T;<br />  FUNCTION GET_CUSTOMERS RETURN CUSTOMERS_T;<br />  FUNCTION GET_FILM(p_film_id INT) RETURN FILM_T;<br />  FUNCTION GET_FILMS RETURN FILMS_T;<br />  FUNCTION GET_CUSTOMER_RENTAL_HISTORY(p_customer_id INT) RETURN CUSTOMER_RENTAL_HISTORY_T;<br />  FUNCTION GET_CUSTOMER_RENTAL_HISTORY(p_customer CUSTOMER_T) RETURN CUSTOMER_RENTAL_HISTORY_T;<br />  FUNCTION GET_FILM_INFO(p_film_id INT) RETURN FILM_INFO_T;<br />  FUNCTION GET_FILM_INFO(p_film FILM_T) RETURN FILM_INFO_T;<br />END RENTALS;<br />/<br /></pre></code><br /><br />So, when calling <code>RENTALS.GET_CUSTOMER_RENTAL_HISTORY</code> we can find all the films that a customer has ever rented. Let's do that for all customers whose <code>FIRST_NAME</code> is "JAMIE", and this time, we're using Java 8:<br /><br /><code><pre>// We call the stored function directly inline in<br />// a SQL statement<br />dsl().select(Rentals.getCustomer(<br />          CUSTOMER.CUSTOMER_ID<br />      ))<br />     .from(CUSTOMER)<br />     .where(CUSTOMER.FIRST_NAME.eq(&quot;JAMIE&quot;))<br /><br />// This returns Result&lt;Record1&lt;CustomerTRecord&gt;&gt;<br />// We unwrap the CustomerTRecord and consume<br />// the result with a lambda expression<br />     .fetch()<br />     .map(Record1::value1)<br />     .forEach(customer -&gt; {<br />         System.out.println(&quot;Customer  : &quot;);<br />         System.out.println(&quot;- Name    : &quot; + customer.getFirstName() + &quot; &quot; + customer.getLastName());<br />         System.out.println(&quot;- E-Mail  : &quot; + customer.getEmail());<br />         System.out.println(&quot;- Address : &quot; + customer.getAddress().getAddress());<br />         System.out.println(&quot;            &quot; + customer.getAddress().getPostalCode() + &quot; &quot; + customer.getAddress().getCity().getCity());<br />         System.out.println(&quot;            &quot; + customer.getAddress().getCity().getCountry().getCountry());<br /><br />// Now, lets send the customer over the wire again to<br />// call that other stored procedure, fetching his<br />// rental history:<br />         CustomerRentalHistoryTRecord history = <br />           Rentals.getCustomerRentalHistory2(dsl().configuration(), customer);<br /><br />         System.out.println(&quot;  Customer Rental History : &quot;);<br />         System.out.println(&quot;    Films                 : &quot;);<br /><br />         history.getFilms().forEach(film -&gt; {<br />             System.out.println(&quot;      Film                : &quot; + film.getTitle());<br />             System.out.println(&quot;        Language          : &quot; + film.getLanguage().getName());<br />             System.out.println(&quot;        Description       : &quot; + film.getDescription());<br /><br />// And then, let's call again the first procedure<br />// in order to get a film's actors and categories<br />             FilmInfoTRecord info = <br />               Rentals.getFilmInfo2(dsl().configuration(), film);<br /><br />             info.getActors().forEach(actor -&gt; {<br />                 System.out.println(&quot;          Actor           : &quot; + actor.getFirstName() + &quot; &quot; + actor.getLastName());<br />             });<br /><br />             info.getCategories().forEach(category -&gt; {<br />                 System.out.println(&quot;          Category        : &quot; + category.getName());<br />             });<br />         });<br />     });<br /></pre></code><br /><br />... and a short extract of the output produced by the above:<br /><pre>Customer  : <br />- Name    : JAMIE RICE<br />- E-Mail  : JAMIE.RICE@sakilacustomer.org<br />- Address : 879 Newcastle Way<br />            90732 Sterling Heights<br />            United States<br />  Customer Rental History : <br />    Films                 : <br />      Film                : ALASKA PHANTOM<br />        Language          : English             <br />        Description       : A Fanciful Saga of a Hunter<br />                            And a Pastry Chef who must<br />                            Vanquish a Boy in Australia<br />          Actor           : VAL BOLGER<br />          Actor           : BURT POSEY<br />          Actor           : SIDNEY CROWE<br />          Actor           : SYLVESTER DERN<br />          Actor           : ALBERT JOHANSSON<br />          Actor           : GENE MCKELLEN<br />          Actor           : JEFF SILVERSTONE<br />          Category        : Music<br />      Film                : ALONE TRIP<br />        Language          : English             <br />        Description       : A Fast-Paced Character<br />                            Study of a Composer And a<br />                            Dog who must Outgun a Boat<br />                            in An Abandoned Fun House<br />          Actor           : ED CHASE<br />          Actor           : KARL BERRY<br />          Actor           : UMA WOOD<br />          Actor           : WOODY JOLIE<br />          Actor           : SPENCER DEPP<br />          Actor           : CHRIS DEPP<br />          Actor           : LAURENCE BULLOCK<br />          Actor           : RENEE BALL<br />          Category        : Music<br /></pre><br /><h2 style="font-size: 2em">If you're using Java and PL/SQL...</h2>... then you should click on the below banner and download the free trial right now to experiment with jOOQ and Oracle:<br /><br /><div style="clear: both; text-align: center;"><a href="http://www.jooq.org/"><img src="http://www.jooq.org/img/jooq-the-best-way-to-write-sql-in-java-small.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;" /></a></div><br />The Oracle port of the Sakila database is available from this URL for free, under the terms of the BSD license:<br /><br /><a href="https://github.com/jOOQ/jOOQ/tree/master/jOOQ-examples/Sakila/oracle-sakila-db">https://github.com/jOOQ/jOOQ/tree/master/jOOQ-examples/Sakila/oracle-sakila-db</a><br /><br />Finally, it is time to enjoy writing PL/SQL again!<br /><br /><h2 style="font-size: 2em">And things get even better!</h2><br />jOOQ is free and Open Source for use with Open Source databases, and it offers commercial licensing for use with commercial databases. So, if you're using Firebird, MySQL, or PostgreSQL, you can leverage all your favourite database's procedural SQL features and bind them easily to Java for free!<br /><br />For more information about jOOQ or jOOQ's DSL API, consider these resources:<br /><br /><ul><li><a href="http://www.jooq.org/learn/">The jOOQ documentation page</a></li><li><a href="http://blog.jooq.org/2012/01/05/the-java-fluent-api-designer-crash-course/">The Java Fluent API Designer Crash Course</a></li><li><a href="https://www.youtube.com/watch?v=ZQ2Y5Z0ju3c">A webinar with Arun Gupta from Red Hat about jOOQ and JavaEE</a></li><li><a href="http://vimeo.com/99526433">A presentation about jOOQ at GeeCON Krakow</a></li></ul><br /><strong>That's it with this year's mini-series on jOOQ. Have a happy Holiday season!</strong> <br /><em>This post is part of the&nbsp;<a href="http://javaadvent.com/">Java Advent Calendar</a>&nbsp;and is licensed under the&nbsp;<a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a>&nbsp;license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em>