---
id: 73
title: 'Cleaning Up After Yourself &#8211; Java Style'
date: 2012-12-16T08:00:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2012/12/cleaning-up-after-yourself-java-style/
permalink: /2012/12/cleaning-up-after-yourself-java-style.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - codejunkie
blogger_permalink:
  - /2012/12/cleaning-up-after-yourself-java-style.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/8104628906528229222
dsq_thread_id:
  - 4962579195
categories:
  - 2012
---
When it comes to developing applications, we as software developers always (almost always at least!) stive to make our code as robust and clean as possible. &nbsp;By robust I mean error conditions are handled gracefully and clean meaning that whatever resources we have opened up are closed appropriately. But let's face it, there will be times when mistakes are made or the unexpected happens. Fortunately, Java provides some great tools that allow for handling just such situations - shutdown hooks and uncaught exception handlers.<br /><br /><h2>Shutdown Hooks</h2><div>Shutdown hooks are Threads that are initialized, but not started. &nbsp;When the virtual machine begins to shutdown it will run all registered shutdown hooks, but not in any specified order. Registering a shutdown hook is trivial:</div><div><code>    Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {    @Override    public void run() {       doSomeCleanup();     }   })); </code></div>Now when a program exits, the code specified in your shutdown hook will run, giving you one last chance to ensure things are left in a consistent state.  There are some caveats with shutdown hooks: <ol><li>Don't assume other services will be available, code defensively</li><li>Make every attempt to make sure the code executes quickly</li><li>Make sure the code is written to avoid deadlocks</li>  </ol>Ideally, shutdown hooks are to ensure that resources are cleaned up properly and not a replacement for good coding practices. <br/><br/><h2>Uncaught Exception Handlers</h2>The Thread.UncaughtExceptionHandler represents a chance to do what the name of the interface implies, handle any uncaught exceptions.  As with the shutdown hook, the UncaughtExceptionHandler should not be used as an excuse to get away from good coding practices.  Rather, the uncaught exception handler is the last line of defense against the truly unexpected exceptions. Setting an UncaughtExceptionHander is equally as easy: <div><code>Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler(){    @Override    public void uncaughtException(Thread thread, Throwable throwable) {           handleTheUnexpectedError();    } }); </code></div>  Now with just four lines of code, you have given your program the ability to handle unexpected errors gracefully.  <br/>I hope the reader found these tips useful and have a great Christmas!  <p><em>Meta: this post is part of the <a href="http://javaadvent.com/">Java Advent Calendar</a> and is licensed under the <a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a> license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on! Want to write for the blog? We are looking for contributors to fill all 24 slot and would love to have your contribution! <a href="mailto:dify.ltd@gmail.com">Contact Attila Balazs</a> to contribute!</em></p>