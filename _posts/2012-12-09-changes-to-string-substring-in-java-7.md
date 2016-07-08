---
id: 80
title: Changes to String.substring in Java 7
date: 2012-12-09T09:00:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2012/12/changes-to-string-substring-in-java-7/
permalink: /2012/12/changes-to-string-substring-in-java-7.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Attila-Mihaly Balazs
blogger_permalink:
  - /2012/12/changes-to-stringsubstring-in-java-7.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/6433931092439400240
dsq_thread_id:
  - 4962579204
categories:
  - 2012
  - String
  - substring
---
<p>It is common knowledge that Java optimizes the substring operation for the case where you generate a lot of substrings of the same source string. It does this by using the <code>(value, offset, count)</code> way of storing the information. See an example below:</p> <div style="clear: both; text-align: center;"><img border="0" height="124" width="320" src="http://4.bp.blogspot.com/-gnaLPXGMeUQ/UMIaKhQ5wsI/AAAAAAAAFn8/wNPgGPtE2qY/s320/Untitled%2Bdrawing.png" /></div> <p>In the above diagram you see the strings "Hello" and "World!" derived from "Hello World!" and the way they are represented in the heap: there is one character array containing "Hello World!" and two references to it. This method of storage is advantageous in some cases, for example for a compiler which tokenizes source files. In other instances it may lead you to an OutOfMemorError (if you are routinely reading long strings and only keeping a small part of it - but the above mechanism prevents the GC from collecting the original String buffer). Some even <a href="http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=4513622">call it a bug</a>. I wouldn't go so far, but it's certainly a leaky abstraction because you were forced to do the following to ensure that a copy was made: <code>new String(str.substring(5, 6))</code>.</p> <div style="clear: both; text-align: center;"><img border="0" height="107" width="320" src="http://3.bp.blogspot.com/-NGSJS_psCIc/UMW40g0NLXI/AAAAAAAAFoQ/7kfOVA8JdC0/s320/Untitled%2Bdrawing.png" /></div> <p>This all changed in <a href="http://mail.openjdk.java.net/pipermail/core-libs-dev/2012-May/010257.html">May of 2012</a> or Java 7u6. The pendulum is swung back and now full copies are made by default. What does this mean for you?</p> <ul><li>For most probably it is just a nice piece of Java trivia</li><li>If you are writing parsers and such, you can not rely any more on the implicit caching provided by String. You will need to implement a similar mechanism based on buffering and a custom implementation of CharSequence</li><li>If you were doing <code>new String(str.substring)</code> to force a copy of the character buffer, you can stop as soon as you update to the latest Java 7 (and you need to do that quite soon since <a href="https://blogs.oracle.com/java/entry/end_of_public_updates_for">Java 6 is being EOLd as we speak</a>).</li></ul> <p>Thankfully the development of Java is an open process and such information is at the fingertips of everyone!</p> <p>A couple of more references (since we don't say pointers in Java :-)) related to Strings:</p> <ul><li>If you are storing the same string over and over again (maybe you're parsing messages from a socket for example), you should <a href="http://hype-free.blogspot.ro/2010/03/stringintern-there-are-better-ways.html">read up on alternatives to String.intern()</a> (and also consider reading chapter 50 from the second edition of Effective Java: Avoid strings where other types are more appropriate)</li><li>Look into (and do benchmarks before using them!) options like UseCompressedStrings (which <a href="http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=7129417">seems to have been removed</a>), UseStringCache and StringCache</li></ul> <p>Hope I didn't strung you along too much and you found this useful! Until next time<br />- Attila Balazs</p> <p><em>Meta: this post is part of the <a href="http://javaadvent.com/">Java Advent Calendar</a> and is licensed under the <a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a> license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on! Want to write for the blog? We are looking for contributors to fill all 24 slot and would love to have your contribution! <a href="mailto:dify.ltd@gmail.com">Contact Attila Balazs</a> to contribute!</em></p>