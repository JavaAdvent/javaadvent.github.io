---
id: 44
title: ArrayList vs LinkedList
date: 2013-12-16T08:10:00+00:00
author: Jean-Philippe BEMPEL
layout: post
guid: http://www.javaadvent.com/2013/12/arraylist-vs-linkedlist/
permalink: /2013/12/arraylist-vs-linkedlist.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Jean-Philippe BEMPEL
blogger_permalink:
  - /2013/12/arraylist-vs-linkedlist.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/7838514884810534013
dsq_thread_id:
  - 4962579287
categories:
  - 2013
  - ArrayList
  - cpu caches
  - hardware counters
  - LinkedList
  - overseer
---
I must confess the title of this post is a little bit catchy. I have recently read this <a href="http://highscalability.com/blog/2013/5/22/strategy-stop-using-linked-lists.html">blog post</a> and this is a good summary of&nbsp; discussions &amp; debates about this subject.<br />But this time I would like to try a different approach to compare those 2 well known data structures: using <a href="http://jpbempel.blogspot.com/2013/08/hardware-performance-counters.html">Hardware Performance Counters</a>. <br /><br />I will not perform a micro-benchmark, well not directly. I will not time using System.nanoTime(), but rather using HPCs like cache hits/misses.<br /><br />No need to present those data structures, everybody knows what they are using for and how they are implemented. I am focusing my study on list iteration because, beside adding an element, this is the most common task for a list. And also because the memory access pattern for a list is a good example of CPU cache interaction.<br /><br /><br />Here my code for measuring list iteration for LinkedList &amp; ArrayList:<br /><br /><pre style="border: solid thin gray;"><code><br />import java.util.ArrayList;<br />import java.util.LinkedList;<br />import java.util.List;<br /><br />import ch.usi.overseer.OverHpc;<br /><br />public class ListIteration<br />{<br />    private static List&lt;String&gt; arrayList = new ArrayList&lt;&gt;();<br />    private static List&lt;String&gt; linkedList = new LinkedList&lt;&gt;();<br /><br />    public static void initializeList(List&lt;String&gt; list, int bufferSize)<br />    {<br />        for (int i = 0; i &lt; 50000; i++)<br />        {<br />            byte[] buffer = null;<br />            if (bufferSize &gt; 0)<br />            {<br />                buffer = new byte[bufferSize];<br />            }<br />            String s = String.valueOf(i);<br />            list.add(s);<br />            // avoid buffer to be optimized away<br />            if (System.currentTimeMillis() == 0)<br />            {<br />                System.out.println(buffer);<br />            }<br />        }<br />    }<br /><br />    public static void bench(List&lt;String&gt; list)<br />    {<br />        if (list.contains("bar"))<br />        {<br />            System.out.println("bar found");<br />        }<br />    }<br /><br />    public static void main(String[] args) throws Exception<br />    {<br />        if (args.length != 2) return;<br />        List&lt;String&gt; benchList = "array".equals(args[0]) ? arrayList : linkedList;<br />        int bufferSize = Integer.parseInt(args[1]);<br />        initializeList(benchList, bufferSize);<br />        HWCounters.init();<br />        System.out.println("init done");<br />        // warmup<br />        for (int i = 0; i &lt; 10000; i++)<br />        {<br />            bench(benchList);<br />        }<br />        Thread.sleep(1000);<br />        System.out.println("warmup done");<br /><br />        HWCounters.start();<br />        for (int i = 0; i &lt; 1000; i++)<br />        {<br />            bench(benchList);<br />        }<br />        HWCounters.stop();<br />        HWCounters.printResults();<br />        HWCounters.shutdown();<br />    }<br />}<br /></code></pre><br />To measure, I am using a class called HWCounters based on <a href="http://www.peternier.com/projects/overseer/overseer.php">overseer library</a> to get Hardware Performance Counters. You can find the code of this class <a href="https://github.com/jpbempel/snippets/blob/master/HPC/HWCounters.java">here</a>.<br /><br />The program take 2 parameters: the first one to choose between ArrayList implementation or LinkedList one, the second one to take a buffer size used in <span style="font-family: Courier New, Courier, monospace;">initializeList</span> method. This method fills a list implementation with 50K strings. Each string is newly created just before add it to the list. We may also allocate a buffer based on our second parameter of the program. if 0, no buffer is allocated.<br /><span style="font-family: Courier New, Courier, monospace;">bench</span> method performs a search of a constant string which is not contained into the list, so we fully traverse the list.<br />Finally, <span style="font-family: Courier New, Courier, monospace;">main </span>method, perform initialization of the list, warmups the bench method and measure 1000 runs of this method. Then, we print results from HPCs.<br /><br />Let's run our program with no buffer allocation on Linux with 2 Xeon X5680:<br /><br /><pre>[root@archi-srv]# java -cp .:overseer.jar com.ullink.perf.myths.ListIteration&nbsp;array 0<br />init done<br />warmup done<br />Cycles: 428,711,720<br />Instructions: 776,215,597<br />L2 hits: 5,302,792<br />L2 misses: 23,702,079<br />LLC hits: 42,933,789<br />LLC misses: 73<br />CPU migrations: 0<br />Local DRAM: 0<br />Remote DRAM: 0</pre><br /><pre>[root@archi-srv]# /java -cp .:overseer.jar com.ullink.perf.myths.ListIteration&nbsp;linked 0<br />init done<br />warmup done<br />Cycles: 767,019,336<br />Instructions: 874,081,196<br />L2 hits: 61,489,499<br />L2 misses: 2,499,227<br />LLC hits: 3,788,468<br />LLC misses: 0<br />CPU migrations: 0<br />Local DRAM: 0<br />Remote DRAM: 0<br /></pre><br />First run is on the ArrayList implementation, second with LinkedList.<br /><br /><ul><li>Number of cycles is the number of CPU cycle spent on executing our code. Clearly LinkedList spent much more cycles than ArrayList.</li><li>Instructions is little higher for LinkedList. But it is not so significant here.</li><li>For L2 cache accesses we have a clear difference: ArrayList has significant more L2 misses compared to LinkedList.&nbsp;&nbsp;</li><li>Mechanically, LLC hits are very important for ArrayList.</li></ul><br />The conclusion on this comparison is that most of the data accessed during list iteration is located into L2 for LinkedList but into L3 for ArrayList.<br />My explanation for this is that strings added to the list are created right before. For LinkedList it means that it is local the Node entry that is created when adding the element. We have more locality with the node.<br /><br />But let's re-run the comparison with intermediary buffer allocated for each new String added.<br /><br /><br /><pre>[root@archi-srv]# java -cp .:overseer.jar com.ullink.perf.myths.ListIteration array 256<br />init done<br />warmup done<br />Cycles: 584,965,201<br />Instructions: 774,373,285<br />L2 hits: 952,193<br />L2 misses: 62,840,804<br />LLC hits: 63,126,049<br />LLC misses: 4,416<br />CPU migrations: 0<br />Local DRAM: 824<br />Remote DRAM: 0<br /></pre><br /><pre>[root@archi-srv]# java -cp .:overseer.jar com.ullink.perf.myths.ListIteration linked 256<br />init done<br />warmup done<br />Cycles: 5,289,317,879<br />Instructions: 874,350,022<br />L2 hits: 1,487,037<br />L2 misses: 75,500,984<br />LLC hits: 81,881,688<br />LLC misses: 5,826,435<br />CPU migrations: 0<br />Local DRAM: 1,645,436<br />Remote DRAM: 1,042<br /></pre><br />Here the results are quite different:<br /><br /><ul><li>Cycles are 10 times more important.</li><li>Instructions stay the same as previously</li><li>For cache accesses, ArrayList have more L2 misses/LLC hits, than previous run, but still in the same magnitude order. LinkedList on the contrary have a lot more L2 misses/LLC hits, but moreover a significant number of LLC misses/DRAM accesses. And the difference is here.</li></ul><br />With the intermediary buffer, we are pushing away entries and strings, which generate more cache misses and the end also DRAM accesses which is much more slower than hitting caches.<br />ArrayList is more predictable here since we keep locality of element from each other.<br /><br />The memory access pattern here is crucial for list iteration performance. ArrayList is more stable than LinkedList in the way that whatever you are doing between each element adding, you are keeping your data&nbsp; much more local than the LinkedList.<br />Remember also that, iterating through an array is much more efficient for CPU since it can trigger Hardware Prefetching because access pattern is very predictable.<br /><br /><br /><br />