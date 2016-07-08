---
id: 53
title: 'A conversational guide for JDK8&#8217;s lambdas &#8211; a glossary of terms'
date: 2013-12-07T08:00:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2013/12/a-conversational-guide-for-jdk8s-lambdas-a-glossary-of-terms/
permalink: /2013/12/a-conversational-guide-for-jdk8s-lambdas-a-glossary-of-terms.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Olimpiu Pop
blogger_permalink:
  - /2013/12/a-conversational-guide-for-jdk8s.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/6712832622562573534
dsq_thread_id:
  - 4962579240
categories:
  - 2013
  - java closures
  - jdk 8
  - lambda
---
<div dir="ltr" style="text-align: left;"><div style="text-align: justify;"><div style="text-align: left;">Last advent...I wrote a post related to the new treats that JDK8 has for us. Most probably the feature that I am the most excited about are the lambdas. I have to admit that in my 1st year of being the prodigal soon (during which I have developed using C#), I loved the LINQ and the nice, elegant things you can do with it. Now, even if erasure is still in the same place where we left it the last time, now we have a better way to filter, alter, walk the collections and beside the syntactic sugar, it might also make you use properly the quad core processor that you bragged about to your friends. And talking about friends this post is a cheatsheet of terms related to lambdas and stream processing that you can throw to your friends when they ask you: "What's that &lt;place term from JDK 8 related to lambdas&gt;?". It is not my intent to have a full list or a how-to lambda guide, if I said something wrong or I missed something please let me know...</div><div style="text-align: left;"><br /></div><h4>Functional interface:</h4><div style="text-align: left;"><br /></div><div style="text-align: left;">According to [jls 8] a functional interface is an interface that has just one abstract method, and thus represents a single function contract (In some cases, this "single" method may take the form of multiple abstract methods with override-equivalent signatures ([jls7_8.4.2]) inherited from superinterfaces; in this case, the inherited methods logically represent a single method).</div><div style="text-align: left;"><br /></div><div style="text-align: left;">@FunctionalInterface - is used to indicate that an interface is meant to be a functional interface. If the annotation is placed on an interface that is actually not, a compile time error occurs.</div><div style="text-align: left;"><br /></div><div style="text-align: left;">Ex:</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><code>interface Runnable { void run(); }</code></div><div style="text-align: left;"><br /></div><div style="text-align: left;">The <span style="font-family: inherit;"><code>Runnable</code> interface </span>is a very appropriate example as the only method present is the <i>run</i> method. Another Java "classic" example of functional interface is the <i>Comparator&lt;T&gt; </i>interface, in the following example is a before mentioned interface and the equals method inherited from <i>Object</i>, the interface is still functional as the <i>compare</i>&nbsp;method is the only abstract method, while the equals is inherited from the superclass.</div><div style="text-align: left;"><br /></div><br /><div style="text-align: left;"><code>interface Comparator&lt;T&gt; {</code></div><div style="text-align: left;"><code>&nbsp; boolean equals(Object obj);</code></div><code></code><br /><div style="text-align: left;"><code>&nbsp; int compare(T o1, T o2);</code></div><code></code><br /><div style="text-align: left;"><code>}</code></div><code><code></code></code><br /><div style="text-align: left;"><code><code><br /></code></code></div><code><code></code></code><br /><h4><code><code>Stream</code></code></h4><code><code></code></code><br /><div style="text-align: left;"><code><code><b>stream </b>- <span style="font-family: inherit;">according </span>to [oxford <span style="font-family: inherit;">dictionary</span>], in computing it is a continuous flow of data or instructions, typically one having a constant or predictable rate.</code></code></div><code><code></code></code><br /><div style="text-align: left;"><code><code><br /></code></code></div><code><code></code></code><br /><div style="text-align: left;"><code><code>Starting with JDK 8 <i>stream</i> represents a mechanism used for conveying elements from a data source, through a computational pipeline. A stream can use as data sources arrays, collections,&nbsp;generator functions, I/O channels.</code></code></div><code><code></code></code><br /><div style="text-align: left;"><code><code><br /></code></code></div><code><code></code></code><br /><div style="text-align: left;"><code><code>Obtaining streams:</code></code><br /><code><code><br /></code></code></div><code><code></code></code><br /><ul><code><code><li>From a <code>Collection</code> via the <code>stream()</code> and/or <code>parallelStream()</code> methods;</li><li><span style="text-align: left;">From an array via </span><code style="text-align: left;">Arrays.stream(Object[])</code></li><li><span style="text-align: left; white-space: pre;">From static factory methods on the stream classes, such as </span><code style="text-align: left; white-space: pre;">Stream.of(Object[])</code><span style="text-align: left; white-space: pre;">, </span><code style="text-align: left; white-space: pre;">IntStream.range(int, int)</code><span style="text-align: left; white-space: pre;"> or </span><code style="text-align: left; white-space: pre;">Stream.iterate(Object, UnaryOperator)</code><span style="text-align: left; white-space: pre;">;</span></li><li><span style="text-align: left; white-space: pre;">The lines of a file can be obtained from </span><code style="text-align: left; white-space: pre;">BufferedReader.lines();</code></li><li><span style="text-align: left; white-space: pre;">Streams of file paths can be obtained from methods in Files;</span></li><li><span style="text-align: left; white-space: pre;">Streams of random numbers can be obtained from </span><code style="text-align: left; white-space: pre;">Random.ints();</code></li><li><span style="text-align: left; white-space: pre;">Numerous other stream-bearing methods in the JDK, </span><code style="text-align: left; white-space: pre;">including BitSet.stream()</code><span style="text-align: left; white-space: pre;">, </span><code style="text-align: left; white-space: pre;">Pattern.splitAsStream(java.lang.CharSequence)</code><span style="text-align: left; white-space: pre;">, and </span><code style="text-align: left; white-space: pre;">JarFile.stream()</code><span style="text-align: left; white-space: pre;">.</span></li></code></code></ul><code><code></code></code></div><div style="text-align: left;">stream operations - actions taken on the stream. From the point of view of stream manipulation there are two types of actions: intermediate and terminal&nbsp;&nbsp;operations</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><b>stream intermediate operation</b> - operations that are narrowing the content of the stream. Intermediate operations are lazy by nature - do not actually&nbsp;alter the content of the stream, but create another narrower stream. The traversal of the stream begins only when the terminal operation is called.</div><div style="text-align: left;"><ul style="text-align: left;"><li>&nbsp;filter - filters the stream based on the provided predicate</li><li>&nbsp;map - creates a new stream by applying the mapping function to each element from the initial stream (corresponding methods for each numeric type: int, long, double)</li><li>&nbsp;flatMap - operation that has the effect of applying a one-to-many transformation to the elements of the stream, and then flattening the results elements into a new stream. For example, if orders is a stream of purchase orders, and each purchase order contains a collection of line items, then the following produces a stream of line items:</li></ul></div><div style="text-align: left;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<code>orderStream.flatMap(order -&gt; order.getLineItems().stream())</code></div><div style="text-align: left;"><ul style="text-align: left;"><li>distinct - returns a stream of distinct operations</li><li>sorted - returns a stream of sorted operations</li><li>peek - debug focused method that returns a stream consisting of elements of this stream, the provided action is performed on each element</li></ul></div><div style="text-align: left;">&nbsp;Ex:</div><span style="white-space: pre;"></span><br /><code></code><br /><div style="text-align: left;"><code><span style="white-space: pre;"><span> </span>list.stream()</span></code></div><code>&nbsp; &nbsp; &nbsp;.filter(filteringFunction)<br /></code><br /><div style="text-align: left;"><code>&nbsp; &nbsp; &nbsp;.peek(e -&gt; {System.out.println("Filtered value: " + e); });</code></div><code></code><br /><div style="text-align: left;"><code>&nbsp; &nbsp; &nbsp;.map(mappingFunction)</code></div><code></code><br /><div style="text-align: left;"><code>&nbsp; &nbsp; &nbsp;.peek(e -&gt; {System.out.println("Mapped value: " + e); });</code></div><code></code><br /><div style="text-align: left;"><code>&nbsp; &nbsp; &nbsp;.collect(Collectors.intoList());</code></div><code></code><span style="white-space: pre;"></span><br /><div style="text-align: left;"></div><div style="text-align: left;"><ul style="text-align: left;"><li>limit - returns a truncated version of the current stream (no more than the limit number of elements)</li><li>substream - returns a stream consisting of the remaining element starting from startposition, or between startPosition and endPosition</li></ul></div><div style="text-align: left;"><br /></div><div style="text-align: left;"><b>stream terminal operation</b> - traverse the stream to produce a result or a side-effect. After the execution of the terminal operation, the stream is considered</div><div style="text-align: left;">consumed( calling another operation on a consumed stream will throw an IllegalStateException). Terminal operations are eager in nature, with the exception of iterator() and splititerator() that provide an extension mechanism for those that don't find the needed function in the API.</div><div style="text-align: left;"><ul style="text-align: left;"><li>forEach - applies the provided operation to every element of the stream. Also the forEachOrdered version exists</li><li>toArray - extracts the elements of the stream into an array</li><li>reduce &nbsp;- reduction method</li><li>collect - mutable reduction method</li><li>min &nbsp; &nbsp; - computes the minimum of the stream</li><li>max &nbsp; &nbsp; - computes the maximum of the stream</li><li>count &nbsp; - counts the elements of the stream</li><li>anyMatch - returns true if there is an element matching the provided criteria</li><li>allMatch - returns true if all the elements match</li><li>noneMatch - returns true if none of the elements match&nbsp;</li><li>findFirst - finds the first element that matches the provided condition</li><li>findAny - returns an element from the stream</li></ul></div><div style="text-align: left;"><br /></div><div style="text-align: left;"><br /></div><div style="text-align: left;"><b>stream pipeline</b>: consists of a source, followed by zero or more intermediate operations and a terminal operation.</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><b>spliterator </b>- spliterator for traversing and partinioning elements of a source. One can use it for traverse, estimate element count or split it in multiple spliterators</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><b>Reduction </b>- a reduction operation (or fold) takes a sequence of input elements and combines them into a single summary result by repeated application</div><div style="text-align: left;">of a combining operation. A reduction operation can be computing the sum, max, min, count or collecting the elements in a list. The reduction operation</div><div style="text-align: left;">is also parallelizable as long as the function(s) used are associative and stateless. The method used for reduction is reduce()</div><div style="text-align: left;"><br /></div><div style="text-align: left;">Ex: reduction using sum:</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><code>int sum = numbers.stream().reduce(0, (x,y) -&gt; x + y);</code></div><div style="text-align: left;"><br /></div><div style="text-align: left;">or</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><code>int sum = numbers.stream().reduce(0, Integer::sum);</code></div><div style="text-align: left;"><br /></div><div style="text-align: left;">Mutable reduction - is a operation that accumulates input elements into a mutable result container (StringBuilder or Collection) as it processes the elements in</div><div style="text-align: left;">the stream.</div><div style="text-align: left;"><br /></div><div style="text-align: left;">Ex: <code>String concatenated = strings.reduce("", String::concat)</code></div><div style="text-align: left;"><br /></div><div style="text-align: left;">Predicate - functional interface that determines whether the input object matches some criteria</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><br /></div><div style="text-align: left;">I hope you find this condensed list beneficial and you keep it in your bookmarks for those moments when you need all these terms on one page.</div><div style="text-align: left;">If you find something that is missing, please let me know so I can correct it.</div><div style="text-align: left;"><br /></div><div style="text-align: left;">So...I wish you a nice Advent Time and a Happy/Jolly/Merry but most important of all I wish you a peaceful Christmas!</div><div style="text-align: left;"><br /></div><div style="text-align: left;"><br /></div><div style="text-align: left;"><br /></div><div style="text-align: left;"><br /></div><div style="text-align: left;">&nbsp;Useful resources:</div><div style="text-align: left;"><br /></div><div style="text-align: left;">Most of the information was extracted from [stream]. Particularly useful I found the very comprehensive javadoc of the given classes.&nbsp;</div><div style="text-align: left;"><br /></div><div style="text-align: left;">&nbsp;[stream] -&nbsp;<a href="http://download.java.net/jdk8/docs/api/java/util/stream/package-summary.html#package.description">http://download.java.net/jdk8/docs/api/java/util/stream/package-summary.html#package.description</a></div><div style="text-align: left;">&nbsp;[oxford dictionary] - <a href="http://www.oxforddictionaries.com/definition/english/stream?q=stream">http://www.oxforddictionaries.com/definition/english/stream?q=stream</a></div><div style="text-align: left;">&nbsp;[jls 8] - <a href="http://cr.openjdk.java.net/~dlsmith/jsr335-0.6.1/A.html">http://cr.openjdk.java.net/~dlsmith/jsr335-0.6.1/A.html</a></div><div style="text-align: left;">&nbsp;[jls7_8.4.2] -&nbsp;<a href="http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.2">http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.2</a><br /><br />Lambda Related resources on <a href="https://plus.google.com/110574453398024123194">+ManiSarkar</a>'s recommendation:<br /><br /><span style="background-color: #fcffee; color: #222222; font-family: Georgia, Utopia, 'Palatino Linotype', Palatino, serif; font-size: 14px; line-height: 19px; text-align: justify;"><a href="https://github.com/AdoptOpenJDK/lambda-tutorial">https://github.com/AdoptOpenJDK/lambda-tutorial</a></span><br /><span style="background-color: #fcffee; color: #222222; font-family: Georgia, Utopia, 'Palatino Linotype', Palatino, serif; font-size: 14px; line-height: 19px; text-align: justify;"><a href="http://lambdafaq.com/">http://lambdafaq.com</a></span><br /><br /><em>Meta: this post is part of the&nbsp;<a href="http://javaadvent.com/">Java Advent Calendar</a>&nbsp;and is licensed under the&nbsp;<a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a>&nbsp;license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em></div></div>