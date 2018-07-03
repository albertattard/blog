---
layout: post
title:  Compress Large Text Data in Table
description: There can be cases where we need to save large text data into a database tables.  This data can take lots of space and may introduce new challenges which needs to be addressed.  Backups for example may take longer and if uncompressed, these will take more space.  In this article we explore several possibilities available when dealing with large data and compare the results obtained.
date: 2015-11-07 08:00:00 +0200
categories: performance, sql
img: compress-large-text-data-in-table/splash.png
permalink: compress-large-text-data-in-table
author: Albert Attard
published: true
---

There can be cases where we need to save large text data into a database table.  This data can take lots of space and may introduce new challenges which need to be addressed.  Backups, for example, may take longer and if uncompressed, these will take more space.  In this article, we explore several possibilities available when dealing with large data and compare the results obtained.

All code listed below is available at [Git Hub](https://github.com/javacreed/compress-large-text-data-in-table).  Most of the examples will not contain the whole code and may omit fragments which are not relevant to the example being discussed. The readers can download or view all code from the above link.

In this article, we make use of the MySQL database ([Homepage](https://www.mysql.com/)) and the SQL statements used may not work as expected with different databases.

This article is divided into three sections.  It starts by introducing the database structure and some common code which are used in the following section.  The second section compares several approaches and highlights their strengths and weaknesses.  In the last section is compares the results obtained in the previous section and provides a conclusion.

## Introduction

In this section, we go through the database and the tables used together with some common code.  The article flags all areas where changes may be required.

### Database

In this article, we discuss how to compress large text data and saved this into a database table.  The examples used in this article make use of one of the following two tables which can be created using the following SQL code.

```sql
DROP TABLE IF EXISTS `large_text_table`;
DROP TABLE IF EXISTS `compressed_table`;

CREATE TABLE `large_text_table` (
  `id` BIGINT(18) UNSIGNED NOT NULL AUTO_INCREMENT,
  `text` LONGTEXT NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

CREATE TABLE `compressed_table` (
  `id` BIGINT(18) UNSIGNED NOT NULL AUTO_INCREMENT,
  `compressed` MEDIUMBLOB NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

We have two test tables one holds the plain text data while the second holds the binary data.  The above SQL was executed against a MySQL database and uses the MyISAM ([Wiki](https://en.wikipedia.org/wiki/MyISAM)) table structure which is considerably faster than the alternative option InnoDB ([Wiki](https://en.wikipedia.org/wiki/InnoDB)).  Given that we do not need transactional support, we opted for this database engine to minimise the database overheads.  Some alterations may be required if a different database is used.

**Please be careful when choosing the database engine as this may have negative repercussions.**  Do not simply pick MyISAM just because it is faster.  MyISAM lack transaction support which may be required by your application.

The data source is provided by the following utility class, which for simplicity hard codes the database settings.

```java
public class DatabaseUtils {
  public static BasicDataSource createDataSource() {
    final BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/test_large_text");
    dataSource.setUsername("user");
    dataSource.setPassword("@?#%&£!");
    return dataSource;
  }
}
```

Kindly update the above database configuration as required.

### Common Code

In this article, we will see how we can compress text data to reduce the space used and also possibly protect such data using encryption algorithms.  In order to facilitate the tests, three classes were added which performs the following

1. Write data into a database table
1. Read data from the database table and validate it
1. Runs a simple test

These three classes are used by our tests, where each test defines the way data is written into the test database table.  For example when writing plain-text data we simply set the original text data to the prepared statement using the prepared statements `setString()` ([Java Doc](https://docs.oracle.com/javase/7/docs/api/java/sql/PreparedStatement.html#setString(int,%20java.lang.String))).  On the other hand, when dealing with compressed data, we cannot use the aforementioned method but instead we need to use `setBinaryStream()` ([Java Doc](https://docs.oracle.com/javase/7/docs/api/java/sql/PreparedStatement.html#setBinaryStream(int,%20java.io.InputStream))) version.

Let us start with the write data to table class which is shown next.

```java
public abstract class WriteDataToTable {

  private static final Logger LOGGER = LoggerFactory.getLogger(WriteDataToTable.class);

  private final String tableName;
  private final String columnName;
  private final Connection connection;
  private int testSize = 1000;

  public WriteDataToTable(final Connection connection, final String tableName, final String columnName) {
    this.connection = connection;
    this.tableName = tableName;
    this.columnName = columnName;
  }

  protected void clearTable() throws SQLException {
    try (Statement statement = connection.createStatement()) {
      statement.execute("TRUNCATE TABLE `" + tableName + "`");
    }
  }

  public long insert() throws Exception {
    WriteDataToTable.LOGGER.debug("Starting test");

    /* The data to be saved in the database */
    final String hamlet = IOUtils.toString(WriteDataToTable.class.getResourceAsStream("/Shakespeare Hamlet.txt"), "UTF-8");
    WriteDataToTable.LOGGER.debug("Loaded test ({} bytes long)", hamlet.length());

    /* Clear table */
    WriteDataToTable.LOGGER.debug("Clearing table so that the test starts on a fresh table");
    clearTable();

    WriteDataToTable.LOGGER.debug("Inserting the sample data {} time in table '{}'", testSize, tableName);
    final long startedInMillis = System.currentTimeMillis();
    /* Insert the text several times in the database using prepared statement */
    for (int i = 0; i < testSize; i++) {
      try (PreparedStatement statement = connection.prepareStatement("INSERT INTO `" + tableName + "` (`" + columnName + "`) VALUES (?)")) {
        setPreparedStatement(hamlet, statement);
        statement.execute();
      }
    }

    final long takenInMillis = System.currentTimeMillis() - startedInMillis;
    WriteDataToTable.LOGGER.debug("Inserted {} records in {} millis", testSize, takenInMillis);
    return takenInMillis;
  }

  protected abstract void setPreparedStatement(String data, PreparedStatement statement) throws Exception;

  public WriteDataToTable testSize(final int testSize) {
    this.testSize = testSize;
    return this;
  }
}
```

The class shown above may seem long and overwhelming but it is quite straightforward.  In a nutshell, the class reads a long text file and inserts the same text data several times into the same database table.  This class abstracts the part where the prepared statement sets the value to be passed to the database.  This will vary between test to test and is left for the implementation to deal with.  The rest of the code is the same for all tests implementations.  The `insert()` method is described in some details next.


1. The test inserts large text data into a database table.  This data is read from the resources available with the same project.  This is Shakespeare's Hamlet play ([Wiki](https://en.wikipedia.org/wiki/Hamlet)).

    ```java
        // The data to be saved in the database
        final String hamlet = IOUtils.toString(WriteDataToTable.class.getResourceAsStream("/Shakespeare Hamlet.txt"), "UTF-8");
    ```

    The Apache IO Common `IOUtils`</code> class ([Java Doc](https://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/IOUtils.html)) is used to ready this play as a string.

1. The table where the data is written is emptied so that all tests start on an empty table.

    ```java
        // Clear table
        WriteDataToTable.LOGGER.debug("Clearing table so that the test starts on a fresh table");
        clearTable();
    ````

    **The table needs to be empty as the validation process reads all records from this table and compares them with the original text to make sure that what is written can also be correctly read**.

    A small note about the `clearTable()` method.  This method makes use of the `TRUNCATE TABLE` SQL statement which may behave differently on different databases.  For example, in MySQL, the `TRUNCATE TABLE` SQL also resets the auto-generated ids while in H2 Database ([Homepage](http://www.h2database.com/html/main.html)) it just empties the table.

    ```java
     protected void clearTable() throws SQLException {
       try (Statement statement = connection.createStatement()) {
         statement.execute("TRUNCATE TABLE `" + tableName + "`");
       }
     }
    ```

    Kindly change this method to accommodate your database requirements.

1. The same data is inserted several times in order to be able to make a better measurement.  This process is measured to determine how long it takes to prepare the data to be written and write the data to the database table.

    ```java
        WriteDataToTable.LOGGER.debug("Inserting the sample data {} time in table '{}'", testSize, tableName);
        final long startedInMillis = System.currentTimeMillis();
        // Insert the text several times in the database using prepared statement
        for (int i = 0; i < testSize; i++) {
          try (PreparedStatement statement = connection.prepareStatement("INSERT INTO `" + tableName + "` (`" + columnName + "`) VALUES (?)")) {
            setPreparedStatement(hamlet, statement);
            statement.execute();
          }
        }
    ```

    Smaller samples may be easily affected by some anomalies caused by the underlying systems, such as the OS or DB, and it is always recommended to run with a decent sample size.  The decent sample varies from one problem to another and in this case we used a size of thousands of records.  We do so as this sample size is large enough to absorb any discrepancies caused by the underlying systems.

    The above code makes use of an abstract method, which method is implemented at a later stage by each test class.

    ```java
    protected abstract void setPreparedStatement(String data, PreparedStatement statement) throws Exception;
    ```

    This method will update the prepared statement with the correct data type, such as uncompressed text or compressed text (as a stream).  As hinted above, the write to table process is similar for all tests with the exception of one this, that is, what is written to the database.  This is defined by this method and each test will implement its approach.

1. Finally, we calculate the time taken in milliseconds and return this value

    ```java
        final long takenInMillis = System.currentTimeMillis() - startedInMillis;
        WriteDataToTable.LOGGER.debug("Inserted {} records in {} millis", testSize, takenInMillis);
        return takenInMillis;
    ```

As described above, this class provides all common code required to write data into a test table.  The next class reads the data back from the test table and makes sure that the correct data is found in this table.  There is no point if the example corrupts the data and write unreadable data.

The next class reads all data saved in test table and compare the value read with the original text.  Like before, the `ReadDataFromTable` is abstract and leaves the actual reading to be implemented by each individual test.  This allows for each test to determine how the data is read from the table.

```java
public abstract class ReadDataFromTable {

  private static final Logger LOGGER = LoggerFactory.getLogger(ReadDataFromTable.class);

  private final String tableName;

  private final String columnName;

  private final Connection connection;

  public ReadDataFromTable(final Connection connection, final String tableName, final String columnName) {
    this.connection = connection;
    this.tableName = tableName;
    this.columnName = columnName;
  }

  protected abstract String parseRow(ResultSet resultSet) throws Exception;

  public void verify() throws Exception {
    ReadDataFromTable.LOGGER.debug("Starting verification");

    // The data to be saved in the database
    final String hamlet = IOUtils.toString(ReadDataFromTable.class.getResourceAsStream("/Shakespeare Hamlet.txt"), "UTF-8");
    ReadDataFromTable.LOGGER.debug("Loaded test ({} bytes long)", hamlet.length());

    try (PreparedStatement preparedStatement = connection.prepareStatement("SELECT `" + columnName + "` FROM `" + tableName + "`");
        ResultSet resultSet = preparedStatement.executeQuery()) {

      int rowNumber = 0;
      final long startedOnInMillis = System.currentTimeMillis();
      for (; resultSet.next(); rowNumber++) {
        final String inTable = parseRow(resultSet);
        if (hamlet.equals(inTable) == false) {
          throw new RuntimeException("Invalid data found in row " + rowNumber);
        }
      }

      final long takenInMillis = System.currentTimeMillis() - startedOnInMillis;
      ReadDataFromTable.LOGGER.debug("Verified {} record in {} millis", rowNumber, takenInMillis);
    }
  }
}
```

The `verify()` method is described in detail next.


1. The original data is read again from the source file.

    ```java
        // The data to be saved in the database
        final String hamlet = IOUtils.toString(ReadDataFromTable.class.getResourceAsStream("/Shakespeare Hamlet.txt"), "UTF-8");
    ```

    This will be used to compare each row read and makes sure that the read value is the same as the original text.

1. Reads all data from the table using a prepared statement and validated each row read.

    ```java
        try (PreparedStatement preparedStatement = connection.prepareStatement("SELECT `" + columnName + "` FROM `" + tableName + "`");
            ResultSet resultSet = preparedStatement.executeQuery()) {

          int rowNumber = 0;
          final long startedOnInMillis = System.currentTimeMillis();
          for (; resultSet.next(); rowNumber++) {
            final String inTable = parseRow(resultSet);
            if (hamlet.equals(inTable) == false) {
              throw new RuntimeException("Invalid data found in row " + rowNumber);
            }
          }
    ```

The actual retrieval of the information is delegated to the subclass.

    ```java
      protected abstract String parseRow(ResultSet resultSet) throws Exception;
    ```

    Each test can then implement the appropriate method to retrieve the data from the `ResultSet` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/sql/ResultSet.html)) and convert it back to a string.

1. Finally, we measure the time taken and log this.

    ```java
          final long takenInMillis = System.currentTimeMillis() - startedOnInMillis;
          ReadDataFromTable.LOGGER.debug("Verified {} record in {} millis", rowNumber, takenInMillis);
    ```

For convenience, these two classes are grouped into one class which is shown next.

```java
public abstract class ExampleTest {

  private WriteDataToTable writeDataToTable;

  private ReadDataFromTable readDataFromTable;

  public ExampleTest(final Connection connection, final String tableName, final String columnName) {
    writeDataToTable = new WriteDataToTable(connection, tableName, columnName) {
      @Override
      protected void setPreparedStatement(final String data, final PreparedStatement statement) throws Exception {
        ExampleTest.this.setPreparedStatement(data, statement);
      }
    };

    readDataFromTable = new ReadDataFromTable(connection, tableName, columnName) {
      @Override
      protected String parseRow(final ResultSet resultSet) throws Exception {
        return ExampleTest.this.parseRow(resultSet);
      }
    };
  }

  protected abstract String parseRow(ResultSet resultSet) throws Exception;

  public long runTest() throws Exception {
    final long timeTakenInMillis = writeDataToTable.insert();
    readDataFromTable.verify();
    return timeTakenInMillis;
  }

  protected abstract void setPreparedStatement(String data, PreparedStatement statement) throws Exception;

}
```

This class is quite simple.  It has two abstract methods, one responsible for writing the data and the other for reading the data.  These have the same method signature as those found in `WriteDataToTable` and  `ReadDataFromTable`.

The tests shown in the following sections make use of this class and override these two methods with the appropriate implementation.

## Tests

In this section, we carry out fours tests and compare their results.

### Plain Large Text

The first test writes the data as it comes.  It does not alter the data and simply write and reads the data as text.

```java
public class Example1 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example1.class);

  public static void main(final String[] args) throws Exception {
    try (BasicDataSource dataSource = DatabaseUtils.createDataSource(); Connection connection = dataSource.getConnection()) {
      final ExampleTest test = new ExampleTest(connection, "large_text_table", "text") {
        @Override
        protected String parseRow(final ResultSet resultSet) throws Exception {
          return resultSet.getString("text");
        }

        @Override
        protected void setPreparedStatement(final String data, final PreparedStatement statement) throws SQLException {
          statement.setString(1, data);
        }
      };
      test.runTest();
    }
    Example1.LOGGER.debug("Done");
  }
}
```

Inserting a thousand records of the long text play, Shakespeare's Hamlet, into the table without modifying it take almost 8 seconds.  Reading such data is quite fast as it takes less than half of a second.

## GZIP

In the second example, we compress the data using the Java implementation of the GZIP ([Package Summary](https://docs.oracle.com/javase/7/docs/api/java/util/zip/package-summary.html)) lossless compression algorithm as shown next.

```java
public class Example2 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example2.class);

  public static void main(final String[] args) throws Exception {
    try (BasicDataSource dataSource = DatabaseUtils.createDataSource(); Connection connection = dataSource.getConnection()) {
      final ExampleTest test = new ExampleTest(connection, "compressed_table", "compressed") {
        @Override
        protected String parseRow(final ResultSet resultSet) throws Exception {
          try (GZIPInputStream in = new GZIPInputStream(resultSet.getBinaryStream("compressed"))) {
            return IOUtils.toString(in, "UTF-8");
          }
        }

        @Override
        protected void setPreparedStatement(final String data, final PreparedStatement statement) throws Exception {
          // Compress the data before inserting it. We need to compress before inserting the data to make this process as realistic as possible.
          final ByteArrayOutputStream baos = new ByteArrayOutputStream(data.length());
          try (OutputStream out = new GZIPOutputStream(baos, data.length())) {
            out.write(data.getBytes("UTF-8"));
          }
          statement.setBinaryStream(1, new ByteArrayInputStream(baos.toByteArray()));
        }
      };
      test.runTest();
    }
    Example2.LOGGER.debug("Done");
  }
}
```

The data is compressed before written to the database as shown next.

```java
          final ByteArrayOutputStream baos = new ByteArrayOutputStream(data.length());
          try (OutputStream out = new GZIPOutputStream(baos, data.length())) {
            out.write(data.getBytes("UTF-8"));
          }
          statement.setBinaryStream(1, new ByteArrayInputStream(baos.toByteArray()));
```

The data is compressed every time before writing it into the table to simulate a production environment.  One understands that compression will increase in the time required as additional processing is required.  On the other hand, such approach writes fewer data to the disk which should improve the write time and compensate for the time lost during compression.  Unfortunately, this approach is quite slow and the compression takes far longer than what is saved from writing fewer data to the disk.  Inserting a thousand records of the long text play, Shakespeare's Hamlet, into the table after compressing each instance takes more than 19 seconds.  Reading such data is slower too as it needs more than 2 seconds.  This approach reduced the space required by more than half, but it takes more to process.  In summary, this approach trades time with space.  You need less space on disk but requires further processing time.

### LZ4

LZ4 ([Homepage](http://cyan4973.github.io/lz4/)) is another lossless compression algorithm, similar to GZIP, which is considerably faster than GZIP as we will see in this test.

```java
public class Example3 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example3.class);

  public static void main(final String[] args) throws Exception {
    try (BasicDataSource dataSource = DatabaseUtils.createDataSource(); Connection connection = dataSource.getConnection()) {
      final ExampleTest test = new ExampleTest(connection, "compressed_table", "compressed") {
        @Override
        protected String parseRow(final ResultSet resultSet) throws Exception {
          try (InputStream in = new LZ4BlockInputStream(resultSet.getBinaryStream("compressed"))) {
            return IOUtils.toString(in, "UTF-8");
          }
        }

        @Override
        protected void setPreparedStatement(final String data, final PreparedStatement statement) throws Exception {
          final ByteArrayOutputStream baos = new ByteArrayOutputStream(data.length());
          try (OutputStream out = new LZ4BlockOutputStream(baos)) {
            out.write(data.getBytes("UTF-8"));
          }
          statement.setBinaryStream(1, new ByteArrayInputStream(baos.toByteArray()));
        }
      };
      test.runTest();
    }
    Example3.LOGGER.debug("Done");
  }
}
```

Instead of `GZIPOutputStream` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/util/zip/GZIPOutputStream.html)), in this example we use `LZ4BlockOutputStream` ([Source Code](https://github.com/jpountz/lz4-java/blob/master/src/java/net/jpountz/lz4/LZ4BlockOutputStream.java)) as shown next.

```java
          final ByteArrayOutputStream baos = new ByteArrayOutputStream(data.length());
          try (OutputStream out = new LZ4BlockOutputStream(baos)) {
            out.write(data.getBytes("UTF-8"));
          }
          statement.setBinaryStream(1, new ByteArrayInputStream(baos.toByteArray()));
```

LZ4 is faster compared to GZIP and faster to write data even when compared to the plain text approach.  Compressing and inserting a thousand records takes about 5 seconds.  Reading data is faster than GZIP as it needs about 800 milliseconds, but slightly slower when compared to the plain text method.  Therefore, this method is faster when it comes to writing but a bit slower when it comes to reading.

LZ4 can be tuned to compress the data further at a cost of performance, but in this example, we used the default settings.  Furthermore, we can reduce the compression ration and obtain faster performance too at the cost of space.

### LZ4 and Encryption

As a final test, we encrypt the data using AES ([Wiki](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)) after this is compressed with the LZ4 compression algorithm with the default settings as above.  As one would expect encryption will add an overhead and this example is expected to obtain worse results when compared with the previous test.

```java
public class Example4 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example4.class);

  public static void main(final String[] args) throws Exception {
    try (BasicDataSource dataSource = DatabaseUtils.createDataSource(); Connection connection = dataSource.getConnection()) {
      final ExampleTest test = new ExampleTest(connection, "compressed_table", "compressed") {

        @Override
        protected String parseRow(final ResultSet resultSet) throws Exception {
          try (InputStream in = new LZ4BlockInputStream(CryptoUtils.wrapInToCipheredInputStream(resultSet.getBinaryStream("compressed")))) {
            return IOUtils.toString(in, "UTF-8");
          }
        }

        @Override
        protected void setPreparedStatement(final String data, final PreparedStatement statement) throws Exception {
          final ByteArrayOutputStream baos = new ByteArrayOutputStream(data.length());
          try (OutputStream out = new LZ4BlockOutputStream(CryptoUtils.wrapInToCipheredOutputStream(baos))) {
            out.write(data.getBytes("UTF-8"));
          }
          statement.setBinaryStream(1, new ByteArrayInputStream(baos.toByteArray()));
        }

      };
      test.runTest();
    }
    Example4.LOGGER.debug("Done");
  }
}
```

As expected this approach is somewhat slower than the previous one.  To compress and then encrypt a thousand records take almost 10 seconds which is almost twice as much as the version that does not use encryption.  Reading and decryption are considerably slower too when compared to the previous approach.  The overhead added by the decryption process makes this the worse approach from all tests.  This approach requires the same size as the one before as the encryption does not inflate or deflate the message.

## Comparison

In the previous section, we described fours tests cases, where each test case inserted a thousand records and then read and verify these records.  The following stacked graph compares the time required to write and then read these records.

![Write and Reed Performance]({{ "/assets/images/compress-large-text-data-in-table/write-and-reed-performance.png" | absolute_url }})

In all cases, the write was slower than the read.  The next bar graph compares the space required on the disk.

![Tables Sizes]({{ "/assets/images/compress-large-text-data-in-table/tables-sizes.png" | absolute_url }})

As expected, the plain text test requires most space while GZIP requires the least space.  Compressing data makes the data smaller and reduces the required space, but this comes at a cost of performance.  While database backups may run faster, processing such data will be hindered as it needs to be uncompressed first.  

Premature optimisation is the root of all evil.  It is imperative to measure the changes before applying them.  Setting up a small and isolated test similar to the one shown here is not complex.  Many are those who simply go ahead and implement such changes in production without measuring their impact first.  Such approach may come with a detriment effect on the overall system performance.
