---
id: 734
title: Effective UI tests with Selenide
date: 2015-12-23T00:00:46+00:00
author: Andrei Solntsev
layout: post
guid: http://www.javaadvent.com/?p=734
permalink: /2015/12/effective-ui-tests-with-selenide.html
categories:
  - Uncategorized
tags:
  - java
  - Java Advent
  - selenide
  - selenium
  - TDD
  - testing
---
<h3>Waiting for miracles</h3>
Christmas is a time for miracles. On the eve of the new year we all build plans for the next. And we hope that all problems will leave in the ending year, and a miracle happens in the coming year.

Every Java developer dreams about a miracle that lets him become The Most Effective Java Developer in the world.

I want to show you such a miracle.

It's called <em>automated tests</em>!
<h3>Ugh, tests?</h3>
Yes. You will not become a real master thanks to micro/pico/nano services. You will become a real master thanks to discipline. Discipline claiming that developer only then reports jobs as <em>done</em>when code <strong>and tests</strong> are written and run.
<h3>But, isn't testing boring?</h3>
Oh no, believe me! Writing of <strong>fast</strong> and <strong>stable</strong> automated tests is a great challenge for smartest heads. And it can be very fun and interesting. You only need to use right tools.

The right tool for writing UI tests is:
<h2>Selenide</h2>
Selenide is an open-source library for writing concise and stable UI tests.

Selenide is an ideal choice for software developers because it has a very low learning curve. Thus, you don't need to bother with browser details, all these typical ajax and time issues that eat most of QA automation engineers' time.

Let's look at a simplest Selenide test:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="kd">public</span> <span class="kd">class</span> <span class="nc">GoogleTest</span> <span class="o">{</span>
  <span class="nd">@Test</span>
  <span class="kd">public</span> <span class="kt">void</span> <span class="nf">user_can_search_everything_in_google</span><span class="o">()</span> <span class="o">{</span>
    <span class="n">open</span><span class="o">(</span><span class="s">"http://google.com/ncr"</span><span class="o">);</span>
    <span class="n">$</span><span class="o">(</span><span class="n">By</span><span class="o">.</span><span class="na">name</span><span class="o">(</span><span class="s">"q"</span><span class="o">)).</span><span class="na">val</span><span class="o">(</span><span class="s">"selenide"</span><span class="o">).</span><span class="na">pressEnter</span><span class="o">();</span>

    <span class="n">$$</span><span class="o">(</span><span class="s">"#ires .g"</span><span class="o">).</span><span class="na">shouldHave</span><span class="o">(</span><span class="n">size</span><span class="o">(</span><span class="mi">10</span><span class="o">));</span>

    <span class="n">$</span><span class="o">(</span><span class="s">"#ires .g"</span><span class="o">).</span><span class="na">shouldBe</span><span class="o">(</span><span class="n">visible</span><span class="o">).</span><span class="na">shouldHave</span><span class="o">(</span>
        <span class="n">text</span><span class="o">(</span><span class="s">"Selenide: concise UI tests in Java"</span><span class="o">),</span>
        <span class="n">text</span><span class="o">(</span><span class="s">"selenide.org"</span><span class="o">));</span>
  <span class="o">}</span>
<span class="o">}</span>
</code></pre>
</div>
Let's look closer what happens here.
<ul>
	<li>You <strong>open a browser</strong> with just one command <code>open(url)</code></li>
	<li>You <strong>find an element</strong> on a page with command <code>$</code>.
You can find element by name, ID, CSS selector, attributes, xpath and even by text.</li>
	<li>You <strong>manipulate the element</strong>: enter some text with <code>val()</code> and press enter with (surprise-surprise!) <code>pressEnter()</code>.</li>
	<li>You <strong>check the results</strong>: find all found results with <code>$$</code> (it returns a collection of all matched elements). You check the size and content of the collection.</li>
</ul>
Isn't this test easy to read? Isn't this test easy to write?

I believe it is.
<h2>Deeper into details</h2>
<h3>Ajax/timing problems</h3>
Nowdays web applications are dynamic. Every single piece of application can be rendered/changed dynamically at any moment. This creates a lot of problems for automated tests. Test that is green today can suddenly become red at any moment, just because browser executed some javascript a little bit longer than usual.

It's a real <em>pain in the ajjaxx</em>.

Quite unbelievably, but Selenide resolves most of the these problems in a very simple way.

Simply said, every <strong>Selenide method waits</strong> a little bit <em>if needed</em>. People call it "smart waiting".

