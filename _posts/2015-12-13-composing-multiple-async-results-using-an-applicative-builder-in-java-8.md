---
id: 292
title: Composing Multiple Async Results via an Applicative Builder in Java 8
date: 2015-12-13T01:00:48+00:00
author: Guillermo Szeliga
layout: post
guid: http://www.javaadvent.com/?p=292
permalink: /2015/12/composing-multiple-async-results-using-an-applicative-builder-in-java-8.html
categories:
  - 2015
  - functional programming
  - java 8
  - Java Advent
  - jdk 8
  - lambda
  - parallel
  - threading
tags:
  - applicative
  - async
  - builder
  - CompletableFuture
  - Java 8
  - monads
---
A few months ago, I put out a <a href="http://covariantblabbering.blogspot.com.es/2015/07/wanna-annihilate-exceptions-roll-out.html">publication</a> where I explain in detail an abstraction I came up with named <a href="https://gist.github.com/gszeliga/e81341dc165a2b3f1fa1">Outcome</a>, which helped me <em><strong>A LOT</strong></em> to code <span style="text-decoration: underline;"><em>without side-effects</em></span> by enforcing the use of <strong><em><span style="text-decoration: underline;">semantics</span></em></strong>. By following this simple (and yet powerful) convention, I ended up turning any kind of failure (a.k.a. Exception) into an explicit result from a function, making everything much easier to reason about. I don't know you but I was tired of dealing with exceptions that teared everything down, so I did something about it, and to be honest, it worked really well. So before I keep going with my<em> tales from the trenches</em>, I really recommend going over that post. Now let's solve some asynchronous issues by using eccentric applicative ideas, shall we?
<h3>Something wicked this way comes</h3>
<p style="text-align: left;">Life was real good, our coding was fast-paced,  cleaner and composable as ever, but, out of the blue, we stumble upon a "missing" feature (evil laughs please): we needed to combine several <span style="text-decoration: underline;"><em>asynchronous</em></span> <strong><em>Outcome</em></strong> instances in a non-blocking fashion....</p>
<p style="text-align: center;"><a href="http://www.javaadvent.com/content/uploads/2015/12/ohgodwhy.jpg"><img class="wp-image-309 aligncenter" src="http://www.javaadvent.com/content/uploads/2015/12/ohgodwhy-300x281.jpg" alt="ohgodwhy" width="293" height="274" /></a></p>
Excited by the idea, I got down to work. I experimented for a fair amount of time seeking for a robust and yet simple way of expressing these kind of situations; while the new <em><strong>ComposableFuture</strong></em> API turned out to be much nicer that I expected (though I still don't understand why they decided to use names like <em>applyAsync </em> or <em>thenComposeAsync</em> instead of<em> map </em>or<em> flatMap</em>), I always ended up with implementations too verbose and repetitive comparing to some stuff I did with <em>Scala</em>, but after some long "<a href="http://www.noticiario-sur.com.ar/wp-content/uploads/2012/01/Mate-drink-2.jpg">Mate</a>" sessions, I had my "Hey! moment": Why not using something similar to an <span style="text-decoration: underline;"><em>applicative</em></span>?
<h3>The problem</h3>
Suppose that we have these two asynchronous results

https://gist.github.com/gszeliga/d4637bd7ba5ac7fd3838

and a silly entity called Message

https://gist.github.com/gszeliga/2f87c9a7bfd94386406d

I need something that given <em>textf</em> and <em>numberf</em> it will give me back something like
<pre>//After combining textf and numberf
CompletableFuture&lt;Outcome&lt;Message&gt;&gt; message = ....</pre>
So I wrote a letter to Santa Claus:
<ol>
	<li>I want to asynchronously format the string returned by <span style="text-decoration: underline;"><em>textf</em></span> using the number returned by <em><span style="text-decoration: underline;">numberf </span></em>only when both values are available, meaning that both <span style="text-decoration: underline;"><em>futures completed successfully</em></span> and none of the <span style="text-decoration: underline;"><em>outcomes did fail.</em></span><em> Of course, we need to be non</em><em>-blocking.</em><span style="text-decoration: underline;"><em>
</em></span></li>
	<li>In case of failures, I want to collect all failures that took place during the execution of <span style="text-decoration: underline;"><em>textf</em></span> and/or <span style="text-decoration: underline;"><em>numberf </em></span>and return them to the caller, again, without blocking at all.</li>
	<li>I don't want to be constrained by the number of values to be combined,  it must be capable of handling a fair amount of asynchronous results. Did I say without blocking? There you go...</li>
	<li style="text-align: left;">Not die during the attempt.</li>
</ol>
<a href="http://www.javaadvent.com/content/uploads/2015/12/waaat.jpg"><img class=" wp-image-368 aligncenter" src="http://www.javaadvent.com/content/uploads/2015/12/waaat-300x225.jpg" alt="waaat" width="355" height="266" /></a>
<h3>Applicative  builder to the rescue</h3>
If you think about it, one simple way to put what we're trying to achieve is as follows:
<pre>// Given a String -&gt; Given a number -&gt; Format the message
f: String -&gt; Integer -&gt; Message</pre>
Checking the definition of  <strong><em>f</em></strong>, it is saying something like: "Given a <em>String</em>, I will return a function that takes an <em>Integer</em> as parameter, that when applied, will return an instance of type <em>Message</em>", this way, instead of waiting for all values to be available at once, we can partially apply one value at a time, getting an actual description of the construction process of a <em>Message</em> instance. That sounded great.

To achieve that, it would be really awesome if we could take the construction lambda <strong><em>Message:new</em></strong> and <a href="https://en.wikipedia.org/wiki/Currying">curry</a> it, boom!, done!, but in Java that's impossible (to do in a generic, beautiful and concise way), so for the sake of our example, I decided to go with our beloved <em>Builder</em> pattern, which kinda does the job:

https://gist.github.com/gszeliga/ecc6baaccff8580d6800

And here's the WannabeApplicative&lt;T&gt; definition
<pre>public interface WannabeApplicative&lt;V&gt;
{
    V apply();
}</pre>
<span style="text-decoration: underline;"><strong><em>Disclamer</em></strong></span>: <em>For those functional freaks out there, this is not an applicative per se, I'm aware of that, but I took some ideas from it an adapted them according to the tools that the language offered me out of the box. So, if you're feeling curious, go check this <a href="http://covariantblabbering.blogspot.com.es/2015/03/enhancing-shine-of-builder-pattern.html">post</a> for a more formal example.</em>

If you're still with me, we could agree that we've done nothing too complicated so far, but now we need to express a building step, which, remember, needs to be <em>non-blocking</em> and capable to combine any previous failure that might have took place in other executions with potentially new ones. So, in order to do that, I came up with something as follows:

https://gist.github.com/gszeliga/ce7c95f4aa7f98e4bce4

First of all, we've got two functional interfaces: one is <em>Partial&lt;B&gt;</em>, which represents a <span style="text-decoration: underline;"><em>lazy application of a value to a builder</em></span>, and the second one, <em>MergingStage&lt;B,V&gt;</em>, represents the<span style="text-decoration: underline;"><em> "how" to combine both the builder and the value</em></span>. Then, we've got a method called <em>value</em> that, given an instance of type <em>CompletableFuture&lt;Outcome&lt;V&gt;&gt;</em>, it will return an instance of type <em>MergingStage&lt;B,V&gt;</em>, and believe or not, here's where the magic takes place. If you remember the <em>MergingState</em> definition, you'll see it's a <em>BiFunction</em>, where the first parameter is of type <em>Outcome&lt;B&gt;</em> and the second one is of type <em>Outcome&lt;V&gt;</em>. Now, if you follow the types, you can tell that we've got two things: the partial state of the building process on one side (type parameter B)  and a new value that need to be applied to the current state of the builder (type parameter V), so that, when applied, it will generate a new builder instance with the "next state in the building sequence", which is represented by <em>Partial&lt;B&gt;</em>. Last but not least, we've got the <em>stickedTo</em> method, which basically is a (awful java) hack to stick to a specific applicative type (builder) while defining building step. For instance, having:

https://gist.github.com/gszeliga/20d1e4123a1ca4ec7421

I can define partial value applications to any <em>Builder</em> instance as follows:

https://gist.github.com/gszeliga/d1e79a16d1f2acec19c0

See that we haven't built anything yet, we <span style="text-decoration: underline;"><em>just described what we want to do with each value when the time comes</em></span>, we might want to perform some validations before using the new value (here's when <em>Outcome</em> plays an important role) or just use it as it is, it's really up to us, but the main point is that we haven't applied anything yet. In order to do so, and to finally tight up all loose ends, I came up with some other definition, which looks as follows:

https://gist.github.com/gszeliga/6a5fcd0849a70f901dc5

Hope it's not that overwhelming, but I'll try to break it down as clearer as possible. In order to start specifying how you're going to combine the whole thing together, you will start by calling <em>begin</em> with an instance of type <em>WannabeApplicative&lt;V&gt;</em>, which, in our case, type parameter V is equal to <em>Builder</em>.
<pre>FutureCompositions&lt;Message, Builder&gt; ab = begin(Message.applicative())</pre>
See that, after you invoke <em>begin</em>, you will get a new instance of <em>FutureCompositions</em> with a <span style="text-decoration: underline;"><em>lazily evaluated partial state</em></span> inside of it, making it the <span style="text-decoration: underline;"><em>one and only owner of the whole building process state,</em></span> and that was the ultimate goal of everything we've done so far, to fully gain control over when and how things will be combined. Next, we must specify the values that we want to combine, and that's what the <em>binding</em> method is for:
<pre>ab.binding(textToApply)
  .binding(numberToApply);</pre>
This is how we supply our builder instance with all the values that need to be merged together along with the specification of what's supposed to happen with each one of them, by using our previously defined <em>Partial</em> instances. Also see that everything's still lazy evaluated, nothing has happened yet, but still we stacked all "steps" until we finally decide to materialize the result, which will happen when you call <em>perform</em>.
<pre>CompletableFuture&lt;Outcome&lt;Message&gt;&gt; message = ab.perform();</pre>
From that very moment everything will unfold,  each building stage will get evaluated, where failures could be returned and collected within an <em>Outcome</em> instance or simply the newly available values will be supplied to the target builder instance, one way or the other, all steps will be executed until nothing's to be done. I will try to depict what just happened as follows

