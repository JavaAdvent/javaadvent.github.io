---
id: 383
title: How to write a java agent
date: 2015-12-03T13:43:08+00:00
author: ThomasKrieger
layout: post
guid: http://www.javaadvent.com/?p=383
permalink: /2015/12/write-java-agent.html
dsq_thread_id:
  - 4962579371
categories:
  - 2015
  - java agent
---
For <a href="http://vmlens.com">vmlens</a>, a lightweight java race condition catcher, we are using a java agent to trace field accesses. Here are the lessons we learned implementing such an agent.
<h1>The Start</h1>
Create an agent class with a "static public static void premain(String args, Instrumentation inst)" method. Put this class into a jar file with a manifest pointing to the Agent class. The premain method will be called before the main method of the application.
<pre>
Manifest-Version: 1.0
Ant-Version: Apache Ant 1.9.2
Created-By: 1.8.0_05-b13 (Oracle Corporation)
Built-By: Thomas Krieger
Implementation-Vendor: Anarsoft
Implementation-Title: VMLens Agent
Implementation-Version: 2.0.0.201511181111
Can-Retransform-Classes: true
Premain-Class: com.anarsoft.trace.agent.Agent
Boot-Class-Path: agent_bootstrap.jar
</pre>
The MANIFEST.MF file from <a href="http://vmlens.com">vmlens</a>.



<h1>Class loader magic part 1</h1>
The agent class will be loaded by the system class loader. But we have to avoid version conflicts between the classes used by the agent and the application. Especially the frameworks used in the agent should not be visible to the application classes. So we use a dedicated URLClassLoader to load all other agent classes:
<pre>
// remember the currently used classloader
ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
		
// Create and set a special URLClassLoader
URLClassLoader classloader = new URLClassLoader(urlList.toArray(new URL[]{}) , null );
Thread.currentThread().setContextClassLoader(classloader);
	
// Load and execute the agent
String agentName = "com.anarsoft.trace.agent.runtime.AgentRuntimeImpl";
AgentRuntime agentRuntime  =  (AgentRuntime) classloader.loadClass(agentName).newInstance();
    
// reset the classloader
Thread.currentThread().setContextClassLoader(contextClassLoader);
</pre>

<h1>Class loader magic part 2</h1>
Now we use <a href="http://asm.ow2.org/">asm</a> to add our static callbacks methods when a field is accessed. To make sure that the classes are visible in every other class, they have to be loaded by the bootstrap classloader. To do this they have to be in a java package and the jar containing them have to be in the  boot class path.
<pre>
package java.anarsoft.trace.agent.bootstrap.callback;

public class FieldAccessCallback {

public static  void getStaticField(int field,int methodId) {
 }

}
</pre>
A callback class from <a href="http://vmlens.com">vmlens</a>. It has to be in the java package namespace to be visible in all classes.
<pre>
Boot-Class-Path: agent_bootstrap.jar
</pre>The boot class path entry in the MANIFEST.MF file from <a href="http://vmlens.com">vmlens</a>.



<a href="http://vmlens.com">VMLens</a>, a lightweight java race condition catcher, is built as a java agent. We know, writing java agents can be a tricky business. So, if you have any questions, just ask them in a comment below.