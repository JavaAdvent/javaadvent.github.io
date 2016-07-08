---
id: 356
title: 'Adopt OpenJDK &#038; Java community: how can you help Java !'
date: 2015-12-23T01:00:32+00:00
author: Mani Sarkar
layout: post
guid: http://www.javaadvent.com/?p=356
permalink: /2015/12/adopt-openjdk-java-community-how-can-you-help-java.html
categories:
  - 2015
  - adopt openjdk
  - developers
  - garbage collection
  - java
  - java 8
  - java 9
  - jdk
  - jdk 7
  - jdk 8
  - JDK 9
  - jtreg
  - JVM
  - open source
  - openjdk
  - Programming
  - test
  - testing
tags:
  - adopt openjdk
  - bash
  - communities
  - community
  - contribute
  - contribution
  - contributors
  - developers
  - devops
  - docker
  - G1GC
  - GC
  - hackday
  - Java 9
  - jdk
  - jug
  - JVM
  - openjdk
  - scripts
  - shenandoah
  - testing Java
  - vagrant
---
<h4><strong>Introduction</strong></h4>
<p style="text-align: justify;">I want to take the opportunity to show what we have been doing in last year and also what we have done so far as members of the community. Unlike other years I have decided to keep this post less technical compare to the past years and compared to the other posts on Java Advent this year.</p>
<p style="text-align: justify;"><a href="http://www.javaadvent.com/content/uploads/2015/12/InTheBeginning.jpg" rel="attachment wp-att-688"><img class="size-medium wp-image-688 alignleft" src="http://www.javaadvent.com/content/uploads/2015/12/InTheBeginning-300x162.jpg" alt="InTheBeginning" width="300" height="162" /></a></p>
<p style="text-align: justify;">This year marks the fourth year since the first <a href="http://www.meetup.com/Londonjavacommunity/events/42736622/" target="_blank">OpenJDK hackday</a> was held in London (supported by <a href="http://www.meetup.com/Londonjavacommunity/" target="_blank">LJC and its members</a>) and also when the <a href="http://adoptopenjdk.java.net/" target="_blank">Adopt OpenJDK</a> program was started. Four years is a small number on the face of <a href="http://cdn.knightlab.com/libs/timeline3/latest/embed/index.html?source=1kABwU2-m9OZav6PtW1xAiIbQXBe0EdClCVnDy4aKqvM&amp;font=Default&amp;lang=en&amp;initial_zoom=2&amp;height=650" target="_blank">20 years of Java</a>, same goes to the size of the <strong>Adopt OpenJDK</strong> community which forms a small part of the Java community (9+ million users). Although the post is non-technical in nature, the message herein is fairly important for the future growth and progress of our community and the next generation developers.</p>

<h4><strong>Creations of the community</strong></h4>
<img class="alignleft" src="https://groveonline.files.wordpress.com/2010/08/creation.jpg?w=660" alt="Creations from the community" width="279" height="183" />
<p style="text-align: justify;">Over the many months a number of members of our community contributed and passed on their good work to us. In no specific order I have enlisted these picking them from memory. I know there are more to name and you can help us by sharing those with us (we will enlist them here).  So here are some of those that we can talk about and be proud of, and thank those who were involved:</p>

