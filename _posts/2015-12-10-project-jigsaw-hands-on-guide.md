---
id: 480
title: Project Jigsaw Hands-On Guide
date: 2015-12-10T01:00:53+00:00
author: Nicolai Parlog
layout: post
guid: http://www.javaadvent.com/?p=480
permalink: /2015/12/project-jigsaw-hands-on-guide.html
dsq_thread_id:
  - 4962579378
categories:
  - JDK 9
tags:
  - Java 9
  - Project Jigsaw
---
Project Jigsaw will bring modularization to the Java platform and according to <a title="Proposed schedule for JDK 9" href="http://mail.openjdk.java.net/pipermail/jdk9-dev/2015-May/002172.html">the original plan</a> it was going to be feature complete on the 10th of December. So here we are but where is Jigsaw?

Surely a lot happened in the last six months: The <a title="Project Jigsaw: Early-Access Builds" href="http://openjdk.java.net/projects/jigsaw/ea">prototype came out</a>, the looming removal of internal APIs <a title="Removal of sun.misc.Unsafe in Java 9 - A disaster in the making" href="http://blog.dripstat.com/removal-of-sun-misc-unsafe-a-disaster-in-the-making/">caused quite a ruckus</a>, the <a title="jigsaw-dev mailing list" href="http://mail.openjdk.java.net/mailman/listinfo/jigsaw-dev">mailing list</a> is full of <a title="is ClassLoader.loadClass() supposed to work on module-info classes?" href="http://mail.openjdk.java.net/pipermail/jigsaw-dev/2015-December/005511.html">critical discussions</a> about the project's design decisions, and JavaOne saw <a title="JavaOne posts on CodeFX" href="http://blog.codefx.org/tag/javaone/">a series of great introductory talks</a> by the Jigsaw team. And <a title="Six-Month Delay Of Java 9 Release" href="http://blog.codefx.org/java/dev/delay-of-java-9-release/">then Java&nbsp;9 got delayed for half year</a> due to Jigsaw.

But let's ignore all of that for now and just focus on the code. In this post we'll take an existing demo application and modularize it with Java&nbsp;9. If you want to follow along, head over <a title="Jigsaw Advent Calendar" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar">to GitHub</a>, where all of the code can be found. The <a title="Jigsaw Advent Calendar - Setup" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/master#setup">setup instructions</a> are important to get the scripts running with Java&nbsp;9. For brevity, I removed the prefix <code>org.codefx.demo</code> from all package, module, and folder names an this article.



<h2>The Application Before Jigsaw</h2>

Even though I do my best to ignore the whole Christmas kerfuffle, it seemed prudent to have the demo uphold the spirit of the season. So it models an advent calendar:

<ul>
	<li>There is a calendar, which has 24 calendar sheets.</li>
	<li>Each sheet knows its day of the month and contains a surprise.</li>
	<li>The death march towards Christmas is symbolized by printing the sheets (and thus the surprises) to the console.</li>
</ul>

Of course the calendar needs to be created first. It can do that by itself but it needs a way to create surprises. To this end it gets handed a list of surprise factories. This is what <a title="Jigsaw Advent Calendar - Before Jigsaw: Main::main" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/blob/00-before-jigsaw/src/org.codefx.demo.advent/org/codefx/demo/advent/Main.java#L11-L18">the <code>main</code> method</a> looks like:

<pre><tt>public static void main(String[] args) {
	List&lt;SurpriseFactory&gt; surpriseFactories = Arrays.asList(
			new ChocolateFactory(),
			new QuoteFactory()
	);
	Calendar calendar =
		Calendar.createWithSurprises(surpriseFactories);
	System.out.println(calendar.asText());
}</tt></pre>

The <a title="Jigsaw Advent Calendar - Before Jigsaw" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/00-before-jigsaw">initial state of the project</a> is by no means the best of what is possible before Jigsaw. Quite the contrary, it is a simplistic starting point. It consists of a single module (in the abstract sense, not the Jigsaw interpretation) that contains <a title="Jigsaw Advent Calendar - Before Jigsaw: Types" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/00-before-jigsaw/src/org.codefx.demo.advent/org/codefx/demo/advent">all required types</a>:

