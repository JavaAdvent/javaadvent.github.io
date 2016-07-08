---
id: 148
title: 'Java regular expression library benchmarks &#8211; 2015'
date: 2015-12-22T01:00:24+00:00
author: Attila-Mihály Balázs
layout: post
guid: http://www.javaadvent.com/?p=148
permalink: /2015/12/java-regular-expression-library-benchmarks-2015.html
dsq_thread_id:
  - 4962579383
categories:
  - 2015
  - benchmark
  - regular expression
tags:
  - 2015
  - benchmark
  - regular expression
---
While trying to get Java to #1 in the [regexdna challenge for The Computer Language Benchmarks Game](http://benchmarksgame.alioth.debian.org/u64q/performance.php?test=regexdna) I was researching the performance of regular expression libraries for Java. The most recent website I could find was [tusker.org](http://tusker.org/regex/regex_benchmark.html) from 2010. Hence I decided to redo the tests using [the Java Microbenchmarking Harness](http://openjdk.java.net/projects/code-tools/jmh/) and publish the results.
**TL;DR**: regular expressions are good for ad-hoc querying but if you have something performance sensitive, you should hand-code your solution (this doesn't mean that you have to start from absolute zero - the Google Guava library has for example [some nice utilities](https://github.com/google/guava/wiki/StringsExplained) which can help in writing readable but also performant code).

And now, for some charts summarizing the performance - the test was run on an 64bit Ubuntu 15.10 machine with OpenJDK 1.8.0_66:

| | Small texts | Large texts |
|--|-------------|-------------|
|Linear scale| ![Java regular expression libraries results on small texts](http://www.javaadvent.com/content/uploads/2015/10/java-regex-libraries-benchmarks-small-texts.png) | ![Java regular expression libraries results on a large text](http://www.javaadvent.com/content/uploads/2015/10/java-regex-libraries-benchmarks-large-text.png) |
|Logarithmic scale| ![Java regular expression libraries results on small texts - logarithmic scale](http://www.javaadvent.com/content/uploads/2015/10/java-regex-libraries-benchmarks-small-texts-log-scale.png) | ![Java regular expression libraries results on a large text - logarithmic scale](http://www.javaadvent.com/content/uploads/2015/10/java-regex-libraries-benchmarks-large-text-log-scale.png) |

Observations:

* there is no "standard" for regular expressions, so different libraries can behave differently when given a particular regex and a particular string to match against - ie. one might say that it matches but the other might say that it doesn't. For example, even though I used a very reduced set of testcases (5 regexes checked against 6 strings), only two of the libraries managed to match / not match them all correctly (one of them being java.util.Pattern).

* it probably takes more than one try to get your regex right (tools like [regexpal](http://regexpal.com/) or [The Regex Coach](http://www.weitz.de/regex-coach/) are very useful for experimenting)

* the performance of a regex is hard to predict (and sometimes it can have [exponential complexity based on the input length](http://stackoverflow.com/a/8887822)) - because of this you need to think twice if you accept a regular expression from arbitrary users on the Internet (like a search engine which would allow search by regular expressions for example)

* none of the libraries seems to be in active development any more (in fact quite a few from the original list on [tusker.org](http://tusker.org/regex/regex_benchmark.html) are now unavailable) and many of them are slower than the built-in [j.u.Pattern](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html), so *if* you use regexes that should probably be the first choice.

* that said, the performance of both the hardware and JVM has been considerable, so if you *are* using one of these libraries, it is running generally an order of magnitude faster than it was five years ago. So there is no need to quickly replace working code (unless your profiler says that it is a problem :-))

* watch out for calls to [String.split](http://docs.oracle.com/javase/8/docs/api/java/lang/String.html#split-java.lang.String-) in loops. While it has some optimization for particular cases (such as one-char regexes), you should almost always:
  * see if you can use something like [Splitter](https://github.com/google/guava/wiki/StringsExplained#splitter) from Google Guava
  * if you need a regular expression, at least pre-compile it outside of the loop

* the two surprises were [dk.brics.automaton](http://www.brics.dk/automaton/) which outperformed everything else by several orders of magnitude, however:
  * the last release was in 2011 and seems to be more an academic project
  * it doesn't support the same syntax as java.util.Pattern (but doesn't give you a warning if you try to use a j.u.Pattern - it just won't match the strings you think it should)
  * doesn't have an API as comfortable as j.u.Pattern (for example it's missing replacements)

* the other surprise was [kmy.regex.util.Regex](http://jint.sourceforge.net/), which - although not updated since 2000 - outperformed java.util.Pattern and passed all the tests (of which there weren't admittedly many).

The complete list of libraries used:

| Library name and version (release year) | Available in Maven Central | License | Average ops/second | Average ops/second (large text) | Passing tests |
|--------------------------|---------| ----|-------|---------------------------------|---------------|
| [j.util.Pattern](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html) 1.8 (2015) | no (comes with JRE) | JRE license | 19 689 | 22 144 | **5 out of 5** |
| [dk.brics.automaton.Automaton](http://www.brics.dk/automaton/) 1.11-8 (2011) | yes | BSD | **2 600 225** | **115 374 276** | 2 out of 5 |
| [org.apache.regexp](http://jakarta.apache.org/regexp/) 1.4 (2005) | yes | Apache (?) | 6 738 | 16 895 | 4 out of 5 |
| [com.stevesoft.pat.Regex](http://www.javaregex.com/) 1.5.3 (2009) | yes | LGPL v3 | 4 191 | 859 | 4 out of 5 |
| [net.sourceforge.jregex](http://jregex.sourceforge.net/) 1.2_01 (2002) | yes | BSD | 57 811 | 3 573 | 4 out of 5 |
| [kmy.regex.util.Regex](http://jint.sourceforge.net/) 0.1.2 (2000) | no | Artistic License | 217 803 | 38 184 | **5 out of 5** |
| [org.apache.oro.text.regex.Perl5Matcher](http://jakarta.apache.org/oro/) 2.0.8 (2003) | yes | Apache 2.0 | 31 906 | 2383 | 4 out of 5 |
| [gnu.regexp.RE](http://www.cacas.org/java/gnu/regexp/) 1.1.4 (2005?) | yes | GPL (?) | 11 848 | 1 509 | 4 out of 5 |
| [com.basistech.tclre.RePattern](http://basis-technology-corp.github.io/tcl-regex-java/) 0.13.6 (2015) | yes | Apache 2.0 | 11 598 | 43 | 3 out of 5 |
| [com.karneim.util.collection.regex.Pattern](http://www.karneim.com/jrexx/) 1.1.1 (2005?) | yes | ? | - | - | 2 out of 5 |
| [org.apache.xerces.impl.xpath.regex.RegularExpression](http://xerces.apache.org/) 2.11.0 (2014) | yes | Apache 2.0 | - | - | 4 out of 5 |
| com.ibm.regex.RegularExpression 1.0.2 (no longer available) | no | ? | - | - | - |
| RegularExpression.RE 1.1 (no longer available) | no | ? | - | - | - |
| gnu.rex.Rex ? (no longer available) | no | ? | - | - | - |
| monq.jfa.Regexp 1.1.1 (no longer available) | no | ? | - | - | - |
| [com.ibm.icu.text.UnicodeSet (ICU4J)](http://site.icu-project.org/) 56.1 (2015) | yes | ICU License | - | - | - |

If you want to re-run the tests, check out the source code and run it as follows:

````
# we need to skip tests since almost all libraries fail a test or an other
mvn -Dmaven.test.skip=true clean package
# run the benchmarks
java -cp lib/jint.jar:target/benchmarks.jar net.greypanther.javaadvent.regex.RegexBenchmarks
````

Find the complete source for the benchmarks on GitHub: https://github.com/gpanther/regex-libraries-benchmarks