<ul>
	<li style="text-align: justify;"><a href="https://java.net/projects/adoptopenjdk/pages/AdoptOpenJDK#Getting_Started" target="_blank">Getting Started</a> page - created to enabled two way communication with the members of the community, these include a mailing list, an IRC channel, a weekly newsletter, a twitter handle, among other social media channels and collaboration tools.</li>
	<li style="text-align: justify;"><a href="https://github.com/AdoptOpenJDK/jitwatch" target="_blank">Adopt OpenJDK project: jitwatch</a> - a great tool created by <a href="https://twitter.com/chriswhocodes" target="_blank">Chris Newland</a>, its one of its kind, ever growing with features and helping developers fine-tune the performance of your Java/JVM applications running on the JVM.</li>
	<li style="text-align: justify;"><a href="https://github.com/AdoptOpenJDK/adoptopenjdk-getting-started-kit" target="_blank">Adopt OpenJDK: GSK</a> - a community effort gathering knowledge and experience from hackday attendees and OpenJDK developers on how to go about with OpenJDK from building it to creating your own version of the JDK. Many JUG members have been involved in the process, and this is now a e-book available in many languages (5 languages + 2 to 3 more languages in progress).</li>
	<li style="text-align: justify;"><a href="https://github.com/AdoptOpenJDK/adopt-openjdk-kiss-vagrant" target="_blank">Adopt OpenJDK vagrant scripts</a> - a collection of vagrant scripts initially created by John Patrick from the LJC, later improved by the community members by adding more scripts and refactoring existing ones. Theses scripts help build OpenJDK projects in a virtualised container i.e. VirtualBox, making building, and testing OpenJDK and also running and testing Java/JVM applications much easier, reliable and in an isolated environment.</li>
	<li style="text-align: justify;"><a href="https://github.com/neomatrix369/BuildHelpers/tree/master/openjdk-docker" target="_blank">Adopt OpenJDK docker scripts</a> - a collection of docker scripts created with the help of the community, this is now also receiving contributions from a number of members like Richard Kolb (SA JUG). Just like the vagrant scripts mentioned above, the docker scripts have similar goals, and need your DevOps foo!</li>
	<li style="text-align: justify;"><a href="https://github.com/AdoptOpenJDK/mjprof" target="_blank">Adopt OpenJDK project: mjprof </a>- mjprof is a Monadic <em>jstack</em> analysis tool set. It is a fancy way to say it analyzes jstack output using a series of simple composable building blocks (monads). Many thanks to <a href="https://twitter.com/lifeyx" target="_blank">Haim Yadid</a> for donating it to the community.</li>
	<li style="text-align: justify;"><a href="https://github.com/AdoptOpenJDK/jcountdown" target="_blank">Adopt OpenJDK project: jcountdown </a>- built by the community that mimics the spirit of ie6countdown.net. That is, to encourage users to move to the latest and greatest Java! Many thanks to all those involved, you can already see from the <a href="https://github.com/AdoptOpenJDK/jcountdown/commits/master" target="_blank">commit history</a>.</li>
	<li style="text-align: justify;"><a href="https://adopt-openjdk.ci.cloudbees.com/" target="_blank">Adopt OpenJDK CloudBees Build Farm</a> - thanks to the folks at <a href="https://www.cloudbees.com/" target="_blank">CloudBees</a> for helping us host our build farm on their CI/CD servers. This one was initially started by <a href="https://twitter.com/karianna" target="_blank">Martijn Verburg</a> and later with the help of a number of JUG members have come to the point that major Java projects are built against different versions of the JDK. These projects include building the JDKs themselves (versions 1.7, 1.8, 1.9, Jigsaw and Shenandoah). This project has also helped support the <em>Testing Java Early project</em> and <em>Quality  Outreach program</em>.</li>
</ul>
<p style="text-align: justify;">These are just a handful of such creations and contributions from the members of the community, some of these projects would certainly need help from you. As a community one more thing we could do well is celebrate our victories and successes, and especially credit those that have been involved whether as individuals or a community. So that our next generation contributors feel inspired and encourage to do more good work and share it with us.</p>

<h4><strong>Contributions from the community</strong></h4>
<a href="http://www.javaadvent.com/content/uploads/2015/12/contribution_header-700x325.png" rel="attachment wp-att-676"><img class="alignleft wp-image-676 size-medium" src="http://www.javaadvent.com/content/uploads/2015/12/contribution_header-700x325-300x139.png" alt="We want to contribute" width="300" height="139" /></a>
<p style="text-align: justify;">In a recent <a href="https://twitter.com/theNeomatrix369/status/672169033765142528" target="_blank">tweet</a> and posts to various Java / JVM and developer mailing lists, I requested the community to come forward and share their contribution stories or those from others with our community. The purpose was two-fold, one to share it with the community and the other to write this post (which in turn is shared with the community). I was happy to see a handful of messages sent to me and the mailing lists by a number of community members. I'll share some of these with you (in the order I have received them).</p>