<ul>
	<li>"Surprise API" - <code>Surprise</code> and <code>SurpriseFactory</code> (both are interfaces)</li>
	<li>"Calendar API" - <code>Calendar</code> and <code>CalendarSheet</code> to create the calendar</li>
	<li>Surprises - a couple of <code>Surprise</code> and <code>SurpriseFactory</code> implementations</li>
	<li>Main - to wire up and run the whole thing.</li>
</ul>

<a title="Jigsaw Advent Calendar - Before Jigsaw: Compile And Run" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/blob/00-before-jigsaw/compileAndRun.sh">Compiling and running</a> is straight forward (commands for Java&nbsp;8):

<pre><tt># compile
javac -d classes/advent ${source files}
# package
jar -cfm jars/advent.jar ${manifest and compiled class files}
# run
java -jar jars/advent.jar</tt></pre>



<h2>Entering Jigsaw Land</h2>

The <a title="Jigsaw Advent Calendar - Creating A Module" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/01-creating-a-module">next step</a> is small but important. It changes nothing about the code or its organization but moves it into a Jigsaw module.

<h3>Modules</h3>

So what's a module? To quote the highly recommended <a href="http://openjdk.java.net/projects/jigsaw/spec/sotms/" title="The State of the Module System">State of the Module System</a>:

<blockquote>
A <em>module</em> is a named, self-describing collection of code and data. Its code is organized as a set of packages containing types, i.e., Java classes and interfaces; its data includes resources and other kinds of static information.

To control how its code refers to types in other modules, a module declares which other modules it <em>requires</em> in order to be compiled and run. To control how code in other modules refers to types in its packages, a module declares which of those packages it <em>exports</em>.
</blockquote>

So compared to a JAR a module has a name that is recognized by the JVM, declares which other modules it depends on and defines which packages are part of its public API.

<h4>Name</h4>

A module's name can be arbitrary. But to ensure uniqueness it is recommended to stick with the inverse-URL naming schema of packages. So while this is not necessary it will often mean that the module name is a prefix of the packages it contains.

<h4>Dependencies</h4>

A module lists the other modules it depends on to compile and run. This is true for application and library modules but also for modules in the JDK itself, which was split up into about 80 of them (have a look at them with <code>java -listmods</code>).

Again from the design overview:

<blockquote>
When one module depends directly upon another in the module graph then code in the first module will be able to refer to types in the second module. We therefore say that the first module <em>reads</em> the second or, equivalently, that the second module is <em>readable</em> by the first.

[...]

The module system ensures that every dependence is fulfilled by precisely one other module, that no two modules read each other, that every module reads at most one module defining a given package, and that modules defining identically-named packages do not interfere with each other.
</blockquote>

When any of the properties is violated, the module system refuses to compile or launch the code. This is an immense improvement over the brittle classpath, where e.g. missing JARs would only be discovered at runtime, crashing the application.

It is also worth to point out that a module is only able to access another's types if it directly depends on it. So if <em>A</em> depends on <em>B</em>, which depends on <em>C</em>, then <em>A</em> is unable to access <em>C</em> unless it requires it explicitly.

<h4>Exports</h4>

A module lists the packages it exports. Only public types in these packages are accessible from outside the module.

This means that <code>public</code> is no longer really public. A public type in a non-exported package is as hidden from the outside world as much as a non-public type in an exported package. Which is even more hidden than package-private types are today because the module system does not even allow reflective access to them. As Jigsaw is currently implemented command line flags are the only way around this.



<h3>Implementation</h3>

To be able to create a module, the project needs a <code>module-info.java</code> in its root source directory:

<pre><tt>module advent {
    // no imports or exports
}</tt></pre>

Wait, didn't I say that we have to declare dependencies on JDK modules as well? So why didn't we mention anything here? All Java code requires <code>Object</code> and that class, as well as the few others the demo uses, are part of the module <code>java.base</code>. So literally <em>every</em> Java module depends on <code>java.base</code>, which led the Jigsaw team to the decision to automatically require it. So we do not have to mention it explicitly.

The biggest change is the script to compile and run (commands for Java&nbsp;9):

<pre><tt># compile (include module-info.java)
javac -d classes/advent ${source files}
# package (add module-info.class and specify main class)
jar -c \
	--file=mods/advent.jar \
	--main-class=advent.Main \
	${compiled class files}
