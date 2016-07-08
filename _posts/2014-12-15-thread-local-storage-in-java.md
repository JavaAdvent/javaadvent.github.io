---
id: 18
title: Thread local storage in Java
date: 2014-12-15T09:00:00+00:00
author: Mite Mitreski
layout: post
permalink: /2014/12/thread-local-storage-in-java.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Mite Mitreski
blogger_permalink:
  - /2014/12/thread-local-storage-in-java.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/341817940530970968
image: /content/uploads/2014/12/thread-825x506.jpg
categories:
  - java
  - Java Advent
  - java advent 2014
  - local
  - Spring
  - thread
---
One of the rarely known features among developers is Thread-local storage. &nbsp;The idea is simple and need for it comes in &nbsp;scenarios where we need data that is ... well local for the thread. If we have two threads we that refer to the same global variable but we wanna them to have separate value independently initialized of each other.
<div>
<div></div>
<div>Most major programming languages have implementation of the concept. For example C++11 has even the <a href="http://en.cppreference.com/w/cpp/keyword/thread_local">thread_local</a> keyword, Ruby has chosen an API <a href="http://api.rubyonrails.org/classes/Thread.html#method-i-thread_variable_set">approach</a>&nbsp;.
<div></div>
<div>Java has also an implementation of the concept with <a href="http://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html">&nbsp;java.lang.ThreadLocal&lt;T&gt;</a> and its subclass&nbsp;<a href="http://docs.oracle.com/javase/8/docs/api/java/lang/InheritableThreadLocal.html">java.lang.InheritableThreadLocal&lt;T&gt;</a> since version 1.2, so nothing new and shiny here.</div>
<div></div>
<div>Let's say that for some reason we need to have an Long specific for our thread. Using Thread local that would simple be</div>
<div>
<pre><code>
public class ThreadLocalExample {

  public static class SomethingToRun implements Runnable {

    private ThreadLocal threadLocal = new ThreadLocal();

    @Override
    public void run() {
      System.out.println(Thread.currentThread().getName() + " " + threadLocal.get());

      try {
        Thread.sleep(2000);
      } catch (InterruptedException e) {
      }

      threadLocal.set(System.nanoTime());
      System.out.println(Thread.currentThread().getName() + " " + threadLocal.get());
    }
  }


  public static void main(String[] args) {
    SomethingToRun sharedRunnableInstance = new SomethingToRun();

    Thread thread1 = new Thread(sharedRunnableInstance);
    Thread thread2 = new Thread(sharedRunnableInstance);

    thread1.start();
    thread2.start();
  }

}
</code></pre>
</div>
<div>One possible sample run of the following code will result into :
<pre><code>
Thread-0 null

Thread-0 132466384576241

Thread-1 null

Thread-1 132466394296347
</code></pre>
</div>
<div>At the beginning the value is set to null to both threads, obviously each of them works with separate values since after setting the value to <i>System.nanoTime()</i> on <i>Thread-0</i> it will not have any effect on the value of <i>Thread-1 </i>exactly as we wanted, a thread scoped long variable.

One nice side effect is a case where the thread calls multiple methods from various classes. They will all be able to use the same thread scoped variable without major API changes. Since the value is not explicitly passed through one might argue it difficult to test and bad for design, but that is a separate topic altogether.
<h4>In what areas are popular frameworks using Thread Locals?</h4>
Spring being one of the most popular frameworks in Java uses ThreadLocals internally for many parts, easily shown by a simple github <a href="https://github.com/spring-projects/spring-framework/search?p=2&amp;q=threadLocal&amp;utf8=%E2%9C%93">search</a>. Most of the usages are related to the current's user's actions or information. This is actually one of the main uses for ThreadLocals in JavaEE world, storing information for the current request like in&nbsp;<a href="http://requestcontextholder/">RequestContextHolder</a> :
<pre><code>
private static final ThreadLocal requestAttributesHolder = 
    new NamedThreadLocal&lt;RequestAttributes&gt;("Request attributes");
</code></pre>
</div>
<div></div>
<div>Or the current JDBC connection user credentials in&nbsp;<a href="https://github.com/spring-projects/spring-framework/blob/dd2bf28a4f2c20cc6510266f245c619755e851ba/spring-jdbc/src/main/java/org/springframework/jdbc/datasource/UserCredentialsDataSourceAdapter.java">UserCredentialsDataSourceAdapter</a>.

