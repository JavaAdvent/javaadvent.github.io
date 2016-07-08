---
id: 15
title: How jOOQ Allows for Fluent Functional-Relational Interactions in Java 8
date: 2014-12-18T08:30:00+00:00
author: Lukas Eder
layout: post
permalink: /2014/12/how-jooq-allows-for-fluent-functional-relational-interactions-in-java-8.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Lukas Eder
blogger_permalink:
  - /2014/12/how-jooq-allows-for-fluent-functional.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/8456709998588507860
image: /content/uploads/2014/12/jooq-the-best-way-to-write-sql-in-java-small.png
categories:
  - functional programming
  - functional relational programming
  - java
  - java 8
  - Java Advent
  - java advent 2014
  - jooq
  - lambda
  - query
  - SQL
  - streams
---
In this year's Java Advent Calendar, we're thrilled to have been asked to feature a mini-series showing you a couple of advanced and very interesting topics that we've been working on when developing <a href="http://www.jooq.org/" title="jOOQ: The best way to write SQL in Java">jOOQ</a>.<br /><br />The series consists of:<br /><br /><ul><li><a href="http://www.javaadvent.com/2014/12/how-jooq-leverages-generic-type-safety-in-its-dsl/">Dec 17: How jOOQ Leverages Generic Type Safety in its DSL</a></li><li><a href="http://www.javaadvent.com/2014/12/how-jooq-allows-for-fluent-functional-relational-interactions-in-java-8/">Dec 18: How jOOQ Allows for Fluent Functional-Relational Interactions in Java 8</a></li><li>Dec 19: How jOOQ Helps Pretend that Your Stored Procedures are a Part of Java</li></ul><br />Don't miss any of these!<br /><br /><h2 style="font-size: 2em">How jOOQ allows for fluent functional-relational interactions in Java 8</h2><br />In yesterday's article, we've seen <a href="http://www.javaadvent.com/2014/12/how-jooq-leverages-generic-type-safety-in-its-dsl/">How jOOQ Leverages Generic Type Safety in its DSL</a> when constructing SQL statements. Much more interesting than constructing SQL statements, however, is executing them.<br /><br />Yesterday, we've seen a sample PL/SQL block that reads like this:<br /><br /><code><pre>BEGIN<br />    FOR rec IN (<br />        SELECT first_name, last_name FROM customers<br />        UNION<br />        SELECT first_name, last_name FROM staff<br />    )<br />    LOOP<br />        INSERT INTO people (first_name, last_name)<br />        VALUES rec.first_name, rec.last_name;<br />    END LOOP;<br />END;<br /></pre></code><br /><br />And you won't be surprised to see that the exact same thing can be written in Java with jOOQ:<br /><br /><code><pre>for (Record2&lt;String, String> rec : <br />    dsl.select(CUSTOMERS.FIRST_NAME, CUSTOMERS.LAST_NAME).from(CUSTOMERS)<br />       .union(<br />        select(STAFF.FIRST_NAME,     STAFF.LAST_NAME    ).from(STAFF))<br />) {<br />    dsl.insertInto(PEOPLE, PEOPLE.FIRST_NAME, PEOPLE.LAST_NAME)<br />       .values(rec.getValue(CUSTOMERS.FIRST_NAME), rec.getValue(CUSTOMERS.LAST_NAME))<br />       .execute();<br />}<br /></pre></code><br /><br />This is a classic, imperative-style PL/SQL inspired approach at iterating over result sets and performing actions 1-1.<br /><br /><h2 style="font-size: 2em">Java 8 changes everything!</h2><br />With Java 8, lambdas appeared, and much more importantly, <a href="http://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html">Streams</a> did, and tons of other useful features. The simplest way to migrate the above foreach loop to Java 8's <a href="http://callbackhell.com/">"callback hell"</a> would be the following<br /><br /><code><pre>dsl.select(CUSTOMERS.FIRST_NAME, CUSTOMERS.LAST_NAME).from(CUSTOMERS)<br />   .union(<br />    select(STAFF.FIRST_NAME,     STAFF.LAST_NAME    ).from(STAFF))<br />   .forEach(rec -> {<br />        dsl.insertInto(PEOPLE, PEOPLE.FIRST_NAME, PEOPLE.LAST_NAME)<br />           .values(rec.getValue(CUSTOMERS.FIRST_NAME), rec.getValue(CUSTOMERS.LAST_NAME))<br />           .execute();<br />    }<br /></pre></code><br /><br />This is still very simple. How about this. Let's fetch a couple of records from the database, stream them, map them using some sophisticated Java function, reduce them into a batch update statement! Whew... here's the code:<br /><br /><code><pre>dsl.selectFrom(BOOK)<br />   .where(BOOK.ID.in(2, 3))<br />   .orderBy(BOOK.ID)<br />   .fetch()<br />   .stream()<br />   .map(book -> book.setTitle(book.getTitle().toUpperCase()))<br />   .reduce(<br />       dsl.batch(update(BOOK).set(BOOK.TITLE, (String) null).where(BOOK.ID.eq((Integer) null))),<br />       (batch, book) -> batch.bind(book.getTitle(), book.getId()),<br />       (b1, b2) -> b1<br />   )<br />   .execute();<br /></pre></code><br /><br />Awesome, right? Again, with comments<br /><br /><code><pre>// Here, we simply select a couple of books from the database<br />dsl.selectFrom(BOOK)<br />   .where(BOOK.ID.in(2, 3))<br />   .orderBy(BOOK.ID)<br />   .fetch()<br /><br />// Now, we stream the result as a Java 8 Stream<br />   .stream()<br /><br />// Now we map all book titles using the "sophisticated" Java function<br />   .map(book -> book.setTitle(book.getTitle().toUpperCase()))<br /><br />// Now, we reduce the books into a batch update statement...<br />   .reduce(<br /><br />//     ... which is initialised with empty bind variables<br />       dsl.batch(update(BOOK).set(BOOK.TITLE, (String) null).where(BOOK.ID.eq((Integer) null))),<br /><br />//     ... and then we bind each book's values to the batch statement<br />       (batch, book) -> batch.bind(book.getTitle(), book.getId()),<br /><br />//     ... this is just a dummy combiner function, because we only operate on one batch instance<br />       (b1, b2) -> b1<br />   )<br /><br />// Finally, we execute the produced batch statement<br />   .execute();<br /></pre></code><br /><br />Awesome, right? Well, if you're not too functional-ish, you can still resort to the "old ways" using imperative-style loops. Perhaps, your coworkers might prefer that:<br /><br /><code><pre>BatchBindStep batch = dsl.batch(update(BOOK).set(BOOK.TITLE, (String) null).where(BOOK.ID.eq((Integer) null))),<br /><br />for (BookRecord book : <br />    dsl.selectFrom(BOOK)<br />       .where(BOOK.ID.in(2, 3))<br />       .orderBy(BOOK.ID)<br />) {<br />    batch.bind(book.getTitle(), book.getId());<br />}<br /><br />batch.execute();<br /></pre></code><br /><br /><h2 style="font-size: 2em">So, what's the point of using Java 8 with jOOQ?</h2><br />Java 8 might change a lot of things. Mainly, it changes the way we reason about functional data transformation algorithms. Some of the above ideas might've been a bit over the top. But the principal idea is that whatever is your source of data, if you think about that data in terms of Java 8 Streams, you can very easily transform (map) those streams into other types of streams as we did with the books. And nothing keeps you from collecting books that contain changes into batch update statements for batch execution.<br /><br />Another example is one where we claimed that <a href="http://blog.jooq.org/2014/04/11/java-8-friday-no-more-need-for-orms/">Java 8 also changes the way we perceive ORMs</a>. ORMs are very stateful, object-oriented things that help manage database state in an object-graph representation with lots of nice features like optimistic locking, dirty checking, and implementations that support <a href="http://vladmihalcea.com/2014/09/22/preventing-lost-updates-in-long-conversations/">long conversations</a>. But they're quite terrible at data transformation. First off, they're <a href="http://blog.jooq.org/2013/11/13/popular-orms-dont-do-sql/">much much inferior to SQL in terms of data transformation capabilities</a>. This is topped by the fact that object graphs and functional programming don't really work well either.<br /><br />With SQL (and thus with jOOQ), you'll often stay on a flat tuple level. Tuples are extremely easy to transform. The following example shows how you can use an <a href="http://h2database.com/">H2 database</a> to query for <code>INFORMATION_SCHEMA</code> meta information such as table names, column names, and data types, collect those information into a data structure, before mapping that data structure into new <code>CREATE TABLE</code> statements:<br /><br /><code><pre>DSL.using(c)<br />   .select(<br />       COLUMNS.TABLE_NAME,<br />       COLUMNS.COLUMN_NAME,<br />       COLUMNS.TYPE_NAME<br />   )<br />   .from(COLUMNS)<br />   .orderBy(<br />       COLUMNS.TABLE_CATALOG,<br />       COLUMNS.TABLE_SCHEMA,<br />       COLUMNS.TABLE_NAME,<br />       COLUMNS.ORDINAL_POSITION<br />   )<br />   .fetch()  // jOOQ ends here<br />   .stream() // Streams start here<br />   .collect(groupingBy(<br />       r -> r.getTableName(),<br />       LinkedHashMap::new,<br />       mapping(<br />           r -> r,<br />           toList()<br />       )<br />   ))<br />   .forEach(<br />       (table, columns) -> {<br />            // Just emit a CREATE TABLE statement<br />            System.out.println(<br />                "CREATE TABLE " + table + " (");<br /> <br />            // Map each "Column" type into a String<br />            // containing the column specification,<br />            // and join them using comma and<br />            // newline. Done!<br />            System.out.println(<br />                columns.stream()<br />                       .map(col -> "  " + col.getName() +<br />                                    " " + col.getType())<br />                       .collect(Collectors.joining(",n"))<br />            );<br /> <br />            System.out.println(");");<br />       }<br />   );<br /></pre></code><br /><br />The above statement will produce something like the following SQL script:<br /><br /><code><pre>CREATE TABLE CATALOGS(<br />  CATALOG_NAME VARCHAR<br />);<br />CREATE TABLE COLLATIONS(<br />  NAME VARCHAR,<br />  KEY VARCHAR<br />);<br />CREATE TABLE COLUMNS(<br />  TABLE_CATALOG VARCHAR,<br />  TABLE_SCHEMA VARCHAR,<br />  TABLE_NAME VARCHAR,<br />  COLUMN_NAME VARCHAR,<br />  ORDINAL_POSITION INTEGER,<br />  COLUMN_DEFAULT VARCHAR,<br />  IS_NULLABLE VARCHAR,<br />  DATA_TYPE INTEGER,<br />  CHARACTER_MAXIMUM_LENGTH INTEGER,<br />  CHARACTER_OCTET_LENGTH INTEGER,<br />  NUMERIC_PRECISION INTEGER,<br />  NUMERIC_PRECISION_RADIX INTEGER,<br />  NUMERIC_SCALE INTEGER,<br />  CHARACTER_SET_NAME VARCHAR,<br />  COLLATION_NAME VARCHAR,<br />  TYPE_NAME VARCHAR,<br />  NULLABLE INTEGER,<br />  IS_COMPUTED BOOLEAN,<br />  SELECTIVITY INTEGER,<br />  CHECK_CONSTRAINT VARCHAR,<br />  SEQUENCE_NAME VARCHAR,<br />  REMARKS VARCHAR,<br />  SOURCE_DATA_TYPE SMALLINT<br />);<br /></pre></code><br /><br />That's data transformation! <a href="http://blog.jooq.org/2014/04/11/java-8-friday-no-more-need-for-orms/">If you're as excited as we are, read on in this article how this example works exactly</a>.<br /><br /><h2 style="font-size: 2em">Conclusion</h2><br />Java 8 has changed everything in the Java ecosystem. Finally, we can implement functional, transformative algorithms easily using Streams and lambda expressions. SQL is also a very functional and transformative language. With jOOQ and Java 8, you can extend data transformation directly from your type safe SQL result into Java data structures, back into SQL. These things aren't possible with JDBC. These things weren't possible prior to Java 8.<br /><br />jOOQ is free and Open Source for use with Open Source databases, and it offers commercial licensing for use with commercial databases.<br /><br /><div style="clear: both; text-align: center;"><a href="http://www.jooq.org/"><img src="http://www.jooq.org/img/jooq-the-best-way-to-write-sql-in-java-small.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;" /></a></div><br />For more information about jOOQ or jOOQ's DSL API, consider these resources:<br /><br /><ul><li><a href="http://www.jooq.org/learn/">The jOOQ documentation page</a></li><li><a href="http://blog.jooq.org/2012/01/05/the-java-fluent-api-designer-crash-course/">The Java Fluent API Designer Crash Course</a></li><li><a href="https://www.youtube.com/watch?v=ZQ2Y5Z0ju3c">A webinar with Arun Gupta from Red Hat about jOOQ and JavaEE</a></li><li><a href="http://vimeo.com/99526433">A presentation about jOOQ at GeeCON Krakow</a></li></ul><br /><strong>Stay tuned for tomorrow's article "How jOOQ helps pretend that your stored procedures are a part of Java"</strong> <br /><em>This post is part of the&nbsp;<a href="http://javaadvent.com/">Java Advent Calendar</a>&nbsp;and is licensed under the&nbsp;<a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a>&nbsp;license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em>