When you write
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"#menu"</span><span class="o">).</span><span class="na">shouldHave</span><span class="o">(</span><span class="n">text</span><span class="o">(</span><span class="s">"Hello"</span><span class="o">));</span>
</code></pre>
</div>
Selenide checks if the element exists and contains text "Hello".

If not yet, Selenide assumes that probably the element will be updated dynamically soon, and waits a little bit until it happens. The default timeout is 4 seconds, which is typically enough for most web applications. And of course, it's configurable.
<h3>Rich set of matchers</h3>
You can check pretty much everything with Selenide. Using "smart waiting" mechanism mentioned above.

For example, you can check if element exists. If not yet, Selenide will wait <strong>up to</strong> 4 seconds.
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">".loading_progress"</span><span class="o">).</span><span class="na">shouldBe</span><span class="o">(</span><span class="n">visible</span><span class="o">);</span>
</code></pre>
</div>
You can even check that element <strong>does not</strong> exist. If it still exists, Selenide will wait up to 4 seconds until it disappears.
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="n">By</span><span class="o">.</span><span class="na">name</span><span class="o">(</span><span class="s">"gender"</span><span class="o">)).</span><span class="na">should</span><span class="o">(</span><span class="n">disappear</span><span class="o">);</span>
</code></pre>
</div>
And you can use fluent API and chain methods to make your tests really concise:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"#menu"</span><span class="o">)</span>
  <span class="o">.</span><span class="na">shouldHave</span><span class="o">(</span><span class="n">text</span><span class="o">(</span><span class="s">"Hello"</span><span class="o">),</span> <span class="n">text</span><span class="o">(</span><span class="s">"John!"</span><span class="o">))</span>
  <span class="o">.</span><span class="na">shouldBe</span><span class="o">(</span><span class="n">enabled</span><span class="o">,</span> <span class="n">selected</span><span class="o">);</span>
</code></pre>
</div>
<h3>Collections</h3>
Selenide allows you to work with collections, thus checking a lot of elements with one line of code.

For example, you can check that there are exactly N elements on a page:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$$</span><span class="o">(</span><span class="s">".error"</span><span class="o">).</span><span class="na">shouldHave</span><span class="o">(</span><span class="n">size</span><span class="o">(</span><span class="mi">3</span><span class="o">));</span>
</code></pre>
</div>
You can find subset of collections:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$$</span><span class="o">(</span><span class="s">"#employees tbody tr"</span><span class="o">)</span>
  <span class="o">.</span><span class="na">filter</span><span class="o">(</span><span class="n">visible</span><span class="o">)</span>
  <span class="o">.</span><span class="na">shouldHave</span><span class="o">(</span><span class="n">size</span><span class="o">(</span><span class="mi">4</span><span class="o">));</span>
</code></pre>
</div>
You can check texts of elements. In most cases, it's sufficient to check the whole table or table row:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$$</span><span class="o">(</span><span class="s">"#employees tbody tr"</span><span class="o">).</span><span class="na">shouldHave</span><span class="o">(</span>
  <span class="n">texts</span><span class="o">(</span>
      <span class="s">"John Belushi"</span><span class="o">,</span>
      <span class="s">"Bruce Willis"</span><span class="o">,</span>
      <span class="s">"John Malkovich"</span>
  <span class="o">)</span>
<span class="o">);</span>
</code></pre>
</div>
<h3>Upload/download files</h3>
It's pretty easy to upload a file with Selenide:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"#cv"</span><span class="o">).</span><span class="na">uploadFile</span><span class="o">(</span><span class="k">new</span> <span class="nf">File</span><span class="o">(</span><span class="s">"cv.doc"</span><span class="o">));</span>
</code></pre>
</div>
You can even upload multiple files at once:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"#cv"</span><span class="o">).</span><span class="na">uploadFile</span><span class="o">(</span>
  <span class="k">new</span> <span class="nf">File</span><span class="o">(</span><span class="s">"cv1.doc"</span><span class="o">),</span>
  <span class="k">new</span> <span class="nf">File</span><span class="o">(</span><span class="s">"cv2.doc"</span><span class="o">),</span>
  <span class="k">new</span> <span class="nf">File</span><span class="o">(</span><span class="s">"cv3.doc"</span><span class="o">)</span>
<span class="o">);</span>
</code></pre>
</div>
And it's unbelievably simple to download a file:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">File</span> <span class="n">pdf</span> <span class="o">=</span> <span class="n">$</span><span class="o">(</span><span class="s">".btn#cv"</span><span class="o">).</span><span class="na">download</span><span class="o">();</span>
</code></pre>
</div>
<h3>Testing "highly dynamic" web applications</h3>
Some web frameworks (e.g. GWT) generate HTML that is absolutely unreadable. Elements do not have constant IDs or names.