If we get back on RequestContextHolder we can use this class to access all of the current request information for anywhere in our code.
Common use case for this is &nbsp;<a href="https://github.com/spring-projects/spring-framework/blob/dd2bf28a4f2c20cc6510266f245c619755e851ba/spring-context/src/main/java/org/springframework/context/i18n/LocaleContextHolder.java">LocaleContextHolder</a> that helps us store the current user's locale.
Mockito uses it to store the current "global" <a href="https://github.com/mockito/mockito/blob/18dc62d72821fdc54f53ba4a5ca98412ac1f4441/src/org/mockito/internal/configuration/GlobalConfiguration.java">configuration</a>&nbsp;and if we take a look at any framework out there there is a high chance we'll find it as well.

</div>
<div>
<h4>Thread Locals and Memory Leaks</h4>
We learned this awesome little feature so let's use it all over the place. We can do that but few google searches and we can find out that most out there say ThreadLocal is evil. That's not exactly true, it is a nice utility but in some contexts it might be easy to create a memory leak.
<blockquote>“Can you cause unintended object retention with thread locals? Sure you can. But you can do this with arrays too. That doesn’t mean that thread locals (or arrays) are bad things. Merely that you have to use them with some care. The use of thread pools demands extreme care. Sloppy use of thread pools in combination with sloppy use of thread locals can cause unintended object retention, as has been noted in many places. But placing the blame on thread locals is unwarranted.” - Joshua Bloch</blockquote>
</div>
<div>It is very easy to create a memory leak in your server code using&nbsp;<i>ThreadLocal</i>&nbsp;if it runs on an application&nbsp;server.&nbsp;<i>ThreadLocal</i>&nbsp;context is associated to the thread where it runs, and will be garbaged once the&nbsp;thread is dead. Modern app servers use pool of threads instead of creating new ones on each request meaning you can end up holding large objects indefinitely in your application. &nbsp;Since the thread pool is from the app server our memory leak could remain even after we unload our application. The fix for this is simple, free up resources you do not need.

One other <i>ThreadLocal</i> misuse is API design. Often I have seen use of&nbsp;<i>RequestContextHolder</i>(that holds&nbsp;<i>ThreadLocal</i>) all over the place, like the DAO layer for example. Later on if one were to call the same DAO methods outside a request like and scheduler for example he would get a very bad surprise.
This create black magic and many maintenance developers who will eventually figure out where you live and pay you a visit. Even though the variables in ThreadLocal are local to the thread they are very much global in your code. Make sure you really need this thread scope before you use it.
<h4>More info on the topic</h4>
<a href="http://en.wikipedia.org/wiki/Thread-local_storage">http://en.wikipedia.org/wiki/Thread-local_storage</a>
<a href="http://www.appneta.com/blog/introduction-to-javas-threadlocal-storage/">http://www.appneta.com/blog/introduction-to-javas-threadlocal-storage/</a>
<a href="https://plumbr.eu/blog/how-to-shoot-yourself-in-foot-with-threadlocals">https://plumbr.eu/blog/how-to-shoot-yourself-in-foot-with-threadlocals</a>
<a href="http://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable">http://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable</a>
<a href="https://plumbr.eu/blog/when-and-how-to-use-a-threadlocal">https://plumbr.eu/blog/when-and-how-to-use-a-threadlocal</a>
<a href="https://weblogs.java.net/blog/jjviana/archive/2010/06/09/dealing-glassfish-301-memory-leak-or-threadlocal-thread-pool-bad-ide">https://weblogs.java.net/blog/jjviana/archive/2010/06/09/dealing-glassfish-301-memory-leak-or-threadlocal-thread-pool-bad-ide</a>
<a href="https://software.intel.com/en-us/articles/use-thread-local-storage-to-reduce-synchronization">https://software.intel.com/en-us/articles/use-thread-local-storage-to-reduce-synchronization</a>






</div>
</div>
</div>
<em>This post is part of the&nbsp;<a href="http://javaadvent.com/">Java Advent Calendar</a>&nbsp;and is licensed under the&nbsp;<a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a>&nbsp;license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em>