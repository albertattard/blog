---
layout: post
title:  How to Lock a File
description: Locking a file is quite simple in Java.  The FileChannel class provides all methods we need to lock files.  The tryLock() method will try to obtain the lock on the file without waiting.  If the lock is acquired an instance of FileLock is returned, otherwise this method returns null
date: 2015-03-29 08:00:00 +0200
categories: IO
permalink: how-to-lock-a-file
author: Albert Attard
published: true
---

Locking a file is quite simple in Java.  The `FileChannel` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/nio/channels/FileChannel.html)) provides all methods we need to lock files.  The `tryLock()` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/nio/channels/FileChannel.html#tryLock())) method will try to obtain the lock on the file without waiting.  If the lock is acquired an instance of `FileLock` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/nio/channels/FileLock.html)) is returned, otherwise this method returns `null`.

The following example shows how to obtain a file lock.

```java
package com.javacreed.examples.io;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.FileChannel;
import java.nio.channels.FileLock;
import java.nio.channels.WritableByteChannel;
import java.nio.file.Files;

public class Example {
  public static void main(final String[] args) throws Exception {
    /* Read from file */
    final File inputFile = new File("src/main/resources/example.txt");
    try (final RandomAccessFile raf = new RandomAccessFile(inputFile, "rw")) {
      final FileChannel fc = raf.getChannel();
      final FileLock fl = fc.tryLock();
      if (fl == null) {
        /* Failed to acquire lock */
      } else {
        try (final ByteArrayOutputStream baos = new ByteArrayOutputStream();
            final WritableByteChannel outChannel = Channels.newChannel(baos)) {
          for (final ByteBuffer buffer = ByteBuffer.allocate(1024); fc.read(buffer) != -1;) {
            buffer.flip();
            outChannel.write(buffer);
            buffer.clear();
          }

          Files.write(Paths.get("target", "example.txt"), baos.toByteArray());
        } finally {
          fl.release();
        }
      }
    }
  }
}
```

The above code can be downloaded from [Git Hub](https://github.com/javacreed/how-to-lock-a-file.git).