<a href="http://www.javaadvent.com/content/uploads/2015/12/applicative.png"><img class=" wp-image-550 aligncenter" src="http://www.javaadvent.com/content/uploads/2015/12/applicative-300x182.png" alt="applicative" width="652" height="396" /></a>

If you pay attention to the left side of the picture, you can easily see how each step gets "defined" as I showed before, following the previous "declaration" arrow direction, meaning, how you actually described the building process. Now, from the moment that you call <em>perform</em>, each applicative instance (remember <em>Builder</em> in our case) will be lazily evaluated in the opposite direction:  it will start by evaluating the last specified stage in the stack, which will then proceed to evaluate the next one and so forth up to the point where we reach the "beginning" of the building definition, where it will start to unfold o roll out evaluation each step up to the top, collecting everything  it can by using the <em>MergingStage</em> specification.
<h3>And this is just the beginning....</h3>
I'm sure a lot could be done to improve this idea, for example:
<ul>
	<li>The two consecutive calls to <em>dependingOn </em>at<em> CompositionSources.values() <span style="text-decoration: underline;"><strong>sucks</strong></span></em>, too verbose to my taste, I must do something about it.</li>
	<li>I'm not quite sure to keep passing <em>Outcome</em> instances to a <em>MergingStage</em>, it would look cleaner and easier if we unwrap the values to be merged before invoking it and just return <em>Either&lt;Failure,V&gt;</em> instead - this will reduce complexity and increase flexibility on what's supposed to happen behind the scenes.</li>
	<li>Though using the Builder pattern did the job, it feels <strong><em>old-school</em></strong>, I would love to easily curry constructors, so in my to-do list is to check if <a href="https://github.com/jOOQ/jOOL">jOOλ</a> or <a href="https://github.com/javaslang/javaslang">Javaslang</a> have something to offer on that matter.</li>
	<li>Better type inference so that the any unnecessary noise gets remove from the code, for example, the <em>stickedTo</em> method, it really is a code smell, something that I hated from the first place. Definitely need more time to figure out an alternative way to infer the applicative type from the definition itself.</li>
</ul>
You're more than welcome to send me any suggestions and comments you might have. Cheers and remember.....

<a href="http://www.javaadvent.com/content/uploads/2015/12/index.jpeg"><img class="size-medium wp-image-553 aligncenter" src="http://www.javaadvent.com/content/uploads/2015/12/index-300x300.jpeg" alt="index" width="300" height="300" /></a>
<p style="text-align: left;"><a href="https://twitter.com/gszeliga">@gszeliga</a></p>

<h6>Sources</h6>
<ul>
	<li><a href="https://github.com/gszeliga/blogstuff/blob/master/src/main/java/outcome/Outcome.java">Outcome</a></li>
	<li><em><a href="https://github.com/gszeliga/blogstuff/blob/master/src/main/java/outcome/Futures.java">FutureCompositions and CompositionSources</a></em></li>
	<li><a href="https://github.com/gszeliga/blogstuff/blob/master/src/test/java/outcome/TestFutures.java">Wannabe applicative usage samples</a></li>
</ul>
&nbsp;