# run (specify a module path and simply name to module to run)
java -mp mods -m advent</tt></pre>

We can see that compilation is almost the same - we only need to include the new <code>module-info.java</code> in the list of classes.

The jar command will create a so-called modular JAR, i.e. a JAR that contains a module. Unlike before we need no manifest anymore but can specify the main class directly. Note how the JAR is created in the directory <code>mods</code>.

Utterly different is the way the application is started. The idea is to tell Java where to find the application modules (with <code>-mp&nbsp;mods</code>, this is called the <em>module path</em>) and which module we would like to launch (with <code>-m&nbsp;advent</code>).



<h2>Splitting Into Modules</h2>

Now it's time to really get to know Jigsaw and <a title="Jigsaw Advent Calendar - Splitting Into Modules" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/02-splitting-into-modules">split that monolith up</a> into separate modules.

<h3>Made-up Rationale</h3>

The "surprise API", i.e. <code>Surprise</code> and <code>SurpriseFactory</code>, is a great success and we want to separate it from the monolith.

The factories that create the surprises turn out to be very dynamic. A lot of work is being done here, they change frequently and which factories are used differs from release to release. So we want to isolate them.

At the same time we plan to create a large Christmas application of which the calendar is only one part. So we'd like to have a separate module for that as well.

We end up with these modules:

<ul>
	<li><em>surprise</em> - <code>Surprise</code> and <code>SurpriseFactory</code></li>
	<li><em>calendar</em> - the calendar, which uses the surprise API</li>
	<li><em>factories</em> - the <code>SurpriseFactory</code> implementations</li>
	<li><em>main</em> - the original application, now hollowed out to the class <code>Main</code></li>
</ul>

Looking at their dependencies we see that <em>surprise</em> depends on no other module. Both <em>calendar</em> and <em>factories</em> make use of its types so they must depend on it. Finally, <em>main</em> uses the factories to create the calendar so it depends on both.

<img src="http://www.javaadvent.com/content/uploads/2015/12/jigsaw-hands-on-splitting-into-modules.png" alt="jigsaw-hands-on-splitting-into-modules" width="588" height="167" class="aligncenter size-full wp-image-500" />

<h3>Implementation</h3>

The first step is to reorganize the source code. We'll stick with the directory structure as proposed by the <a title="Project Jigsaw: Module System Quick-Start Guide" href="http://openjdk.java.net/projects/jigsaw/quick-start">official quick start guide</a> and have all of our modules in their own folders below <code>src</code>:

<pre><tt>src
  - advent.calendar: the "calendar" module
      - org ...
      module-info.java
  - advent.factories: the "factories" module
      - org ...
      module-info.java
  - advent.surprise: the "surprise" module
      - org ...
      module-info.java
  - advent: the "main" module
      - org ...
      module-info.java
.gitignore
compileAndRun.sh
LICENSE
README</tt></pre>

To keep this readable I truncated the folders below <code>org</code>. What's missing are the packages and eventually the source files for each module. See it <a href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/02-splitting-into-modules" title="Jigsaw Advent Calendar - Splitting Into Modules">on GitHub</a> in its full glory.

Let's now see what those module infos have to contain and how we can compile and run the application.

<h4><em>surprise</em></h4>

There are no required clauses as <em>surprise</em> has no dependencies. (Except for <code>java.base</code>, which is always implicitly required.) It exports the package <code>advent.surprise</code> because that contains the two classes <code>Surprise</code> and <code>SurpriseFactory</code>.

So the <code>module-info.java</code> looks as follows:

<pre><tt>module advent.surprise {
	// requires no other modules
	// publicly accessible packages
	exports advent.surprise;
}</tt></pre>

Compiling and packaging is very similar to the previous section. It is in fact even easier because surprises contains no main class:

<pre><tt># compile
javac -d classes/advent.surprise ${source files}
# package
jar -c --file=mods/advent.surprise.jar ${compiled class files}</tt></pre>

<h4><em>calendar</em></h4>

The calendar uses types from the surprise API so the module must depend on <em>surprise</em>. Adding <code>requires advent.surprise</code> to the module achieves this.

