---
id: 541
title: Using Java 8 Lambdas, Streams, and Aggregates
date: 2015-12-12T01:00:42+00:00
author: Eric Ward
layout: post
guid: http://www.javaadvent.com/?p=541
permalink: /2015/12/using-java-8-lambdas-streams-and-aggregates.html
dsq_thread_id:
  - 4962579380
categories:
  - 2015
---
<h3>Overview</h3>
In this post, we'll take a look at filtering and manipulating objects in a Collection using Java 8 lambdas, streams, and aggregates.  All code in this post is available in BitBucket <a href="https://bitbucket.org/t_eric_ward/javaadventlambdasstreamsaggregates" target="_blank">here</a>.

For this example we'll create a number of objects that represent servers in our IT infrastructure.  We'll add these objects to a List and then we'll use lambdas, streams, and aggregates to retrieve servers from the List based on certain criteria.
<h3>Objectives</h3>
<ol>
	<li>Introduce the concepts of lambdas, streams, and aggregate operations.</li>
	<li>Explain the relationship between streams and pipelines.</li>
	<li>Compare and contrast aggregate operations and iterators.</li>
	<li>Demonstrate the filter, collect, forEach, mapToLong, average, and getAsDouble aggregate operations.</li>
</ol>
<h3>Lambdas</h3>
Lambdas are a new Java language feature that allows us to pass functionality or behavior into methods as parameters.  One example that illustrates the usefulness of Lambdas comes from UI coding. When a user clicks on button on a user interface, it usually causes some action to occur in the application. In this case, we really want to pass a behavior into the onClick(...) method so that the application will execute the given behavior when the button is clicked. In previous versions of Java, we accomplished this by passing an anonymous inner class (that implemented a known interface) into the method. Interfaces used in this kind of scenario usually contain only one method which defines the behavior we wish to pass into the onClick(...) method. Although this works, the syntax is unwieldy. Anonymous inner classes still work for this purpose, but the new Lambda syntax is much cleaner.
<h3>Aggregate Operations</h3>
When we use Collections to store objects in our programs, we generally need to do more than simply put the objects in the collection — we need to store, retrieve, remove, and update these objects. Aggregate operations use lambdas to perform actions on the objects in a Collection. For example, you can use aggregate operations to:
<ul>
	<li>Print the names of all the servers in inventory from a particular manufacturer</li>
	<li>Return all of the servers in inventory older than a particular age</li>
	<li>Calculate and return the average age of Servers in your inventory (provided the Server object has a purchase date field)</li>
</ul>
All of these tasks can be accomplished by using aggregate operations along with pipelines and streams.  We will see examples of these operations below.
<h3>Pipelines and Streams</h3>
A pipeline is simply a sequence of aggregate operations. A stream is a sequence of items, not a data structure, that carries items from the source through the pipeline. Pipelines are composed of the following:
<ol>
	<li>A data source. Most commonly, this is a Collection, but it could be an array, the return from a method call, or some sort of I/O channel.</li>
	<li>Zero or more intermediate operations. For example, a Filter operation. Intermediate operations produce a new stream. A filter operation takes in a stream and then produces another stream that contains only the items matching the criteria of the filter.</li>
	<li>A terminal operation. Terminal operations return a non-stream result. This result could be a primitive type (for example, an integer), a Collection, or no result at all (for example, the operation might just print the name of each item in the stream).</li>
</ol>
Some aggregate operations (i.e. forEach) look like iterators, but they have fundamental differences:
<ol>
	<li>Aggregate operations use internal iteration. Your application has no control over how or when the elements are processed (there is no next() method).</li>
	<li>Aggregate operations process items from a stream, not directly from a Collection.</li>
	<li>Aggregate operations support Lambda expressions as parameters.</li>
