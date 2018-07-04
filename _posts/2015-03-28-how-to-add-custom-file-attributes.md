---
layout: post
title: How to Add Custom File Attributes
description: The Files class, added in Java 7, provides an easy way to add and retrieve custom file's attributes
date: 2015-03-28 08:00:00 +0200
categories: IO
permalink: how-to-add-custom-file-attributes
author: Albert Attard
published: true
---

The `Files` class ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/nio/file/Files.html)), added in Java 7 ([Java Doc](http://docs.oracle.com/javase/7/docs/api/)), provides an easy way to add and retrieve custom file's attributes as shown in the following example.

```java
package com.javacreed.examples.io;

import java.nio.ByteBuffer;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.attribute.UserDefinedFileAttributeView;

public class Example {

  public static void main(final String[] args) throws Exception {
    final Path file = Paths.get(Example.class.getResource("/samples/example.txt").toURI()).toAbsolutePath();

    final UserDefinedFileAttributeView view = Files.getFileAttributeView(file, UserDefinedFileAttributeView.class);

    /* The file attribute */
    final String name = "com.javacreed.attr.1";
    final String value = "Custom Value 1";

    /* Write the properties */
    final byte[] bytes = value.getBytes("UTF-8");
    final ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
    writeBuffer.put(bytes);
    writeBuffer.flip();
    view.write(name, writeBuffer);

    /* Read the property */
    final ByteBuffer readBuffer = ByteBuffer.allocate(view.size(name));
    view.read(name, readBuffer);
    readBuffer.flip();
    final String valueFromAttributes = new String(readBuffer.array(), "UTF-8");
    System.out.println("File Attribute: " + valueFromAttributes);
  }
}
```

The above code can be downloaded from [Git Hub](https://github.com/javacreed/how-to-add-custom-file-attributes).