The module's API consists of the class <code>Calendar</code>. For it to be publicly accessible the containing package <code>advent.calendar</code> must be exported. Note that <code>CalendarSheet</code>, private to the same package, will not be visible outside the module.

But there is an additional twist: We just made <a href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/blob/02-splitting-into-modules/src/org.codefx.demo.advent.calendar/org/codefx/demo/advent/calendar/Calendar.java#L22"><code>Calendar.createWithSurprises(<code>List&lt;SurpriseFactory&gt;</code>)</code></a> publicly available, which exposes types from the <em>surprise</em> module. So unless modules reading <em>calendar</em> also require <em>surprise</em>, Jigsaw will prevent them from accessing these types, which would lead to compile and runtime errors.

Marking the requires clause as <code>public</code> fixes this. With it any module that depends on <em>calendar</em> also reads <em>surprise</em>. This is called <em>implied readability</em>.

The final module-info looks as follows:

<pre><tt>module advent.calendar {
	// required modules
	requires public advent.surprise;
	// publicly accessible packages
	exports advent.calendar;
}</tt></pre>

Compilation is almost like before but the dependency on <em>surprise</em> must of course be reflected here. For that it suffices to point the compiler to the directory <code>mods</code> as it contains the required module:

<pre><tt># compile (point to folder with required modules)
javac -mp mods \
	-d classes/advent.calendar \
	${source files}
# package
jar -c \
	--file=mods/advent.calendar.jar \
	${compiled class files}</tt></pre>

<h4><em>factories</em></h4>

The factories implement <code>SurpriseFactory</code> so this module must depend on <em>surprise</em>. And since they return instances of <code>Surprise</code> from published methods the same line of thought as above leads to a <code>requires public</code> clause.

The factories can be found in the package <code>advent.factories</code> so that must be exported. Note that the public class <code>AbstractSurpriseFactory</code>, which is found in another package, is not accessible outside this module.

So we get:

<pre><tt>module advent.factories {
	// required modules
	requires public advent.surprise;
	// publicly accessible packages
	exports advent.factories;
}</tt></pre>

Compilation and packaging is analog to <em>calendar</em>.

<h4><em>main</em></h4>

Our application requires the two modules <em>calendar</em> and <em>factories</em> to compile and run. It has no API to export.

<pre><tt>module advent {
	// required modules
	requires advent.calendar;
	requires advent.factories;
	// no exports
}</tt></pre>

Compiling and packaging is like with last section's single module except that the compiler needs to know where to look for the required modules:

<pre><tt>#compile
javac -mp mods \
	-d classes/advent \
	${source files}
# package
jar -c \
	--file=mods/advent.jar \
	--main-class=advent.Main \
	${compiled class files}
# run
java -mp mods -m advent</tt></pre>



<h2>Services</h2>

Jigsaw enables loose coupling by implementing the <a title="Service Locator Pattern" href="https://en.wikipedia.org/wiki/Service_locator_pattern">service locator pattern</a>, where the module system itself acts as the locator. Let's see <a title="Jigsaw Advent Calendar - Services" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/tree/03-services">how that goes</a>.

<h3>Made-up Rationale</h3>

Somebody recently read a blog post about how cool loose coupling is. Then she looked at our code from above and complained about the tight relationship between <em>main</em> and <em>factories</em>. Why would <em>main</em> even know <em>factories</em>?

Because...

<pre><tt>public static void main(String[] args) {
	List&lt;SurpriseFactory&gt; surpriseFactories = Arrays.asList(
			new ChocolateFactory(),
			new QuoteFactory()
	);
	Calendar calendar =
		Calendar.createWithSurprises(surpriseFactories);
	System.out.println(calendar.asText());
}</tt></pre>

Really? Just to instantiate some implementations of a perfectly fine abstraction (the <code>SurpriseFactory</code>)?

And we know she's right. Having someone else provide us with the implementations would remove the direct dependency. Even better, if said middleman would be able to find <em>all</em> implementations on the module path, the calendar's surprises could easily be configured by adding or removing modules before launching.

This is indeed possible with Jigsaw. We can have a module specify that it provides implementations of an interface. Another module can express that it uses said interface and find all implementations with the <code>ServiceLocator</code>.

