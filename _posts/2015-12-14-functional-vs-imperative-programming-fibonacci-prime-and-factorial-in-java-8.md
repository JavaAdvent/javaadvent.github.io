---
id: 435
title: Functional vs Imperative Programming. Fibonacci, Prime and Factorial in Java 8
date: 2015-12-14T01:00:31+00:00
author: Artur Mkrtchyan
layout: post
guid: http://www.javaadvent.com/?p=435
permalink: /2015/12/functional-vs-imperative-programming-fibonacci-prime-and-factorial-in-java-8.html
categories:
  - 2015
  - java
  - Java Advent
  - JVM
tags:
  - functional programming
  - Java 8
  - lambda
  - streams
---
There are multiple programming styles/paradigms, but two well-known ones are *Imperative* and *Functional*. 

*Imperative* programming is the most dominant paradigm as nearly all mainstream languages (C++, Java, C#) have been promoting it. But in the last few years functional programming started to gain attention. One of the main driving factors is that simply all new computers are shipped with 4, 8, 16 or more cores and it's very difficult to write a parallel program in imperative style to utilize all cores. Functional style moves this difficultness to the runtime level and frees developers from hard and error-prone work.

Wait! So what's the difference between these two styles.

>**Imperative programming** is a paradigm where you tell how exactly and which exact statements machine/runtime should execute to achieve desired result.

>**Functional programming** is a form of declarative programming paradigm where you tell what you would like to achieve and machine/runtime determines the best way how to do it.

Functional style moves the **how** part to the runtime level and helps developers focus on the **what** part. By abstracting the *how* part we can write more *maintainable and scalable* software. 

To handle the challenges introduced by multicore machines and to remain attractive for developers **Java 8** introduced functional paradigm next to imperative one.

Enough theory, let's implement few programming challenges in Imperative and Functional style using Java and see the difference.

➤**Fibonacci Sequence Imperative vs Functional** (The Fibonacci Sequence is the series of numbers:  1, 1, 2, 3, 5, 8, 13, 21, 34, ... The next number is found by adding up the two numbers before it.)

**Fibonacci Sequence in iterative and imperative style**

    public static int fibonacci(int number) {
      int fib1 = 1;
      int fib2 = 1;
      int fibonacci = fib1;
      for (int i = 2; i < number; i++) {
        fibonacci = fib1 + fib2;
        fib1 = fib2;
        fib2 = fibonacci;
      }
      return fibonacci;
    }
    
    for(int i = 1; i  <= 10; i++) {
      System.out.print(fibonacci(i) +" ");
    }
    // Output: 1 1 2 3 5 8 13 21 34 55 


As you can see here we are focusing a lot on *how* (iteration, state) rather that *what* we want to achieve.  

**Fibonacci Sequence in iterative and functional style**

    IntStream fibonacciStream = Stream.iterate(
        new int[]{1, 1},
        fib -> new int[] {fib[1], fib[0] + fib[1]}
      ).mapToInt(fib -> fib[0]);

    fibonacciStream.limit(10).forEach(fib ->  
        System.out.print(fib + " "));
    // Output: 1 1 2 3 5 8 13 21 34 55 

In contrast, you can see here we are focusing on *what* we want to achieve.

➤**Prime Numbers Imperative vs Functional** (A prime number is a natural number greater than 1 that has no positive divisors other than 1 and itself.)

**Prime Number in imperative style**

    public boolean isPrime(long number) {  
      for(long i = 2; i <= Math.sqrt(number); i++) {  
        if(number % i == 0) return false;  
      }  
      return number > 1;  
    }
    isPrime(9220000000000000039L) // Output: true

Again here we are focusing a lot on *how* (iteration, state).

**Prime Number in functional style**

    public boolean isPrime(long number) {  
      return number > 1 &&  
        LongStream
         .rangeClosed(2, (long) Math.sqrt(number))  
         .noneMatch(index -> number % index == 0);
    }
    isPrime(9220000000000000039L) // Output: true

Here again we are focusing on *what* we want to achieve. The functional style helped us to abstract away the process of explicitly iterating over the range of numbers. 

You might now think, hmmm, is this all we can have ....  ? Let's see how can we use all our cores (gain parallelism) in functional style.


    public boolean isPrime(long number) {  
      return number > 1 &&  
        LongStream
        .rangeClosed(2, (long) Math.sqrt(number))
        .parallel()  
        .noneMatch(index -> number % index == 0);
    }
    isPrime(9220000000000000039L) // Output: true

That's it! We just added **.parallel()** to the stream. You can see how library/runtime handles complexity for us.


➤**Factorial Imperative vs Functional** ( The factorial of n is the product of all positive integers less than or equal to n.)

**Factorial in iterative and imperative style**

    public long factorial(int n) {
      long product = 1;
      for ( int i = 1; i <= n; i++ ) {
        product *= i;
      }
      return product;
    }
    factorial(5) // Output: 120

**Factorial in iterative and functional style**

    public long factorial(int n) {
     return LongStream
       .rangeClosed(1, n)
       .reduce((a, b) -> a *   b)
       .getAsLong();
    }
    factorial(5) // Output: 120

It's worth repeating that by abstracting the *how* part we can write more *maintainable and scalable* software.  

To see all the functional goodies introduced by Java 8 check out the following [Lambda Expressions, Method References and Streams](http://docs.oracle.com/javase/8/docs/technotes/guides/language/enhancements.html#javase8) guide.<!--more-->