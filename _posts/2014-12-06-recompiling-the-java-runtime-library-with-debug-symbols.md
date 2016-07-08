---
id: 28
title: Recompiling the Java runtime library with debug symbols
date: 2014-12-06T08:00:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2014/12/recompiling-the-java-runtime-library-with-debug-symbols/
permalink: /2014/12/recompiling-the-java-runtime-library-with-debug-symbols.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Attila-Mihaly Balazs
blogger_permalink:
  - /2014/12/recompiling-java-runtime-library-with.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/4275031444812885917
dsq_thread_id:
  - 4962579265
categories:
  - 2014
  - debugging
  - java
  - Java Advent
  - jdk
---
<p>The JDK already comes with the sources for the Java runtime library (also known as "rt.jar") and also the class files are compiled with "LineNumberTable" which makes it possible to set breakpoints on a specified line. However the class files are not compiled with "LocalVariableTable" which means that even when you're stopped at a breakpoint, you can only look at the method parameters, not at the local variables (hint you can <a href="http://stackoverflow.com/a/3145870">use <code>javap</code></a> to check if a certain class file contains LineNumberTable, LocalVariableTable, both or neither). Probably this was done to save space, which is understandable in the case of the JRE, but for the JDK it's annoying.</p> <p>Fortunately you can fix this! Just use the ant script at the end of this post (or <a href="https://gist.github.com/cdman/84e195d9f5d36e401c8b">get it here</a>) to build rt_debug.jar. After having built it, you have multiple options to make use of it:</p> <ul><li>Put it in your <code>jre/lib/endorsed</code> directory (create it if doesn't exists)</li><li>Specify it using <code>-Xbootclasspath</code></li><li>Override <code>jre/lib/rt.jar</code> with it (after making a backup of course!)</li></ul> <p>I personally used the first method. The only downside is that classes appear twice in the Eclipse type explorer (once for rt.jar and once for rt_debug.jar. Perhaps in the future we will get the rt.jar for JDK compiled with full debug symbols by default.</p> <p>Final note: the ant script relies on having the JAVA_HOME variable available, so you might need to specify it by hand in case it isn't set. For example on Ubuntu 14.10 the full command to be run is:</p> <code><pre>JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 ant -f debug_jdk.xml</pre></code> <p>Ant script:</p>  <p>Credits for inspiration:</p> <ul><li><a href="http://www.javalobby.org/java/forums/t19866.html">General: Recompile (Some of) The JDK to Ease Debugging</a></li><li><a href="http://www.javalobby.org/java/forums/t103334.html">Enabling debugging inside JRE classes</a></li><li><a href="http://stackoverflow.com/a/18255852">A great SO answer</a></li><li><a href="http://www.javalobby.org/java/forums/t19866.html">General: Recompile (Some of) The JDK to Ease Debugging</a></li><li><a href="http://www.javalobby.org/forums/thread.jspa?messageID=91825915#91825965">Almost perfect solution (needed to be updated, was for Java 1.4)</a></li></ul> <br/> <em>This post is part of the&nbsp;<a href="http://javaadvent.com/">Java Advent Calendar</a>&nbsp;and is licensed under the&nbsp;<a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a>&nbsp;license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em>