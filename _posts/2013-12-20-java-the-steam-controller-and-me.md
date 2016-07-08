---
id: 40
title: Java, the Steam controller and me
date: 2013-12-20T08:00:00+00:00
author: gpanther
layout: post
guid: http://www.javaadvent.com/2013/12/java-the-steam-controller-and-me/
permalink: /2013/12/java-the-steam-controller-and-me.html
blogger_blog:
  - www.javaadvent.com
blogger_author:
  - Xafero
blogger_permalink:
  - /2013/12/java-steam-controller-and-me.html
blogger_internal:
  - /feeds/2481158163384033132/posts/default/7550432491162557680
categories:
  - 2013
  - Gaming
  - java
  - JInput
---
Do you ever wondered if you could use existing stuff for something new? I saw some footage of the so-called 'Steam controller' (SC from now on) and looked at my game pad. Asking myself if it would be possible to use it in a steamy-like way, I've found some Java libraries and created a project that I'd like to share with you today.<br /><br /><div style="clear: both; text-align: center;"><a href="http://media.steampowered.com/steam/store/livingroom/controller/SteamController.jpg" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img border="0" height="231" src="http://media.steampowered.com/steam/store/livingroom/controller/SteamController.jpg" width="320" /></a></div>Of course, there have been a lot of input devices (and especially game controllers) long before the SC's release, but it has one new property which makes it special.<br /><br />It has two touchpads, which can emulate a mouse's or keyboard's input in order to be able to play (practically nearly) every game. As some early videos show, even a mouse-intensive game like the puzzle game 'Portal' seems to be playable by using this compatibility mode.<br /><br /><div style="clear: both; text-align: center;"><a href="http://cloud-2.steampowered.com/ugc/902132260878600701/5B58A127F0AB21E70C208FC877CB3A7BF7E77190/" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em;"><img border="0" height="240" src="http://cloud-2.steampowered.com/ugc/902132260878600701/5B58A127F0AB21E70C208FC877CB3A7BF7E77190/" width="320" /></a></div>As a gamer&nbsp;enthusiast and Java programmer, what could I do with something like this (a XBOX controller which I already got) in order to come close to that?<br /><br />A small tool named 'StrangeCtrl' saw the world's bright light. Talking to the controller needs some JNI (because there is no USB subsystem in the JVM for example), but the rest is written in pure Java. It sits in the system tray and is configured manually per configuration file, although one could build a GUI, too.<br /><br />Its dependencies are 'net.java.jinput.JInput' in version 2.0.5 (still working for Windows 8.1) and a little helper I wrote ('com.xafero.SuperLoader' v0.1). Now I'll explain the steps taken on the way.<br /><br /><b>First step: How do we get Java to talk to my controller?</b><br /><br />Luckily, the BSD-licensed JInput project does exactly this. It connects to Microsoft's XInput interface for example and fills some Java data structures with the native data it gets. Linux and Mac OS X are covered, too, don't worry.<br /><br />So I plugged in my game pad (one XBOX-compatible controller) and the way seemed to be clear:<br /><ol><li>get the controllers</li><li>get their input events&nbsp;</li><li>and convert them to virtual events for keyboard and mouse.&nbsp;</li></ol>The library's native components for the big three OS are delivered in Java archives (at least per Maven). But as you might already know,&nbsp;java.lang.System only loads files directly available on the file system.<br /><br /><b>Second step: So how do we get around this annoying limitation?</b><br /><b><br /></b>After a quick search, I've found wcmatthysen's 'mx-native-loader' which seemed useful as it claims to extract JAR's and load the native stuff. But it didn't work, because the libraries of JInput are packed into several 'jinput-platform-***.jar' files instead of one big chunk under META-INF/lib as this loader suggests.<br /><br />So the new helper library called 'SuperLoader' works around these circumstances:<br /><ol><li>Create a temporary directory for all nasty native libraries, e.g. with the help of the system property 'java.io.tmpdir'. It could also be specified by the user directly as it doesn't really matter where it is.</li><li>Get all nasty libraries out of the JARs already loaded; iterate over all class path's URLs and extract them or exclude most of them by using a filter.</li><li>Extend the existing library path; one thing the other library didn't do and it's very annoying to do it manually, so the system property 'java.library.path' should be extended.</li><li>Force the JVM to renew the system paths; one can do it by resetting the field 'sys_paths' of the system class loader to null. This forces the System class to really appreciate the new circumstances the next time you request a library.</li></ol><div>Now the application pre-loads all native libraries into a temporary folder and when JInput is asked to give a list of the controllers, for example, it doesn't have to be changed for using JAR files. It simply is able to use System.loadLibrary as anyone would do.</div><div><br /></div><div><b>Third step: What's possible to simulate?</b></div><div><br /></div><div>We've finally got to read the game pad's events, so what can we do with it? With AWT's Robot class, it's possible since the early Java days to simulate a key press or mouse movements and such. Although the robot requires one to specify the desktop it should work on, it works just as good on multi-monitor systems. The only difference is the offset of all events it generates - an aspect especially important if one wants to click on specific regions of the PC's screen.</div><div><br />The implemented commands so far are:<br /><ul><li>MouseMoveCmd - moves the mouse by some amount horizontally or vertically</li><li>MouseClickCmd - clicks the given mouse button at the current screen position</li><li>KeyComboCmd - presses some keys and releases them in the reversed order</li></ul>To allow some bit of extensibility, there is an interface which accepts the robot to generate virtual events, the current graphics device and the value given by JInput:<br /><blockquote>public interface ICommand {<br />&nbsp; &nbsp; void execute(Robot rbt, GraphicsDevice dev, float value);<br />}</blockquote>Its abstract implementation 'AbstractCmd' provides one constructor accepting one string. As a first step of processing, the raw string coming from the configuration file is splitted by an empty space into a string array.<br /><br /><b>Fourth step: Which configuration format can we use?</b><br /><br />There are a lot of trendy formats out there, like YAML, JSON, ... But Java already provides us with a simple way to achieve this. So the configuration file is parsed with the XML variant of the Java properties mechanism. To build the actual map out of strings connected with their commands, the class 'com.xafero.strangectrl.cmd.ConfigUtils'<br /><ul><li>loads the configuration,</li><li>iterates through all entries,</li><ul><li>searches a command by each entry's value,</li><li>loads each command by instantiating it with the textual arguments,</li><li>puts the result of key (controller button) and value (associated command) into a new map,</li></ul><li>and produces the actual map used to transform incoming events.</li></ul><div><b>Fifth step: The actual work...</b></div><div><br /></div><div>The helper class 'ControllerPoller' is a periodically executed TimerTask who is responsible for collecting new JInput events from an arbitrary amount of controllers and inform the caller about every new stuff:</div><blockquote><span style="white-space: pre;"> </span>public void run() {<br /><span style="white-space: pre;">  </span>for (Controller controller : controllers) {<br /><span style="white-space: pre;">   </span>if (!controller.poll())&nbsp;continue;<br /><span style="white-space: pre;">   </span>EventQueue queue = controller.getEventQueue();<br /><span style="white-space: pre;">   </span>Event event = new Event();<br /><span style="white-space: pre;">   </span>while (queue.getNextEvent(event))<br /><span style="white-space: pre;">    </span>callback.onNewEvent(this, controller, event);<br /><span style="white-space: pre;">  </span>}<br /><span style="white-space: pre;"> </span>}</blockquote>The caller (in this case the so-called 'App' living in the system tray) just implements the callback interface and gets all information for free whenever some input occurs:<br /><blockquote><span style="white-space: pre;"> </span>public static interface IControllerCallback {<br /><span style="white-space: pre;">  </span>void onNewEvent(ControllerPoller p, Controller c,&nbsp;Event e);<br /><span style="white-space: pre;"> </span>}</blockquote>Left to the 'App' is the search for associated commands to the incoming game pad's events and their execution with the correct parameters. Now we could use it for controlling some game, perhaps an old one like Prince of Persia or something otherwise unplayable with a game pad. But let's step aside...<br /><br /></div><div><div><b>Example aside from games: How to configure it for people with&nbsp;constrained movement?</b></div><div><br /></div><div>To show just another possible&nbsp;field of application, let's configure it for someone who can't press two keys at the same time. An example application should here be a web browser. In the configuration file, there are the following settings:</div><blockquote>&lt;!-- Button A means now left mouse click --&gt;<br />&lt;entry key="Button 0"&gt;mouseClick 1&lt;/entry&gt;<br />&lt;!-- Button B will open a new tab --&gt;<br />&lt;entry key="Button 1"&gt;keyCombo CONTROL T&lt;/entry&gt;<br />&lt;!-- Button X will close an existing tab --&gt;<br />&lt;entry key="Button 2"&gt;keyCombo CONTROL W&lt;/entry&gt;</blockquote><div>The browser in this example doesn't have to know the game controller, because the operating system will produce new virtual input events, and it will operate as demanded. By using Java and being FOSS, the tool is also customizable and easily understandable in every way (in contrast to some C/C++ code which would be otherwise necessary to emulate input devices).</div></div><div><br /></div><div><b>Resources and links</b></div><div><br />The source code is available at&nbsp;<a href="https://github.com/xafero/StrangeCtrl">https://github.com/xafero/StrangeCtrl</a>.<br />Feel free to use, share or modify any aspect (licensed under GPL v3).<br /><br />For further information see:</div><div><ul><li>JInput -&nbsp;<a href="https://java.net/projects/jinput">https://java.net/projects/jinput</a></li><li>AWT Robot -&nbsp;<a href="http://docs.oracle.com/javase/6/docs/api/java/awt/Robot.html">http://docs.oracle.com/javase/6/docs/api/java/awt/Robot.html</a></li></ul></div><em><br /></em><em>This post is part of the&nbsp;<a href="http://javaadvent.com/">Java Advent Calendar</a>&nbsp;and is licensed under the&nbsp;<a href="https://creativecommons.org/licenses/by/3.0/">Creative Commons 3.0 Attribution</a>&nbsp;license. If you like it, please spread the word by sharing, tweeting, FB, G+ and so on!</em>