It's a real <em>pain in the xpathh</em>.

Selenide suggests to resolve this problem by searching elements by text.
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="kn">import</span> <span class="nn">static</span> <span class="n">com</span><span class="o">.</span><span class="na">codeborne</span><span class="o">.</span><span class="na">selenide</span><span class="o">.</span><span class="na">Selectors</span><span class="o">.*;</span>

<span class="n">$</span><span class="o">(</span><span class="n">byText</span><span class="o">(</span><span class="s">"Hello, Devoxx!"</span><span class="o">))</span>     <span class="c1">// find by the whole text</span>
   <span class="o">.</span><span class="na">shouldBe</span><span class="o">(</span><span class="n">visible</span><span class="o">);</span>

<span class="n">$</span><span class="o">(</span><span class="n">withText</span><span class="o">(</span><span class="s">"oxx"</span><span class="o">))</span>              /<span class="c1">/ find by substring</span>
   <span class="o">.</span><span class="na">shouldHave</span><span class="o">(</span><span class="n">text</span><span class="o">(</span><span class="s">"Hello, Devoxx!"</span><span class="o">));</span>
</code></pre>
</div>
Searching by text is not bad idea at all. In fact, I like it because it emulates behaviour of real user. Real user doesn't find buttons by ID or XPATH - he finds by text (or, well, color).

Another useful set of Selenide methods allows you to navigate between parents and children.
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"td"</span><span class="o">).</span><span class="na">parent</span><span class="o">()</span>
<span class="n">$</span><span class="o">(</span><span class="s">"td"</span><span class="o">).</span><span class="na">closest</span><span class="o">(</span><span class="s">"tr"</span><span class="o">)</span>
<span class="n">$</span><span class="o">(</span><span class="s">".btn"</span><span class="o">).</span><span class="na">closest</span><span class="o">(</span><span class="s">".modal"</span><span class="o">)</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">find</span><span class="o">(</span><span class="n">By</span><span class="o">.</span><span class="na">name</span><span class="o">(</span><span class="s">"q"</span><span class="o">))</span>
</code></pre>
</div>
For example, you can find a table cell by text, then by its closest <code>tr</code> descendant and find a "Save" button inside this table row:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"table#employees"</span><span class="o">)</span>
  <span class="o">.</span><span class="na">find</span><span class="o">(</span><span class="n">byText</span><span class="o">(</span><span class="s">"Joshua"</span><span class="o">))</span>
  <span class="o">.</span><span class="na">closest</span><span class="o">(</span><span class="s">"tr.employee"</span><span class="o">)</span>
  <span class="o">.</span><span class="na">find</span><span class="o">(</span><span class="n">byValue</span><span class="o">(</span><span class="s">"Save"</span><span class="o">))</span>
  <span class="o">.</span><span class="na">click</span><span class="o">();</span>
</code></pre>
</div>
<h3>... And many other functions</h3>
Selenide has many more functions, like:
<div class="highlight">
<pre><code class="language-java" data-lang="java"><span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">scrollTo</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">innerText</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">innerHtml</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">exists</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"select"</span><span class="o">).</span><span class="na">isImage</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"select"</span><span class="o">).</span><span class="na">getSelectedText</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"select"</span><span class="o">).</span><span class="na">getSelectedValue</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">doubleClick</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">contextClick</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">hover</span><span class="o">();</span>
<span class="n">$</span><span class="o">(</span><span class="s">"div"</span><span class="o">).</span><span class="na">dragAndDrop</span><span class="o">()</span>
<span class="n">zoom</span><span class="o">(</span><span class="mf">2.5</span><span class="o">);
</span>...</code></pre>
</div>
but the good news is that you don't need to remember all this stuff. Just put $, put dot and choose from available options suggested by your IDE.

Use the power of IDE! Concentrate on business logic.

<img src="http://selenide.org/images/ide-just-start-typing.png" alt="Power of IDE" />
<h1>Make the world better</h1>
I believe the World will get better when all developers start writing automated tests for their code. When developers will get up at 17:00 and go to their children without fearing that they broke something with last changes.

Let's make the world better by writing automated tests!

<center>Deliver working software.

</center><a href="https://twitter.com/asolntsev">Andrei Solntsev</a>

<a href="http://selenide.org/">selenide.org</a>