</ol>
<h3>Lambda Syntax</h3>
Now that we have discussed the concepts related to Lambda expressions, it is time to look at their syntax. You can think of Lambda expressions as anonymous methods because they have no name. Lambda syntax consists of the following:
<ul>
	<li>A comma-separated list of formal parameters enclosed in parentheses. Data types of parameters can be omitted in Lambda expressions. The parentheses can be omitted if there is only one formal parameter.</li>
	<li>The arrow token: -&gt;</li>
	<li>A body consisting of a single expression or code block.</li>
</ul>
<h3>Using Lambdas, Streams, and Aggregate Operations</h3>
As mentioned in the overview, we'll demonstrate the use of lambdas, streams, and aggregates by filtering and retrieving Server objects from a List.  We'll look at four examples:
<ol>
	<li>Finding and printing the names of all the servers from a particular manufacturer.</li>
	<li>Finding and printing the names of all of the servers older than a certain number of years.</li>
	<li>Finding and extracting into a new List all of the servers older than a certain number of years and then printing the names of the servers in the new list.</li>
	<li>Calculating and displaying the average age of the servers in the List.</li>
</ol>
Let's get started...
<h4>The Server Class</h4>
First, we'll look at our Server class.  The Server class will keep track of the following:
<ol>
	<li>Server name</li>
	<li>Server IP address</li>
	<li>Manufacturer</li>
	<li>Amount of RAM (GB)</li>
	<li>Number of processors</li>
	<li>Purchase date (LocalDate)</li>
</ol>
Notice (at line 65) that we've added the method <code>getServerAge()</code> that calculates the age of the server (in years) based on the purchase date - we'll use this method when we calculate the average age of the Servers in our inventory.

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-09-at-9.57.17-AM.png"><img class="wp-image-545 size-full alignleft" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-09-at-9.57.17-AM.png" alt="Screen Shot 2015-12-09 at 9.57.17 AM" width="475" height="918" /></a>

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;
<h4>Creating and Loading the Servers</h4>
Now that we have a Server class, we'll create a List and load several servers:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.17.47-AM.png"><img class="alignnone wp-image-567 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.17.47-AM.png" alt="Screen Shot 2015-12-10 at 10.17.47 AM" width="694" height="797" /></a>
<h4>Example 1: Print the Names of All the Dell Servers</h4>
For our first example, we'll write some code to find all of the servers made by Dell and then print the server names to the console:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.29.48-AM.png"><img class="alignnone wp-image-569 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.29.48-AM.png" alt="Screen Shot 2015-12-10 at 10.29.48 AM" width="903" height="195" /></a>

Our first step is on line 76 - we have to get the stream from our list of servers.  Once we have the stream, we add the <strong>filter</strong> intermediate operation on line 77.  The filter operation takes a stream of servers as input and then produces another stream of servers containing only the servers that match the criteria specified in the filter's lambda.  We select only the servers that are made by Dell using the following lambda:<code>
s -&gt; s.getManufacturer().equalsIgnoreCase(manufacturer)
</code>

