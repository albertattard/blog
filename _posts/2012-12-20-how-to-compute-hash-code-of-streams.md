---
layout: post
title:  How to compute the SHA-1 hash-code of a stream
description: This article shows how to compute the SHA-1 hash-code of a stream
date: 2015-02-06 08:00:00 +0200
categories: SHA-1, Hash Algorithms, Streams
permalink: how-to-compute-hash-code-of-streams
author: Albert Attard
published: true
---

Cryptographic Hash algorithms ([Wiki](http://en.wikipedia.org/wiki/Cryptographic_hash_function)), such as SHA-1 ([Wiki](http://en.wikipedia.org/wiki/SHA-1)), are known for digesting inputs of any size into a fixed length, nondeterministic, output.  Irrespective of the input size, the output is always of a fixed length, 160 bits (or 20 bytes) in case of SHA-1.  Different hash algorithms produce different lengths.

When processing large inputs, such as large files, it is convenient to work with streams, rather than loading all input to memory first.  The  following example illustrates how to compute the hash of a stream.

```java
package com.javacreed.examples.crypto;

import java.io.BufferedInputStream;
import java.net.URL;
import java.security.DigestInputStream;
import java.security.MessageDigest;
import java.util.Formatter;

public class Example {
  public static void main(final String[] args) throws Exception {

    final URL link = new URL("http://www.javacreed.com/");
    try (DigestInputStream in = new DigestInputStream(new BufferedInputStream(link.openStream()),
        MessageDigest.getInstance("SHA-1"))) {
      // Read the stream and do nothing with it
      while (in.read() != -1) {}

      // Get the digest and finialise the computation
      final MessageDigest md = in.getMessageDigest();
      final byte[] digest = md.digest();

      // Format as HEX
      try (Formatter formatter = new Formatter()) {
        for (final byte b : digest) {
          formatter.format("%02x", b);
        }

        final String sha1 = formatter.toString();
        System.out.println(sha1);
      }
    }
  }
}
```

This example applies to any other input/output stream, such as files and other cryptographic hash algorithms.  The above code can be downloaded from [https://github.com/javacreed/how-to-compute-hash-code-of-streams](https://github.com/javacreed/how-to-compute-hash-code-of-streams).
