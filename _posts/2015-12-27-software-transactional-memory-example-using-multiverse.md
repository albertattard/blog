---
layout: post
title:  Software Transactional Memory Example using Multiverse
description: Transactions are an essential part of any business critical application as these ensure the integrity of the data being managed by the same application.  Transactions ensure that the data remains in a consistent and integral state after this is manipulated, thus mitigating the risk of invalid states.  Databases are synonymous with transactions, and many programming languages rely on the underlying database to provide the required transactional support.  This works well when all modified state is found at the database level.  Unfortunately, this is not always the case and there can be cases where the state is found at the application level rather than the database level.  In this article we will see how we can use Multiverse, a Software Transactional Memory, to provide transactions at the software level without using any databases.
date: 2015-12-27 08:00:00 +0200
categories: multiverse, concurrency
img: software-transactional-memory-example-using-multiverse/splash.png
permalink: software-transactional-memory-example-using-multiverse
author: Albert Attard
published: true
---

Transactions are an essential part of any business critical application as these ensure the integrity of the data being managed by the same application.  Transactions ensure that the data remains in a consistent and integral state after this is manipulated, thus mitigating the risk of an invalid state.  Databases are synonymous for transactions, and many programming languages rely on the underlying database to provide the required transactional support.  This works well when all state is found at the database level.  Unfortunately, this is not always the case and there can be cases where the state is found at the application level rather than the database level.  In this article, we will see how we can use Multiverse ([Git Hub](https://github.com/pveentjer/Multiverse)), a Software Transactional Memory, to provide transactions at the software level without using any databases.

All code listed below is available at [Git Hub](https://github.com/javacreed/software-transactional-memory-example-using-multiverse).  Most of the examples will not contain the whole code and may omit fragments which are not relevant to the example being discussed. The readers can download or view all code from the above repository.

## Transactions

Say that we have a bank account, which account has two mutable fields, the _current balance_ and the _date_ when this was last modified as shown in the following image.  Mutable fields are fields which can be modified after the object is created.  On the other hand, _immutable_ fields are fields which value will not change once the object is created.

![Bank Account]({{ "/assets/images/software-transactional-memory-example-using-multiverse/bank-account.png" | absolute_url }})

As shown in the above image, the account now has &euro;100 and was last modified at 8am on the third of March, 2015.  Say that I withdraw &euro;20 from this account at 10am of the following day.  Then the account details should be updated as shown next.

![Transfer Between Accounts]({{ "/assets/images/software-transactional-memory-example-using-multiverse/transfer-between-accounts.png" | absolute_url }})

The date when the balance was last modified and the current balance are bound together and these two need to be updated together.  This is very important to understand.  If we fail to update one of the fields, then none of the fields should be updated.  This property is also known as **atomic**.  If I try to withdraw more money than I currently have, the withdrawal should fail and both the date and the balance should remain unchanged.  In an atomic transaction, a series of operations either all occur, or nothing occurs ([Wiki](https://en.wikipedia.org/wiki/Atomicity_(database_systems))).

This is quite important as the data in memory or in a database, such as our banking accounts, need to always remain in a consistent and valid state.  How would you react if your account balance shows less money than it should following a technical error?

All operations should change the data from one valid state to another valid state.  The state before the withdrawal was valid and the state after the withdrawal remained valid.  This behaviour is provided by the consistency property.  The **consistency** property ensures that any transaction will bring the data from one valid state to another ([Wiki](https://en.wikipedia.org/wiki/Consistency_(database_systems))).  To build on our previous example, the state of the system following a withdrawal, irrespective of the outcome, should always be valid.

Nowadays systems are used by many users at the same time.  For example, a banking system can be accessed by many bank employees and also by the bank customers through the online portal.

![Banking System]({{ "/assets/images/software-transactional-memory-example-using-multiverse/banking-system.png" | absolute_url }})

This scenario introduces new challenges as changes made by one user may interfere with the work of another user.  Let us see this with an example.  Say that we would like to know how many accounts have not been used for some time.  We can use the following simple algorithm to achieve this.

```java
  List<Account> accounts = …
  long threshold = ...
  int count = 0;
  for(Account account : accounts) {
    if(account.getLastUpdate() < threshold) {
      total++;
    }
  }
```

This very simple and trivial algorithm does not work well within a concurrent (or as also known as _multithreaded_) environment as we will see next.  Say that we have a total of 10 accounts, five of which have not been used lately (since 2015-04-01).

|Number|Balance  |Last Used       |
|-----:|--------:|---------------:|
|     1|&euro;100|2015-01-01 10:00|
|     2|&euro;150|2015-02-11 14:10|
|     3|&euro;210|2015-04-21 09:50|
|     4|&euro;344|2015-03-02 08:30|
|     5| &euro;65|2015-05-08 14:30|
|     6|&euro;600|2015-03-15 09:25|
|     7| &euro;90|2015-01-14 08:40|
|     8| &euro;70|2015-05-21 15:20|
|     9| &euro;40|2015-05-04 14:40|
|    10| &euro;85|2015-04-15 15:10|

This algorithm starts iterating through the list and checks each account last modified date and increment accordingly.  

What can possibly go wrong here?  If the accounts are not modified while the algorithm counts them, then the results will be correct, 5.  Consider the following scenario.  The algorithm starts counting and before it reaches the 6th account, another user makes a transfer between account _1_ and account _7_.  Account _1_ was already counted as unused as it was last used before the threshold date.  The transfer is complete and both accounts are updated accordingly as shown in the following table.

|Number|Balance  |Last Used       |
|-----:|--------:|---------------:|
|     1| &euro;40|2015-06-01 10:00|
|     2|&euro;150|2015-02-11 14:10|
|     3|&euro;210|2015-04-21 09:50|
|     4|&euro;344|2015-03-02 08:30|
|     5| &euro;65|2015-05-08 14:30|
|     6|&euro;600|2015-03-15 09:25|
|     7|&euro;150|2015-06-01 10:00|
|     8| &euro;70|2015-05-21 15:20|
|     9| &euro;40|2015-05-04 14:40|
|    10| &euro;85|2015-04-15 15:10|

After this transfer is complete, only 3 are the accounts that have not been modified lately.  The counting algorithm continues and finishes counting.  The result produced is 4, which is incorrect.  We never had 4 unused accounts.  We had 5 unused accounts before the transfer and 3 after the transfer between the two unused accounts was made.  The values of 5 and 3 are both valid, but 4 is incorrect.  This issue occurred as the data was modified while the algorithm was processing it.

Transactions mitigate this issue by providing **isolation** where the changes made by one user do not interfere with the work of another user.  The isolation property ensures that the concurrent execution of transactions results in a system state that would be obtained if transactions were executed serially ([Wiki](https://en.wikipedia.org/wiki/Isolation_(database_systems))).

Database transactions also provide **durability**, which means that once a transaction is committed, it will remain committed, even if the database is abruptly powered off or crashes.  We will not dwell into this property as it is out of our scope since we are referring to software transactional model and these are inherently volatile.

## Multiverse

The word "_multiverse_" means more than one universe and is also the name of a software transactional memory library.  Multiverse allows us to turn a normal Java application into a transactional one, enabling most of the features normally provided by database transactions.  As in many other cases, one of the best ways to understand something is by example and Multiverse is no exception.

Consider the following class.

```java
package com.javacreed.examples.multiverse.part1;

import org.multiverse.api.StmUtils;
import org.multiverse.api.Txn;
import org.multiverse.api.callables.TxnCallable;
import org.multiverse.api.references.TxnInteger;
import org.multiverse.api.references.TxnLong;

public class Account {

  private final TxnLong lastUpdate;
  private final TxnInteger balance;

  public Account(final int balance) {
    this.lastUpdate = StmUtils.newTxnLong(System.currentTimeMillis());
    this.balance = StmUtils.newTxnInteger(balance);
  }

  public void adjustBy(final int amount) {
    adjustBy(amount, System.currentTimeMillis());
  }

  public void adjustBy(final int amount, final long date) {
    StmUtils.atomic(new Runnable() {
      @Override
      public void run() {
        balance.increment(amount);
        lastUpdate.set(date);

        if (balance.get() < 0) {
          throw new IllegalArgumentException("Not enough money");
        }
      }
    });
  }

  @Override
  public String toString() {
    return StmUtils.atomic(new TxnCallable<String>() {
      @Override
      public String call(final Txn txn) throws Exception {
        return String.format("%d (as of %tF %<tT)", balance.get(txn), lastUpdate.get(txn));
      }
    });
  }
}
```

The class shown above has two fields, the `lastUpdate` and `balance`.  The first field holds the last update date (in milliseconds) while the second field holds the account balance. **Please note that the monetary values, such as balance, should not be saved in an integer field as we did above**.  Instead one should opt for more suitable types such as `BigDecimal` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html)).  In this example, we used an integer type to keep these examples as simple as possible.

The example shown above needs more than just a little introduction.  Let us break the code into parts and analyse each part further.


1. All fields that will participate in the transaction need to be of a special type as shown next.

    ```java
      private final TxnLong lastUpdate;
      private final TxnInteger balance;
    ```

    We cannot use the primitives `long` and `int`, but instead, we need to use the special fields (referred to hereunder as _transactional fields_).  This is a big disadvantage as you cannot take any existing class and use it with Multiverse without first modifying its fields.  Furthermore, you cannot easily switch from one software transactional model to another as the fields are bound to this library.

    Another important observation is that all fields are `final`.  This means that the `Account` class is immutable.  In other words, the state of this class does not change.  We change the values of these objects but not the objects themselves.

    Most of the transactional fields are found under the `org.multiverse.api.references` package ([Git Hub](https://github.com/pveentjer/Multiverse/tree/master/multiverse-core/src/main/java/org/multiverse/api/references)).

    ```java
    import org.multiverse.api.references.TxnInteger;
    import org.multiverse.api.references.TxnLong;
    ```

1. The fields are initialised within the constructor when the instance of the `Account` is created.  Instead of using traditional constructors we use `StmUtils` ([Source Code](https://github.com/pveentjer/Multiverse/blob/master/multiverse-core/src/main/java/org/multiverse/api/StmUtils.java)) class to create and wire our objects.  

    ```java
      public Account(final int balance) {
        this.lastUpdate = StmUtils.newTxnLong(System.currentTimeMillis());
        this.balance = StmUtils.newTxnInteger(balance);
      }
    ```

    `StmUtils` uses the proper factory to initialise the required object.

1. Transactional fields, such as our fields `lastUpdate` and `balance`, can only be accessed from within a transaction.  If such fields are accessed outside a transaction, a `TxnMandatoryException` ([Source Code](https://github.com/pveentjer/Multiverse/blob/master/multiverse-core/src/main/java/org/multiverse/api/exceptions/TxnMandatoryException.java)) is thrown.  This is an important feature as Multiverse ensures that all transactional fields are accessed from within a transaction.  You can rest assure that such fields cannot be accessed outside a transaction.  Therefore all concurrent access is properly made as otherwise, it will never silently fail.  This is quite an advantage and anyone who worked with large multithreaded applications can tell you how challenging this is.

    The `adjustBy()` method increments (or decrements) the balance and updates the modified date.  If the new balance is negative, that is, there is not enough balance to cover this withdrawal, then an `IllegalArgumentException` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/lang/IllegalArgumentException.html)) is thrown.

    ```java
      public void adjustBy(final int amount, final long date) {
        StmUtils.atomic(new Runnable() {
          @Override
          public void run() {
            balance.increment(amount);
            lastUpdate.set(date);

            if (balance.get() < 0) {
              throw new IllegalArgumentException("Not enough money");
            }
          }
        });
      }
    ```

    If an exception is thrown, then both fields are reverted to their values before the transaction started.  This ensures that the state of the object moves from one valid state to another valid state.

    Note that all code is wrapped within a `Runnable` ([Java Doc](https://docs.oracle.com/javase/7/docs/api/java/lang/Runnable.html)) and executed by the `atomic()` shown next.

    ```java
        StmUtils.atomic(new Runnable() {
          @Override
          public void run() {
            /* Code within the transaction */
          }
        });
    ```

    All transaction fields, such as our `lastUpdate` and `balance` fields, need to be accessed and modified from within the `atomic()` method directly or indirectly.  The `atomic()` method creates the required environment for the transaction.  A `TxnMandatoryException` is thrown if such fields are not accessed properly.

    Later on, in this article, we will see how to access the transactional fields without having to wrap this within the `atomic()` method.

Using the above class is very simple as we see next.

```java
package com.javacreed.examples.multiverse.part1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example1 {

   private static final Logger LOGGER = LoggerFactory.getLogger(Example1.class);

   public static void main(final String[] args) {
     final Account a = new Account(10);
     a.adjustBy(-5);
     Example1.LOGGER.debug("Account {}", a);

     try {
       a.adjustBy(-10);
     } catch (final IllegalArgumentException e) {
       Example1.LOGGER.warn("Failed to withdraw money");
     }
     Example1.LOGGER.debug("Account {}", a);
   }
}
```

The above example makes two transactions.  We start with a balance of 10 and then withdraw 5 from this account leaving it with a new balance of 5.  Then we make another withdrawal, only this time we try to withdraw more money than we have.  This action is followed by an exception as expected.

## Deadlocks

One of the deadliest things in concurrent programming is deadlock ([Article](http://www.javacreed.com/what-is-deadlock-and-how-to-prevent-it/)).  Unfortunately, once in deadlock, the involved threads will always remain in deadlock until the JVM is restarted.  **Deadlocks cannot happen with Multiverse**.  The transaction mechanism prevents deadlocks by either restart the transaction or failing it.  This is quite a relief as we do not have to worry about deadlocks.  

In this section, we will work with two instances of account to make a transfer between one account and another.  Addressing such problem using a traditional approach (synchronising on the account instance) can easily lead to deadlocks.  Multiverse simplifies this as we will see in the following example.

```java
package com.javacreed.examples.multiverse.part2;

import org.multiverse.api.StmUtils;
import org.multiverse.api.Txn;
import org.multiverse.api.callables.TxnCallable;
import org.multiverse.api.references.TxnInteger;
import org.multiverse.api.references.TxnLong;

public class Account {

  private final TxnLong lastUpdate;
  private final TxnInteger balance;

  public Account(final int balance) {
    this.lastUpdate = StmUtils.newTxnLong(System.currentTimeMillis());
    this.balance = StmUtils.newTxnInteger(balance);
  }

  public void adjustBy(final int amount) {
    adjustBy(amount, System.currentTimeMillis());
  }

  public void adjustBy(final int amount, final long date) {
    StmUtils.atomic(new Runnable() {
      @Override
      public void run() {
        balance.increment(amount);
        lastUpdate.set(date);

        if (balance.get() < 0) {
          throw new IllegalArgumentException("Not enough money");
        }
      }
    });
  }

  @Override
  public String toString() {
    return StmUtils.atomic(new TxnCallable<String>() {
      @Override
      public String call(final Txn txn) throws Exception {
        return String.format("%d (as of %tF %<tT)", balance.get(txn), lastUpdate.get(txn));
      }
    });
  }

  public void transferTo(final Account other, final int amount) {
    StmUtils.atomic(new Runnable() {
      @Override
      public void run() {
        final long date = System.currentTimeMillis();
        adjustBy(-amount, date);
        other.adjustBy(amount, date);
      }
    });
  }
}
```

A new method called `transferTo()` is added to the `Account` class shown next.

```java
  public void transferTo(final Account other, final int amount) {
    StmUtils.atomic(new Runnable() {
      @Override
      public void run() {
        final long date = System.currentTimeMillis();
        adjustBy(-amount, date);
        other.adjustBy(amount, date);
      }
    });
  }
```

The `transferTo()` method takes two parameters, the account to which the money will be transferred to, and the amount of money to be transferred from this account to the given one.  If the amount is negative, then the reverse will happen.  In other words, `a.transferTo(b, -5)` will transfer 5 from account _b_ to account _a_.

Together with avoiding deadlocks, we need to make sure that the involved accounts remain in a valid state.  If the transaction fails, both accounts need to go back to their state before the transaction started.  We cannot risk that the money is deducted from one account and it is not transferred to the other one.  Similarly, we cannot create money by adding money to one account without withdrawing it from the other.  

```java
package com.javacreed.examples.multiverse.part2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example2 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example2.class);

  public static void main(final String[] args) {
    final Account a = new Account(10);
    final Account b = new Account(10);

    a.transferTo(b, 5);
    Example2.LOGGER.debug("Account (a) {}", a);
    Example2.LOGGER.debug("Account (b) {}", b);

    try {
      a.transferTo(b, 20);
    } catch (final IllegalArgumentException e) {
      Example2.LOGGER.warn("Failed to transfer money");
    }

    Example2.LOGGER.debug("Account (a) {}", a);
    Example2.LOGGER.debug("Account (b) {}", b);
  }
}
```

The above example makes two transfers.  The first transfer is expected to work without any problems but the second one should fail as the account from where the money is withdrawn does not have enough money to support this transaction.  This example will never deadlock even if we inject systematic delays to cause a collision and both accounts will remain in a valid state.

## Memory Usage

One of the main problems with Multiverse is the memory usage.  A Multiverse program can easily take more than 5 times the memory than a traditional one.  In this section, we will compare a traditional POJO (Plain Old Java Object) with a transactional one and see how much more memory the latter requires for the same amount of objects.

The `PojoAccount` class is shown next.

```java
package com.javacreed.examples.multiverse.part3;

public class PojoAccount {

  private final long lastUpdate;
  private final int balance;

  /* Methods removed for brevity */
}
</pre>
```

This class uses the traditional primitive fields instead of the transactional ones used by the `StmAccount` shown next.

```java
package com.javacreed.examples.multiverse.part3;

public class StmAccount {

  private final TxnLong lastUpdate;
  private final TxnInteger balance;

  /* Methods removed for brevity */
}
```

In both cases, the class methods are removed for brevity.  One million instances of `PojoAccount` take approximately 25MB of memory whereas the same amount of `StmAccount` instances needs approximately 141MB of memory.

![1 Million POJO Objects]({{ "/assets/images/software-transactional-memory-example-using-multiverse/1-million-pojo-objects.png" | absolute_url }})

![1 Million STM Objects]({{ "/assets/images/software-transactional-memory-example-using-multiverse/1-million-stm-objects.png" | absolute_url }})

These figures were extracted using the Your Kit Java Profiler ([Home Page](https://www.yourkit.com/features/)).  It is always recommended to use profilers, such as the Your Kit Java Profiler, to measure the performance of your programs instead of guessing or trying to build something yourself.

If we replicate the same experiment using 10 million objects instead of 1 million, we will find that the memory growth ratio is linear and we will need 10 times as much memory in both cases.  The following graph compares memory footprint ratio for these two types of objects.

![Memory Required Ratio]({{ "/assets/images/software-transactional-memory-example-using-multiverse/memory-required-ratio.png" | absolute_url }})

As anticipated at the beginning of this section, Multiverse requires more than 5 times the memory required by traditional objects.  This can be a problem with environments that have limited memory or larger applications that need to create many transactional objects.  One need to pay extra attention when calculating the memory required as this can easily lead to out of memory errors.

## Supported Types

Multiverse supports primitive types, such as `TxnInteger`, together with objects (non-primitives) such as `String`.  Multiverse has a special type, `TxnRef` ([Source Code](https://github.com/pveentjer/Multiverse/blob/master/multiverse-core/src/main/java/org/multiverse/api/references/TxnRef.java)), which can be used to represent all other (non-primitive) objects.  The `TxnRef` is a generic object that can take any object as its generic type.  As an example, we will change the last modified date field from primitive long to a `Date` ([Java Docs](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html)).

```java
package com.javacreed.examples.multiverse.part4;

import java.util.Date;

import org.multiverse.api.StmUtils;
import org.multiverse.api.Txn;
import org.multiverse.api.callables.TxnCallable;
import org.multiverse.api.references.TxnInteger;
import org.multiverse.api.references.TxnRef;

public class Account {

  private final TxnRef&lt;Date&gt; lastUpdate;
  private final TxnInteger balance;

  public Account(final int balance) {
    this.lastUpdate = StmUtils.newTxnRef(new Date());
    this.balance = StmUtils.newTxnInteger(balance);
  }

  public void adjustBy(final int amount, final Date date) {
    StmUtils.atomic(new Runnable() {
      @Override
      public void run() {
        balance.increment(amount);
        lastUpdate.set(date);

        if (balance.get() < 0) {
          throw new IllegalArgumentException("Not enough money");
        }
      }
    });
  }
}
```

As you can see, very little change was required.  Instead of a field of type long, now we have a date object of type `TxnRef`.

```java
  private final TxnRef<Date> lastUpdate;
```

This field can be modified using the setter method like any other object.

```java
        lastUpdate.set(date);
```

Using the `TxnRef` we can wrap any object into a transactional one, but **not all objects are suitable** to be wrapped by the `TxnRef` and extra care is required.  Mutable objects, such as our `Date`, are a bit dangerous as these can be accessed and modified outside a transaction as we will see next.

```java
package com.javacreed.examples.multiverse.part4;

import java.util.Date;
import java.util.concurrent.TimeUnit;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example5 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example5.class);

  public static void main(final String[] args) {
    final Account a = new Account(10);
    final Date date = new Date();
    a.adjustBy(-5, date);
    Example5.LOGGER.debug("Account {}", a);

    /* Date is modified outside the transaction */
    date.setTime(System.currentTimeMillis() - TimeUnit.HOURS.toMillis(1));
    Example5.LOGGER.debug("Account {}", a);
  }
}
```

The `date` object passed as a parameter to the `adjustBy()` can be modified outside a transaction and thus nullify all benefits gained when using Multiverse.  In the above example, we were able to modify the last modified date outside a transaction.

This problem only appears with mutable objects.  Immutable objects, on the other hand, such as `BigDecimal`, do not have this problem as these objects cannot be modified after they are created.  In order to be able to work with mutable objects, we need to make use of defensive copying ([Article](http://www.javacreed.com/what-is-defensive-copying/)) in order to protect the mutable objects from being accidentally or intentionally modified outside a transaction.

## Collections

A group of objects can be stored as an array or a collection.  <strong>Multiverse provides naïve collection support as this feature is incomplete</strong>.  One need to either fill in the blanks (provide the missing functionality) or avoid the missing functionality by using other methods.  In this section, we will see how collections can be used with Multiverse and will also see some of its limitations.

```java
package com.javacreed.examples.multiverse.part5;

import org.multiverse.api.StmUtils;
import org.multiverse.api.Txn;
import org.multiverse.api.callables.TxnCallable;
import org.multiverse.api.references.TxnInteger;
import org.multiverse.api.references.TxnLong;

public class Account {

  private final TxnLong lastUpdate;
  private final TxnInteger balance;

  public Account(final int balance) {
    this.lastUpdate = StmUtils.newTxnLong(System.currentTimeMillis());
    this.balance = StmUtils.newTxnInteger(balance);
  }

  public int getBalance(final Txn txn) {
    return balance.get(txn);
  }

  /* Some methods were removed for brevity */
}
```

Note how the `getBalance()` method accesses the transactional `balance` field without wrapping it within the `atomic()` method.

```java
  public int getBalance(final Txn txn) {
    return balance.get(txn);
  }
```

This is possible because the transaction (an instance of type `Txn` - [Source Code](https://github.com/pveentjer/Multiverse/blob/master/multiverse-core/src/main/java/org/multiverse/api/Txn.java)) is passed as a parameter to the object's `get()` method.  Whoever invokes the above-shown method needs to be within a transaction as it needs to provide the transaction as a parameter.  We will see how this is used in a minute.

The next class is the `Accounts` class as it is used to hold a collection of accounts.  The list of accounts is saved within an object of type `TxnList` ([Source Code](https://github.com/pveentjer/Multiverse/blob/master/multiverse-core/src/main/java/org/multiverse/api/collections/TxnList.java)) as shown next.

```java
package com.javacreed.examples.multiverse.part5;

import org.multiverse.api.StmUtils;
import org.multiverse.api.Txn;
import org.multiverse.api.callables.TxnDoubleCallable;
import org.multiverse.api.collections.TxnList;

public class Accounts {

  private final TxnList<Account> list;

  public Accounts() {
    list = StmUtils.newTxnLinkedList();
  }

  public void addAccount(final Account account) {
    StmUtils.atomic(new Runnable() {
      @Override
      public void run() {
        list.add(account);
      }
    });
  }

  public double calculateAverageBalance() {
    return StmUtils.atomic(new TxnDoubleCallable() {
      @Override
      public double call(final Txn txn) throws Exception {
        if (list.isEmpty(txn)) {
          return 0;
        }

        final int size = list.size(txn);
        double total = 0;
        /* Iteration is not supported by Multiverse at this stage */
        for (int i = 0; i < size; i++) {
          final Account account = list.get(txn, i);
          total += account.getBalance(txn);
        }
        return total / size;
      }
    });
  }
}
```

Let us break this class further and discuss each respective part in more detail.

1. The `Accounts` class contains a list of accounts of type `TxnList`.

    ```java
      private final TxnList&lt;Account&gt; list;
    ```

    Similar to the other transactional fields, the list is initialised using the `StmUtls` utilities class.

    ```java
      public Accounts() {
        list = StmUtils.newTxnLinkedList();
      }
    ```

    Multiverse (version <code>0.7.0</code>) only supports `LinkedList`.  The `ArrayList` is not supported.  Furthermore, some of the list functionality such as iterator is not supported as we will see later on.

1. Adding a new account to the list is very straightforward as shown next.

    ```java
      public void addAccount(final Account account) {
        StmUtils.atomic(new Runnable() {
          @Override
          public void run() {
            list.add(account);
          }
        });
      }
    ```

    The list modification happens within a transaction similar to what we saw before.

1. At the beginning of this article, we talked about isolation and the problems we can encounter when multiple threads interact with the same data.  Transactions, by providing isolation allow multiple threads to interact with the same data without interfering with each other.  The `calculateAverageBalance()` is a good example of isolation.

    ```java
      public double calculateAverageBalance() {
        return StmUtils.atomic(new TxnDoubleCallable() {
          @Override
          public double call(final Txn txn) throws Exception {
            if (list.isEmpty(txn)) {
              return 0;
            }

            final int size = list.size(txn);
            double total = 0;

            /* Iteration is not supported by Multiverse at this stage */
            for (int i = 0; i < size; i++) {
              final Account account = list.get(txn, i);
              total += account.getBalance(txn);
            }
            return total / size;
          }
        });
      }
    ```

    Adding or removing accounts by another thread will not affect the calculation of average by this thread.  Multiverse guards us against such problem.

    Unfortunately we cannot iterate the `NaiveTxnLinkedList` instance (the list object returned by `StmUtils.newTxnLinkedList()`) as the implementation is missing as shown next.

    ![Unimplemented Functionality]({{ "/assets/images/software-transactional-memory-example-using-multiverse/Unimplemented-functionality.png" | absolute_url }})

    Multiverse only provides an incomplete naïve implementation.

The `Accounts` class can be used like any other Java object as shown in the following example.

```java
package com.javacreed.examples.multiverse.part5;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example6 {

  private static final Logger LOGGER = LoggerFactory.getLogger(Example6.class);

  public static void main(final String[] args) {
    final Accounts accounts = new Accounts();
    accounts.addAccount(new Account(10));
    accounts.addAccount(new Account(20));

    Example6.LOGGER.debug("Average Balance: {}", accounts.calculateAverageBalance());
  }
}
```

## Conclusion

Multithreading is a complex topic and it can have unpredicted results.  What works without problems on one computer may continuously fail on another computer.  Furthermore, certain events may increase the probability of a problem manifesting while others reduce it.  A combination of different libraries can prove deadly as these may interfere with each other.  All this is somewhat random and does not help to understand the side effects of a multithreaded application.  The larger the application the worse this problem gets.  Multiverse can help us mitigate multithreaded related problems and provides several benefits such as:

1. Reduces the multithreading complexity
1. Removes the deadlock problem
1. Does not fail silently

In other words, you do not have to worry whether your application is thread-safe or not as all transactional fields need to be accessed within a transaction otherwise these will fail.  All you need to care about is the transaction scope.

Multiverse is not the panacea for all multithreaded problems as it has several limitations such as:

1. Incomplete
1. Requires a great deal of memory
1. Specific types
1. Poor support for immutable objects

In my opinion, one of the biggest drawbacks of this library is the fact that it is incomplete.  For example, only a naïve version of `LinkedList` is supported and some of its methods are not implemented.  With that said, one can always extend the current library and add any missing methods or functionality.  The second drawback is the amount of memory it requires.  An application can easily end up using five times the amount of memory simply because it works with Multiverse.  Such approach cannot be used with systems that have limited resources.  Furthermore, one needs to take extra care with pay per use systems as this approach will require more memory which may result in larger and more expensive server/service plans.

Multiverse requires its own types and thus it will prove challenging to switch an existing application to use Multiverse.  Multiverse types do not extend or inherit from the existing Java API and thus it makes them harder to use with existing code.  For example, you cannot sort a `TxnList` using the `Collections`'s `sort()` method ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#sort(java.util.List))) as `TxnList` does not extend `List` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/java/util/List.html)).

One needs to pay extra attention when working with mutable fields as we saw before.  Mutable fields can be modified outside a transaction and thus nullifies all benefits that Multiverse provides.
