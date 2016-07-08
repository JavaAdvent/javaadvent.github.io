---
id: 758
title: 'Java in 2015 &#8211; Major happenings'
date: 2015-12-24T00:01:21+00:00
author: Martijn Verburg
layout: post
guid: http://www.javaadvent.com/?p=758
permalink: /2015/12/java-in-2015-the-language-platform-ecosystem-and-community-continue-to-dominate.html
categories:
  - 2015
  - collections
  - functional programming
  - java
  - java 8
  - Java Advent
  - jdk 8
---
2015 was the year where Java the language, platform, ecosystem and community continue to dominate the software landscape, with only Javascript having a similar sized impact on the industry.  In case you missed the highlights of 2015, here's some of the major happenings that occurred.

<h2>Java 20 years old and still not dead yet!</h2>

<a href="https://community.oracle.com/community/java/javas-20th-anniversary" target="_blank">Java turned 20 this year</a> and swept back to the top of the Tiobe index in <a href="http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html" target="_blank">December 2015</a>. Although the Tiobe index is hardly a 100% peer reviewed scientific methodology, it is seen as a pretty strong barometer for the health of a language/platform.  So what the heck happened to boost Java so dramatically again?

Firstly, the release of Java 8 the previous year was adopted by mainstream Java enterprise shops.  The additional functional capabilities of Lambdas combined with the new Streams and Collections framework breathed a new lease of life into the language. Although Java 8 is not as rich in its feature set as say Scala or Python it is seen as the steady workhorse that now has at least some feature parity with more aggressive languages.  Enterprises love a stable platform and it's unlikely that Java will be disappearing any time soon.

Secondly, Java has become a strong platform to use for infrastructure platforms/frameworks.  Many popular NoSQL, datagrid solutions such as <a href="http://cassandra.apache.org/" target="_blank">Apache Cassandra</a>, <a href="http://hazelcast.org/" target="_blank">Hazelcast</a> are written in Java, again due to its stability and strong threading and networking support. CI tools such as Jenkins are widely adopted and of course business productivity tools such as Atlassian's JIRA are again Java based.

<h2>Oracle guts its Java evangelism team</h2>

Oracle <a href="https://jaxenter.com/oracle-sacks-java-evangelists-120362.html" target="_blank">fired much of its Java evangelism team</a> just before JavaOne which wasn't the greatest PR move by the stewards of Java.  Over the subsequent months it became clearer that this wasn't a step by Oracle to reduce its engineering efforts into Java but there were nervous times for much of the community as they feared the worst.  A salient reminder that big corporations don't always get their left hand talking to their right!

<h2>Java 9 delay announced</h2>

In the "We're not really surprised" bucket came the announcement the <a href="http://mail.openjdk.java.net/pipermail/jdk9-dev/2015-December/003149.html" target="_blank">Java 9 will be delayed</a> until March 2017 in order to ensure that the new modularisation system will not break the millions of Java applications running out there today.

Although the technical work of Jigsaw is progressing nicely, the entire ecosystem will need to test on the new system.  The <a href="http://openjdk.java.net/groups/quality/" target="_blank">Quality group</a> in OpenJDK is leading this effort. I highly recommend you contact them to be part of the early access and feedback loop.

<h2>OpenJDK supports further mobile platforms</h2>

The creation of the <a href="http://openjdk.java.net/projects/mobile/" target="_blank">OpenJDK mobile project</a> came as a surprise to many and although it doesn't represent a change in Oracle's business direction it was a wlecome release of code to enable Java on ARM, Android and iOS platforms.  There's much technical work to do but it will be interesting to watch if the software community at large picks up on this new support and tries Java out as a language for the iOS and Android platforms in 2016 and beyond. There is a possibility that OpenFX (JavaFX) combined with Java mobile on iOS or Android may entice a slew of developers to this 'new' platform.

<h2>Was I right about 2015?</h2>
It's always fun to <a href="http://www.javaadvent.com/2014/12/the-java-ecosystem-my-top-5-highlights-of-2014.html" target="_blank">look at past predictions</a>, let's see how I did!

<ol>
	<li>I expected 2015 to be a little bit quieter.  Well I clearly got that wrong! Despite no major releases for ME, SE or EE, the excitement of celebrating 20 years of Java and a surge of new developers using Java 8 meant 2015 was busier than ever.</li>
<li>Embracing Javascript for the front end. This trend continues and stacks such as <a href="https://jhipster.github.io/" target="_blank">JHipster</a> show the new love affair that Java developers have with Javascript.</li>
<li>Devops toolchains to the fore. Docker continues to steamroll ahead in terms of popularity and Java developers are especially starting to use Docker in test environments to avoid polluting environments with variations in Java runtimes, web servers, data stores etc.</li>
<li>IoT and Java to be a thing. Nope, not yet! Perhaps in 2016 with the new <a href="http://openjdk.java.net/projects/mobile/" target="_blank">Mobile Java project</a> in OpenJDK and further refinement of Java ME, we may start to see serious inroads.</li>
</ol>

I'm not going to make any predictions for 2016 as I clearly need to stick to my day job :-)

One final important note. Project Jigsaw is the modularisation story for Java 9 that will massively impact tool vendors and day to day developers alike. The community at large needs your help to help test out early builds of Java 9 and to help OpenJDK developers and tool vendors ensure that IDEs, build tools and applications are ready for this important change. You can join us in the <a href="http://adoptopenjdk.java.net" target="_blank">Adoption Group at OpenJDK</a>. I hope everyone has a great holiday break - I look forward to seeing the Twitter feeds and the GitHub commits flying around in 2016 :-).

Cheers,
Martijn (CEO - <a href="http://www.jclarity.com/">jClarity</a>, Java Champion &amp; Diabolical Developer)

<em style="background-color: #fcffee; color: #222222; font-family: Verdana, Geneva, sans-serif; font-size: 18px; line-height: 24.6399993896484px;">This post is part of the <a style="color: #888888; text-decoration: none;" href="http://javaadvent.com/">Java Advent Calendar</a> and is licensed under the <a style="color: #888888; text-decoration: none;" href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a> license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em>