<div class="fX"><strong>Sebastian Daschner:</strong></div>
<blockquote>
<p style="text-align: justify;">I don't know if that counts as contribution but I've hacked on the
OpenJDK compiler for fun several times. For example I added a new
thought up 'maybe' keyword which produces randomly executed code:
<a href="https://blog.sebastian-daschner.com/entries/maybe_keyword_in_java" target="_blank" rel="noreferrer">https://blog.sebastian-daschne<wbr />r.com/entries/maybe_keyword_<wbr />in_java</a></p>
</blockquote>
<strong>Thomas Modeneis:</strong>
<blockquote>
<div style="text-align: justify;">Thanks for writing, I like your initiative, its really good to show how people are doing and what they have been focusing on. Great idea.</div>
<div style="text-align: justify;"></div>
<div style="text-align: justify;">From my part, I can tell about the DevoxxMA last month, I did a talk on the Hacker Space about the Adopt the OpenJDK and it was really great. We had about 30 or more attendees, it was in a open space so everyone that was going to any talk was passing and being grabbed to have a look about the topic, it was really challenging because I had no mic. but I managed to speak out loud and be listen, and I got great feedback after the session. I'm going to work over the weekend to upload the presentation and the recorded video and I will be posting here as soon as I have it done! :)</div></blockquote>
<strong>Martijn Verburg:</strong>
<blockquote>
<p style="text-align: justify;">Good initiative.  So the major items I participated in were Date and Time and Lambdas Hackdays (reporting several bugs), submitted some warnings cleanups for OpenJDK.  Gave ~10 pages of feedback for jshell and generally tried to encourage people more capable than me to contribute :-).</p>
</blockquote>
<strong>Andrii Rodionov:</strong>
<blockquote>
<p style="text-align: justify;">Olena Syrota and Oleg Tsal-Tsalko from Ukraine JUG: Contributing to JSR 367 test code-base (<a href="https://github.com/olegts/jsonb-spec" target="_blank">https://github.com/olegts/<wbr />jsonb-spec</a>), promoting ‘Adopt a JSR’ and JSON-B spec at JUG UA meetings (<a href="http://jug.ua/2015/04/json-binding/" target="_blank">http://jug.ua/2015/04/json-<wbr />binding/</a>) and also at JavaDay Lviv conference (<a href="http://www.slideshare.net/olegtsaltsalko9/jsonb-spec" target="_blank">http://www.slideshare.net/<wbr />olegtsaltsalko9/jsonb-spec</a>).</p>
</blockquote>
<h4><strong>Contributors</strong></h4>
<a href="http://www.javaadvent.com/content/uploads/2015/12/contributor.png" rel="attachment wp-att-677"><img class="alignleft wp-image-677" src="http://www.javaadvent.com/content/uploads/2015/12/contributor-300x245.png" alt="Contributors gathering together" width="234" height="191" /></a>
<p style="text-align: justify;">As you have seen that from out of a community of 9+ million users, only a handful of them came forward to share their stories. While I can point you out to another list of contributors who have been paramount with their contributions to the <a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/" target="_blank">Adopt OpenJDK GitBook</a>, for example, take a look at the <a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/contributors.html" target="_blank">list of contributors</a> and also the <a href="https://github.com/AdoptOpenJDK/adoptopenjdk-getting-started-kit/commits/master" target="_blank">committers on the git-repo</a>. They have not just contributed to the book but to Java and the OpenJDK community, especially those who have helped translate the book into multiple languages. And then there are a number of them who haven't come forward to add their names to the list, even though they have made valuable contributions.
<img class="wp-image-678 alignright" src="http://www.javaadvent.com/content/uploads/2015/12/superherocoupleK9shadowcopy-300x224.jpg" alt="Super heroes together" width="228" height="170" /></p>
<p style="text-align: justify;">From this I can say contributors can be like unsung heroes, either due their shy or low-profile nature or they just don't get noticed by us. So it would only be fair to encourage them to come forward or share with the community about their contributions, however simple or small those may be. In addition to the above list I would like to also add a number of them (again apologies if I have missed out your name or not mentioned about you or all your contributions). These names are in <span style="text-decoration: underline;">no particular order</span> but as they come to my mind as their contributions have been invaluable:</p>

<ul>
	<li style="text-align: justify;">Dalibor Topic (OpenJDK Project Lead) &amp; the OpenJDK team</li>
	<li style="text-align: justify;">Mario Torre &amp; the RedHat OpenJDK team</li>
	<li style="text-align: justify;">Tori Wieldt (Java Community manager) and her team</li>
	<li style="text-align: justify;">Heather Vancura &amp; the JCP team</li>
	<li style="text-align: justify;">NightHacking, vJUG and RebelLabs (and the great people behind them)</li>
	<li style="text-align: justify;">Nicolaas &amp; the team at Cloudbees</li>
	<li style="text-align: justify;">Chris Newland (JitWatch developer)</li>
	<li>Lucy Carey, Ellie &amp; Mark Hazell (Devoxx UK &amp; Voxxed)</li>
	<li style="text-align: justify;">Richard Kolb (JUG South Africa)</li>
	<li style="text-align: justify;">Daniel Bryant, Richard Warburton, Ben Evans, and a number of others from LJC</li>
	<li style="text-align: justify;">Members of SouJava (Otavio, Thomas, Bruno, and others)</li>
	<li style="text-align: justify;">Members of Bulgarian JUG (Ivan, Martin, Mitri) and neighbours</li>
	<li style="text-align: justify;">Oti, Ludovic &amp; Patrick Reinhart</li>
	<li style="text-align: justify;">and a number of other contributors who for some reason I can't remember...</li>
</ul>
<p style="text-align: justify;">I have named them for their contributions to the community by helping organise Hackdays during the week and weekends, workshops and hands-on sessions at conferences, giving lightening talks, speaking at conferences, allowing us to host our CI and build farm servers, travelling to different parts of the world holding the Java community flag, writing books, giving Java and advance-level training, giving feedback on new technologies and features, and innumerable other activities that support and push forward the Java / JVM platform.</p>

<h4><strong>How you can make a difference ? And why ?</strong></h4>
<a href="http://www.javaadvent.com/content/uploads/2015/12/make_a_difference.jpeg" rel="attachment wp-att-679"><img class="alignleft wp-image-679" src="http://www.javaadvent.com/content/uploads/2015/12/make_a_difference-300x205.jpeg" alt="Make a difference" width="225" height="154" /></a>
<p style="text-align: justify;">You can make a difference by doing something as simple as clicking the <em>like</em> button (on Twitter, LinkedIn, Facebook, etc...) or responding to a message on a mailing list by expressing your opinion about something you see or read about --as to why you think about it that way or how it could be different.</p>
<p style="text-align: justify;">The answer to the question "And why ?" is simple, because you are part of a community and 'you care' and want to share your knowledge and experience with others -- just like the others above who have spared free moments of their valuable time for us.</p>

<h4><strong>Is it hard to do it ? Where to start ? What needs most attention ?</strong></h4>
<p style="text-align: justify;"><a href="http://www.javaadvent.com/content/uploads/2015/12/important-checklist.jpg" rel="attachment wp-att-681"><img class="wp-image-681 alignleft" src="http://www.javaadvent.com/content/uploads/2015/12/important-checklist-300x278.jpg" alt="important-checklist" width="129" height="120" /></a> The answer is its not hard to do it, if so many have done it, you can do it as well. Where to start and what can you do ? I have written <a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/how-to-navigate/how_to_contribute_to_adopt_openjdk_and_openjdk.html" target="_blank">a page on this topic</a>. And its worth reading it before going any further.</p>
<p style="text-align: justify;">There is a <a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/whatsChanged.html" target="_blank">dynamic list of topics</a> that is worth considering when thinking of contributing to OpenJDK and Java. But recently I have filtered this list down to a few topics (in order of precedence):</p>

<ul>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/intermediate-steps/testing_java_early_project.html">Testing Java Early project</a> (Priority one)</li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/adoptopenjdk-projects/g1gc_feedback.html">G1GC feedback</a></li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/openjdk-projects/jigsaw/jigsaw.html">Project Jigsaw</a></li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/adoptopenjdk-projects/unified_jvm_logging.html">Unified JVM logging</a></li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/openjdk-projects/valhalla.html">Project Valhalla</a></li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/openjdk-projects/kulla/kulla.html">Project Kulla</a></li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/openjdk-projects/shenandoah.html">Project Shenandoah</a></li>
	<li><a href="https://adoptopenjdk.gitbooks.io/adoptopenjdk-getting-started-kit/content/en/openjdk-projects/penrose.html">Project Penrose</a></li>
</ul>
<h4><strong>We need you!</strong></h4>
With that I would like to close by saying:

<a href="https://java.net/projects/adoptopenjdk/pages/AdoptOpenJDK#Getting_Started" target="_blank" rel="attachment wp-att-686"><img class="alignnone size-medium wp-image-686" src="http://www.javaadvent.com/content/uploads/2015/12/i_need_you_duke3-300x198.gif" alt="i_need_you_duke3" width="300" height="198" /></a>

Not just <strong>"I", </strong>but <strong>we</strong> as a community need <em><span style="text-decoration: underline;">you</span></em>.
<p style="text-align: justify;"><em>This post is part of the <a href="http://javaadvent.com/">Java Advent Calendar</a> and is licensed under the <a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a> license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em></p>