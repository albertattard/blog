---
layout: post
title:  How to Force Garbage Collection
description: The System.gc() does not guarantee that the garbage collection has executed and that the memory was cleaned from unused objects.  This post shows you how to force garbage collection.
date: 2014-08-22 08:00:00 +0200
categories: lang
permalink: how-to-force-garbage-collection
author: Albert Attard
published: true
---

The `System.gc()` does not guarantee that the garbage collection has executed and that the memory was cleaned from unused objects.  The following code fragment shows how to ensure the invocation of garbage collection.

```java
Object object = new Object();
final WeakReference<Object> ref = new WeakReference<>(object);
object = null;
while (ref.get() != null) {
  System.gc();
}
```

The above code make use of the `WeakReference` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/lang/ref/WeakReference.html)) class ability.

> Suppose that the garbage collector determines at a certain point in time that an object is weakly reachable. At that time it will atomically clear all weak references to that object and all weak references to any other weakly-reachable objects from which that object is reachable through a chain of strong and soft references. At the same time it will declare all of the formerly weakly-reachable objects to be finalizable. At the same time or at some later time it will enqueue those newly-cleared weak references that are registered with reference queues. ([reference](http://docs.oracle.com/javase/7/docs/api/java/lang/ref/WeakReference.html))

The weak reference will ensure us that the garbage collection has executed when the weak reference is cleared.

The code shown in this example can be downloaded from [https://github.com/javacreed/how-to-force-garbage-collection](https://github.com/javacreed/how-to-force-garbage-collection).