The variable <strong>s</strong> represents each server that is processed from the stream (remember that we don't have to declare the type).  The right hand side of the arrow operator represents the statement we want to evaluate for each server processed.  In this case, we'll return true if the current server's manufacturer is Dell and false otherwise.  The resulting output stream from the filter contains only those servers made by Dell.

Finally, we add the <strong>forEach</strong> terminal operation on line 78.  The forEach operation takes a stream of servers as input and then runs the given lambda on each server in the stream.   We print the names of the Dell servers to the console using the following lambda:<code>
server -&gt; System.out.println(server.getName())
</code>

Note that we used <strong>s</strong> as the variable name for each server in the stream in the first lambda and <strong>server</strong> as the variable name in the second - they don't have to match from one lambda to the next.

The output of the above code is what we expect:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-11.08.38-AM.png"><img class="alignnone wp-image-577 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-11.08.38-AM.png" alt="Screen Shot 2015-12-10 at 11.08.38 AM" width="506" height="58" /></a>
<h4>Example 2: Print the Names of All the Servers Older Than 3 Years</h4>
Our second example is similar to the first except that we want to find the servers that are older than 3 years:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.45.33-AM.png"><img class="alignnone wp-image-570 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.45.33-AM.png" alt="Screen Shot 2015-12-10 at 10.45.33 AM" width="831" height="191" /></a>

The only difference between this example and the first is that we changed the lambda expression in our filter operation (line 89) to this:<code>
s -&gt; s.getServerAge() &gt; age
</code>

The output stream from this filter contains only servers that are older than 3 years.

The output of the above code is:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-11.25.27-AM.png"><img class="alignnone wp-image-580 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-11.25.27-AM.png" alt="Screen Shot 2015-12-10 at 11.25.27 AM" width="482" height="86" /></a>
<h4>Example 3: Extract All Servers Older Than 3 Years Into a New List</h4>
Our third example is similar to the second in that we are looking for the servers that are older than three years.  The difference in this example is that we will create a new List containing only the servers that meet our criteria:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.47.36-AM.png"><img class="alignnone wp-image-571 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.47.36-AM.png" alt="Screen Shot 2015-12-10 at 10.47.36 AM" width="911" height="244" /></a>

As in the previous example, we get the stream from the List and add the filter intermediate operation to create a stream containing only those servers older than 3 years (lines 102 and 103).  Now, on line 104, we use the <strong>collect</strong> terminal operation rather than the <strong>forEach </strong>terminal operation.  The collect terminal operation takes a stream of servers as input and then puts them in the data structure specified in the parameter.  In our case, we convert the stream into a list of servers.  The resulting list is referenced by the <strong>oldServers</strong> variable declared on Line 100.

Finally, to demonstrate that we get the same set of servers in this example as the last, we print the names of all the servers in the oldServers list.  Note that, because we want all of the servers in the list, there is no intermediate filter operation.  We simply get the stream from oldServers and feed it to the forEach terminal operation.

The output is what we expect:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-12.13.44-PM.png"><img class="alignnone wp-image-582 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-12.13.44-PM.png" alt="Screen Shot 2015-12-10 at 12.13.44 PM" width="546" height="113" /></a>
<h4>Example 4: Calculate and Print the Average Age of the Servers</h4>
In our final example, we'll calculate the average age of our servers:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.50.02-AM.png"><img class="alignnone wp-image-572 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.50.02-AM.png" alt="Screen Shot 2015-12-10 at 10.50.02 AM" width="697" height="208" /></a>

The first step is the same as our previous examples - we get the stream from our list of servers.  Next we add the <strong>mapToLong</strong> intermediate operation.  This aggregate operation takes a stream of servers as input and produces a stream of Longs as output.  The servers are mapped to Longs according to the specified lambda on Line 119 (you can also use the equivalent syntax on Line 120).  In this case, we are grabbing the age of each incoming server and putting it into the resulting stream of Longs.

Next we add the <strong>average</strong> terminal operation.  Average<strong> </strong>does exactly what you would expect - it calculates the average of all of the values in the Stream.  Terminal operations like average that return one value by combining or operating on the contents of a stream are known as <strong>reduction operations</strong>.  Other examples of reduction operations include <b>sum</b>, <strong>min</strong>, <strong>max</strong>, and <strong>count</strong>.

Finally, we add the operation <strong>getAsDouble</strong>.  This is required because average returns the type <strong>OptionalDouble</strong>.  If the incoming stream is empty, average returns an empty instance of OptionalDouble.  If this happens, calling getAsDouble will throw a<b> NoSuchElementException</b>, otherwise it just returns the Double value in the OptionalDouble instance.

The output of this example is:

<a href="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.35.24-PM.png"><img class="alignnone wp-image-593 size-full" src="http://www.javaadvent.com/content/uploads/2015/12/Screen-Shot-2015-12-10-at-10.35.24-PM.png" alt="Screen Shot 2015-12-10 at 10.35.24 PM" width="425" height="53" /></a>
<h3>Conclusion</h3>
We've only scratched the surface as to what you can do with lambdas, streams, and aggregates.  I encourage you to grab the source code, play with it, and start to explore all the possibilities of these new Java 8 features.

&nbsp;