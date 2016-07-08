---
id: 68
title: Java Runtime options
date: 2012-12-20T14:14:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2012/12/java-runtime-options/
permalink: /2012/12/java-runtime-options.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Attila-Mihaly Balazs
blogger_permalink:
  - /2012/12/java-runtime-options.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/8214535049932163331
categories:
  - 2012
  - command line
  - java
---
<p>The Java runtime is a complex beast - and it has to be since it runs officially on seven platforms and unofficially on many more. Give this, it is normal that there are many knobs and dials to control how things function. The more well known ones are:</p> <ul><li>-Xmx for the maximum heap size</li><li>-client and -server for selecting the default set of parameters from classes of defaults</li><li>-XX:MaxPermGen for controlling the permanent generation size</li></ul> <p>Other than these, it is (very) rarely the case that you need to change the defaults. However, thanks to Java being open source you can see the list of options, their default values and a short explanation directly <a href="http://hg.openjdk.java.net/jdk7/jdk7/hotspot/file/b92c45f2bc75/src/share/vm/runtime/globals.hpp">from the source code</a>. Currently there are almost 800 options in there!</p> <p>An other way to see the options (but one which doesn't display the explanations unfortunately) is the following command:</p> <pre><code style="prettyprint">java -XX:+UnlockDiagnosticVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal -version</code></pre> <p>These options are well worth studying. Not for tweaking them (since there is a wealth of testing behind the defaults the extent of which would be very hard to replicate), but rather to understand the different functionalities offered by the JVM (for example <a href="http://hype-free.blogspot.com/2009/07/why-cant-i-see-stacktrace-under-java.html">why you might not see stacktraces in exceptions</a>).</p> <p><em>Meta: this post is part of the <a href="http://javaadvent.com/">Java Advent Calendar</a> and is licensed under the <a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a> license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on! Want to write for the blog? We are looking for contributors to fill all 24 slot and would love to have your contribution! <a href="mailto:dify.ltd@gmail.com">Contact Attila Balazs</a> to contribute!</em></p>