We use this opportunity to split <em>factories</em> into <em>chocolate</em> and <em>quote</em> and end up with these modules and dependencies:

<ul>
	<li><em>surprise</em> - <code>Surprise</code> and <code>SurpriseFactory</code></li>
	<li><em>calendar</em> - the calendar, which uses the surprise API</li>
	<li><em>chocolate</em> - the <code>ChocolateFactory</code> as a service</li>
	<li><em>quote</em> - the <code>QuoteFactory</code> as a service</li>
	<li><em>main</em> - the application; no longer requires individual factories</li>
</ul>

<img src="http://www.javaadvent.com/content/uploads/2015/12/jigsaw-hands-on-services.png" alt="jigsaw-hands-on-services" width="684" height="258" class="aligncenter size-full wp-image-499" />

<h3>Implementation</h3>

The first step is to reorganize the source code. The only change from before is that <code>src/advent.factories</code> is replaced by <code>src/advent.factory.chocolate</code> and <code>src/advent.factory.quote</code>.

Lets look at the individual modules.

<h4><em>surprise</em> and <em>calendar</em></h4>

Both are unchanged.

<h4><em>chocolate</em> and <em>quote</em></h4>

Both modules are identical except for some names. Let's look at <em>chocolate</em> because it's more yummy.

As before with <em>factories</em> the module <code>requires public</code> the <em>surprise</em> module.

More interesting are its exports. It provides an implementation of <code>SurpriseFactory</code>, namely <code>ChocolateFactory</code>, which is specified as follows:

<pre><tt>provides advent.surprise.SurpriseFactory
	with advent.factory.chocolate.ChocolateFactory;</tt></pre>

Since this class is the entirety of its public API it does not need to export anything else. Hence no other export clause is necessary.

We end up with:

<pre><tt>module advent.factory.chocolate {
	// list the required modules
	requires public advent.surprise;
	// specify which class provides which service
	provides advent.surprise.SurpriseFactory
		with advent.factory.chocolate.ChocolateFactory;
}</tt></pre>

Compilation and packaging is straight forward:

<pre><tt>javac -mp mods \
	-d classes/advent.factory.chocolate \
	${source files}
jar -c \
	--file mods/advent.factory.chocolate.jar \
	${compiled class files}</tt></pre>

<h4><em>main</em></h4>

The most interesting part about <em>main</em> is how it uses the ServiceLocator to find implementation of SurpriseFactory. From <a title="Jigsaw Advent Calendar - Services: Main::main" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar/blob/03-services/src/org.codefx.demo.advent/org/codefx/demo/advent/Main.java#L13-L14">its main method</a>:

<pre><tt>List surpriseFactories = new ArrayList&lt;&gt;();
ServiceLoader.load(SurpriseFactory.class)
	.forEach(surpriseFactories::add);</tt></pre>

Our application now only requires <em>calendar</em> but must specify that it uses <code>SurpriseFactory</code>. It has no API to export.

<pre><tt>module advent {
	// list the required modules
	requires advent.calendar;
	// list the used services
	uses advent.surprise.SurpriseFactory;
	// exports no functionality
}</tt></pre>

Compilation and execution are like before.

And we can indeed change the surprises the calendar will eventually contain by simply removing one of the factory modules from the module path. Neat!



<h2>Summary</h2>

So that's it. We have seen how to move a monolithic application into a single module and how we can split it up into several. We even used a service locator to decouple our application from concrete implementations of services. All of this is <a title="Jigsaw Advent Calendar on GitHub" href="https://github.com/CodeFX-org/demo-jigsaw-advent-calendar">on GitHub</a> so check it out to see more code!

But there is lots more to talk about! Jigsaw brings <a title="How Java 9 And Project Jigsaw May Break Your Code" href="http://blog.codefx.org/java/dev/how-java-9-and-project-jigsaw-may-break-your-code/">a couple of incompatibilities</a> but also the means to solve many of them. And we haven't talked about how reflection interacts with the module system and how to migrate external dependencies.

If these topics interest you, watch <a title="CodeFX - Project Jigsaw" href="http://blog.codefx.org/tag/project-jigsaw/">the Jigsaw tag</a> on my blog as I will surely write about them over the coming months.