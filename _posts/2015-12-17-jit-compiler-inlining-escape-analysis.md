---
id: 628
title: JIT Compiler, Inlining and Escape Analysis
date: 2015-12-17T01:00:55+00:00
author: Artur Mkrtchyan
layout: post
guid: http://www.javaadvent.com/?p=628
permalink: /2015/12/jit-compiler-inlining-escape-analysis.html
categories:
  - 2015
  - java
  - JVM
tags:
  - EscapeAnalysis
  - Inlining
  - JIT
  - JVM
---
### Just-in-time (JIT)
**Just-in-time (JIT)** compiler is the brain of the Java Virtual Machine. Nothing in the JVM affects performance more than the JIT compiler.

For a moment let's step back and see examples of compiled and non compiled languages.

Languages like Go, C and C++ are called *compiled languages* because their programs are distributed as binary (compiled) code, which is targeted to a particular CPU. 

On the other hand languages like PHP and Perl, are *interpreted*. The same program source code can be run on any CPU as long as the machine has the interpreter. The interpreter translates each line of the program into binary code as that line is executed.

Java attempts to find a middle ground here. Java applications are compiled, but instead of being compiled into a specific binary for a specific CPU, they are compiled into a  *bytecode*. This gives Java the platform independence of an interpreted language. But Java doesn't stop here.

In a typical program, only a small sections of the code is executed frequently, and the performance of an application depends primarily on how fast those sections of code are executed. These critical sections are known as the *hot spots* of the application.
The more times JVM executes a particular code section, the more information it has about it. This allows the JVM to make smart/optimized decisions and compile small hot code into a CPU specific binary. This process is called **Just in time compilation (JIT)**.

Now let's run a small program and observe JIT compilation.

    public class App {
      public static void main(String[] args) {
        long sumOfEvens = 0;
        for(int i = 0; i < 100000; i++) {
          if(isEven(i)) {
            sumOfEvens += i;
          }
        }
        System.out.println(sumOfEvens);
      }

      public static boolean isEven(int number) {
        return number % 2 == 0;
      }
    }
    

    #### Run
    javac App.java && \
    java -server \
         -XX:-TieredCompilation \
         -XX:+PrintCompilation \
                  - XX:CompileThreshold=100000 App


    #### Output
    87    1             App::isEven (16 bytes)
    2499950000

Output tells us that isEven method is compiled. I intentionally disabled *TieredCompilation* to get only the most frequently compiled code. 

**JIT compiled** code will give a great performance boost to your application. Want to check it ? Write a simple benchmark code.

### Inlining
**Inlining** is one of the most important optimizations that JIT compiler makes. Inlining replaces a method call with the body of the method to avoid the overhead of method invocation.

Let's run the same program again and this time observe inlining.

    #### Run
    javac App.java && \
    java -server \
         -XX:+UnlockDiagnosticVMOptions \
         -XX:+PrintInlining \
         -XX:-TieredCompilation App

    #### Output
    @ 12   App::isEven (16 bytes)   inline (hot)
    2499950000

**Inlining** again will give a great performance boost to your application.

### Escape Analysis
**Escape analysis** is a technique by which the JIT Compiler can analyze the scope of a new object's uses and decide whether to allocate it on the Java heap or (Wrong: on the method stack) [Update] handle object members directly (scalar replacement)[/Update]. It also eliminates locks for all non-globally escaping objects

Let's run a small program and observe garbage collection.

    public class App {
      public static void main(String[] args) {
        long sumOfArea = 0;
        for(int i = 0; i < 10000000; i++) {
          Rectangle rect = new Rectangle(i+5, i+10);
          sumOfArea += rect.getArea();
        }
        System.out.println(sumOfArea);
      }
      
      static class Rectangle {
        private int height;
        private int width;

        public Rectangle(int height, int width) {
          this.height = height;
          this.width = width;
        }

        public int getArea() {
          return height * width;
        }
      }
    }

In this example Rectangle objects are created and available only within a loop, they are characterised as NoEscape and can handle object members directly (scalar replacement) instead of allocating objects in heap. Specifically, this means that no garbage collection will happen.

Let's run the program without EscapeAnalysis.

    #### Run
    javac App.java && \
    java -server \
         -verbose:gc \
         -XX:-DoEscapeAnalysis App

    #### Output
    [GC (Allocation Failure)  65536K->472K(251392K), 0.0007449 secs]
    [GC (Allocation Failure)  66008K->440K(251392K), 0.0008727 secs]
    [GC (Allocation Failure)  65976K->424K(251392K), 0.0005484 secs]
    16818403770368

As you can see GC kicked-in. *Allocation Failure* means no more space is left in young generation to allocate objects. So, it is normal cause of young GC.

This time let's run it with EscapeAnalysis.

    #### Run
    javac App.java && \
    java -server \
        -verbose:gc \
        -XX:+DoEscapeAnalysis App

    #### Output
    16818403770368


No GC happened this time. Which basically means creating short lived and narrow scoped objects is not necessarily introducing garbage. 

*DoEscapeAnalysis* option is enabled by default. Note that only Java HotSpot Server VM supports this option.


As a consequence, we all should avoid premature optimization, focus on writing  more readable/maintainable code and let JVM do it's job.


