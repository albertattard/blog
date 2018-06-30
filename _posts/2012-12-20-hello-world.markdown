---
layout: post
title:  "Hello World"
description: "Hello World" is the first program found in most programming books.  Even a book was also written that has "Hello World" as part of its title ("Hello World(s) -- From Code to Culture: A 10 Year Celebration of Java Technology").  We cannot have a website about Java without starting with the Hello World example.  As a typical hello word post, this post briefly introduces the blog and how it can be used.
date: 2012-12-20 08:00:00 +0200
categories: introduction
img: hello-world.png
author: Albert Attard
---
Hello World is the first program found in most programming books ([Wiki](http://en.wikipedia.org/wiki/Hello_world_program)).  Even a book was also written that has Hello World as part of its title ([Hello World(s) — From Code to Culture: A 10 Year Celebration of Java Technology](http://www.amazon.com/Hello-World-From-Code-Culture/dp/0131888676/)).  We cannot have a website about Java without starting with the Hello World example.

Following is the classic _Hello World_ program.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

This is a very simple program for an experienced developer but it looks cryptic for someone new to Java.

Let's start decrypting the above code.  The first line defines a class named `Main`.  All Java code is found in a `class`, (or better a within a `Type`, which is more generic than a `class`). Therefore all things that you need to be done by your program, will need to be written inside a `class` (or a `Type`) like this one. Different from other scripting languages, such as PHP ([Homepage](http://www.php.net/)), in Java, we cannot have code outside a `class`.

Right after the class name, `Main`, there is an opening curly bracket: `{`. There can be as many spaces as required between the class name and the open curly bracket.  Some programmers prefer to have the curly bracket in the next line instead as shown next.

```java
public class Main
{
    public static void main(String[] args)
    {
        System.out.println("Hello World");
    }
}
```

This is nothing but style.  Some programmers prefer two space indentation while other prefer four-space indentation.  This does not affect the program nor its performance, but it is recommended to be consistent.

The open curly bracket marks the opening of the `class` content. This is matched by another closing curly bracket which is found at the end of the example.  All code that belongs to this `class` is found between these two matching curly brackets.  Sometimes this is referred to as encapsulations, and we say that all code is encapsulated between the opening and closing curly brackets,

Note how the code is formatted. The formatting means nothing to Java but ease helps a lot in distinguishing what is inside what. As a rule of thumb, the code is indented whenever a curly bracket is opened.

Inside this `class`, we have a method called `main`. Methods, or functions as sometimes these are referred to, represents tasks that this `class` can perform. For example, the task run (just to clarify _run_ mean _running on a track_ or anywhere you like) can be expressed as a method. The `main` method prints the text _"Hello World"_ to the command prompt, using another method named `println()`, which is defined yet within another class.

The keyword `public` defines the visibility of the class and method.  `public` means that any other Java code can see and access the `main()` method within the `Main` class. The method `println()`, mentioned before is too `public`, that is why we can use it from our class. On the other hand, should the method be marked as `private`, then only the class where it is defined will be able to see and access this method.  This is like the private room in your house that no-one has access to but you.  This is how we control the visibility of code within Java.  With that said, Java 9 introduces modules ([article](https://www.oracle.com/corporate/features/understanding-java-9-modules.html)) which provide visibility across modules.

<span style="display:block;text-align:center"><img align="center" src="https://raw.githubusercontent.com/javacreed/hello-world/master/images/Clone%20or%20download.png" alt="Clone or Download"/></span>

This concludes this simple article. Here we simply introduced the most popular class and some key concepts about Java. We’re far from ready and many other interesting articles will follow.  Let us know what articles you would like to see here and we will try our best to accommodate you. Feedback is always welcome.


[jekyll-gh]:   https://github.com/javacreed/hello-word
