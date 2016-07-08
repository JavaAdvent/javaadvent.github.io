---
id: 767
title: Decorator Design Pattern using lambdas
date: 2015-12-24T17:53:43+00:00
author: Stefan Bulzan
layout: post
guid: http://www.javaadvent.com/?p=767
permalink: /2015/12/decorator-design-pattern-using-lambdas.html
categories:
  - design pattern
  - developers
  - lambda
  - Uncategorized
tags:
  - design pattern
  - lambda
  - method reference
---
With the advent of lambdas in Java we now have a new tool to better design our code. Of course the first step is using streams, method references and other neat features introduced in Java 8.

Going forward I think the next step is to revisit the well established Design Patterns and see them through the functional programming lenses. For this purpose I’ll take the Decorator Pattern and implement it using lambdas.

We’ll take an easy and delicious example of the Decorator Pattern: adding toppings to pizza. Here is the standard implementation as suggested by GoF:

First we have the interface that defines our component:
<pre style="font-size: 0.8em !important;"><code>public interface Pizza {
    String bakePizza();
}
</code></pre>
We have a concrete component:
<pre style="font-size: 0.8em !important;"><code>public class BasicPizza implements Pizza {
    @Override
    public String bakePizza() {
        return "Basic Pizza";
    }
}
</code></pre>
We decide that we have to decorate our component in different ways. We go with Decorator Pattern. This is the abstract decorator:
<pre style="font-size: 0.8em !important;"><code>public abstract class PizzaDecorator implements Pizza {
    private final Pizza pizza;
    
    protected PizzaDecorator(Pizza pizza) {
        this.pizza = pizza;
    }

    @Override
    public String bakePizza() {
        return pizza.bakePizza();
    }
}
</code></pre>
we provide some concrete decorators for the component:
<pre style="font-size: 0.8em !important;"><code>public class ChickenTikkaPizza extends PizzaDecorator {
    protected ChickenTikkaPizza(Pizza pizza) {
        super(pizza);
    }

    @Override
    public String bakePizza() {
        return super.bakePizza() + " with chicken topping";
    }
}

public class ProsciuttoPizza extends PizzaDecorator {

    protected ProsciuttoPizza(Pizza pizza) {
        super(pizza);
    }

    @Override
    public String bakePizza() {
        return super.bakePizza() + " with prosciutto";
    }
}
</code></pre>
and this is the way to use the new structure:
<pre style="font-size: 0.8em !important;"><code>Pizza pizza = new ChickenTikkaPizza(new BasicPizza());
String finishedPizza = pizza.bakePizza();   //Basic Pizza with chicken topping

pizza = new ChickenTikkaPizza(new ProsciuttoPizza(new BasicPizza()));
finishedPizza  = pizza.bakePizza();  //Basic Pizza with prosciutto with chicken topping
</code></pre>
we can see that this can get very messy, and it did get very messy if we think about how we handle buffered readers in java:
<pre style="font-size: 0.8em !important;"><code>new DataInputStream(new BufferedInputStream(new FileInputStream(new File("myfile.txt"))))
</code></pre>
of course, you can split that in multiple lines, but that won’t solve the messiness, it will just spread it.
Now lets see how we can do the same thing using lambdas.
We start with the same basic component objects:
<pre style="font-size: 0.8em !important;"><code>public interface Pizza {
    String bakePizza();
}

public class BasicPizza implements Pizza {
    @Override
    public String bakePizza() {
        return "Basic Pizza";
    }
}
</code></pre>
But now instead of declaring an abstract class that will provide the template for decorations, we will create the decorator that asks the user for functions that will decorate the component.
<pre style="font-size: 0.8em !important;"><code>public class PizzaDecorator {
    private final Function&lt;Pizza, Pizza&gt; toppings;

    private PizzaDecorator(Function&lt;Pizza, Pizza&gt;... desiredToppings) {
        this.toppings = Stream.of(desiredToppings)
                .reduce(Function.identity(), Function::andThen);

    }

    
    public static String bakePizza(Pizza pizza, Function&lt;Pizza, Pizza&gt;... desiredToppings) {
        return new PizzaDecorator(desiredToppings).bakePizza(pizza);
    }

private String bakePizza(Pizza pizza) {
    return this.toppings.apply(pizza).bakePizza();
}

}

</code></pre>
There is this line that constructs the chain of decorations to be applied:
<pre style="font-size: 0.8em !important;"><code>Stream.of(desiredToppings).reduce(identity(), Function::andThen);
</code></pre>
This line of code will take your decorations (which are of Function type) and chain them using andThen. This is the same as
<pre style="font-size: 0.8em !important;"><code>(currentToppings, nextTopping) -&gt; currentToppings.andThen(nextTopping)
</code></pre>
and it sure that the functions are called subsequently in the order you provided.
Also Function.identity() is translated to elem -&gt; elem lambda expression.

Ok, now where we’ll we define our decorations? You can add them as static methods in PizzaDecorator or even in the interface:
<pre style="font-size: 0.8em !important;"><code>public interface Pizza {
    String bakePizza();

    static Pizza withChickenTikka(Pizza pizza) {
        return new Pizza() {
            @Override
            public String bakePizza() {
                return pizza.bakePizza() + " with chicken";
            }
        };
    }

    static Pizza withProsciutto(Pizza pizza) {
        return new Pizza() {
            @Override
            public String bakePizza() {
                return pizza.bakePizza() + " with prosciutto";
            }
        };
    }
}

</code></pre>
And now, this is how this pattern gets to be used:
<pre style="font-size: 0.8em !important;"><code>String finishedPizza = PizzaDecorator.bakePizza(new BasicPizza(),Pizza::withChickenTikka, Pizza::withProsciutto);

//And if you static import PizzaDecorator.bakePizza:

String finishedPizza  = bakePizza(new BasicPizza(),Pizza::withChickenTikka, Pizza::withProsciutto);

</code></pre>
As you can see, the code got more clear and more concise, and we didn’t use inheritance to build our decorators.

This is just one of the many design pattern that can be improved using lambdas. There are more features that can be used to improve the rest of them like using partial application (currying) to implement Adapter Pattern.

I hope I got you thinking about adopting a more functional programming approach to your development style.

UPDATE: Here you can find a video walkthrough of this article, created by my friends from Webucator:

[embed]https://www.youtube.com/watch?v=lP_kGX56_eM[/embed]

If you want to see more of their tutorials you can visit <a href="https://www.webucator.com/java-training/javacore.cfm" target="_blank">Webucator site</a>

Bibliography:

The decorator example was inspired by <a href="https://dzone.com/articles/gang-four-%E2%80%93-decorate-decorator" target="_blank">Gang of Four – Decorate with Decorator Design Pattern</a> article

The refactoring method was inspired by the following Devoxx 2015 talks (which I recommend watching as they treat the subject at large):
<a href="https://www.youtube.com/watch?v=-k2X7guaArU" target="_blank">Design Pattern Reloaded by Remi Forax</a>
<a href="https://www.youtube.com/watch?v=e4MT_OguDKg" target="_blank">Design Patterns in the Light of Lambda Expressions by Venkat Subramaniam</a>

&nbsp;