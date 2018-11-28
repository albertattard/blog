---
layout: post
title:  How to Write InputStream to RandomAccessFile
description: Writing an InputStream to a RandomAccessFile is quite simple as shown in the following example.  This example also makes use of FileLock to make sure that no one modifies the file while the data is being written to file.
date: 2014-12-22 08:00:00 +0200
categories: io
permalink: how-to-write-inputstream-to-randomaccessfile
author: Albert Attard
published: true
---

Writing an `InputStream` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/io/InputStream.html)) to a `RandomAccessFile` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/io/RandomAccessFile.html)) is quite simple as shown in the following example.  This example also makes use of `FileLock` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/nio/channels/FileLock.html)) to make sure that no one modifies the file while the data is being written to file.

```java
package com.javacreed.examples.io;

import java.io.BufferedInputStream;
import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.FileChannel;
import java.nio.channels.FileLock;
import java.nio.channels.ReadableByteChannel;

public class Example {
  public static void main(final String[] args) throws IOException {
    final File outputFile = new File("output.data");
    try (RandomAccessFile raf = new RandomAccessFile(outputFile, "rw"); FileChannel fc = raf.getChannel();) {
      final FileLock fl = fc.tryLock();
      if (fl == null) {
        /* Failed to acquire lock */
      } else {
        try (final ReadableByteChannel in = Channels.newChannel(new BufferedInputStream(Example.class
            .getResourceAsStream("/remote.data")))) {
          for (final ByteBuffer buffer = ByteBuffer.allocate(1024); in.read(buffer) != -1;) {
            buffer.flip();
            fc.write(buffer);
            buffer.clear();
          }
        } finally {
          fl.release();
        }
      }
    }
  }
}
```

This example also makes use of `ByteBuffer` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/nio/ByteBuffer.html)) to store the data read from the `InputStream` and then written to the file.  The code shown in this example can be downloaded from [https://github.com/javacreed/how-to-write-inputstream-to-randomaccessfile](https://github.com/javacreed/how-to-write-inputstream-to-randomaccessfile).
