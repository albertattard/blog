---
layout: post
title: How to get Auto-Generated Key with JdbcTemplate
description: Spring provides GeneratedKeyHolder class which can be used to retrieve the auto generated values as shown in this article
date: 2014-05-07 08:00:00 +0200
categories: auto-generated, jdbctemplate, spring
permalink: how-to-get-auto-generated-key-with-jdbctemplate
author: Albert Attard
published: true
---

Spring provides `GeneratedKeyHolder` ([Java Doc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/support/GeneratedKeyHolder.html)) class which can be used to retrieve the auto generated values.

Most of the examples will not contain the whole code and may omit fragments which are not relevant to the example being discussed. The readers can download or view all code from the above link.

The following class shows how to retrieve the auto generated key after a new value is added to the table.

```java
package com.javacreed.examples.spring;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;

public class ExampleDao {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  public long addNew(final String name) {
    final PreparedStatementCreator psc = new PreparedStatementCreator() {
      @Override
      public PreparedStatement createPreparedStatement(final Connection connection) throws SQLException {
        final PreparedStatement ps = connection.prepareStatement("INSERT INTO `names` (`name`) VALUES (?)",
            Statement.RETURN_GENERATED_KEYS);
        ps.setString(1, name);
        return ps;
      }
    };

    /* The newly generated key will be saved in this object */
    final KeyHolder holder = new GeneratedKeyHolder();

    jdbcTemplate.update(psc, holder);

    final long newNameId = holder.getKey().longValue();
    return newNameId;
  }
}
```

The readers can download or view the code from [https://github.com/javacreed/how-to-get-auto-generated-key-with-jdbctemplate/](https://github.com/javacreed/how-to-get-auto-generated-key-with-jdbctemplate/).
