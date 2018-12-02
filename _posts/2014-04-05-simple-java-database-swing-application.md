---
layout: post
title: Simple Java Database Swing Application
description: This tutorial demonstrates how to develop a Java application that connects with a database and provides a user interface and is targeted to students who studied Java, but lack experience.  This tutorial assumes basic knowledge of Java and it describes everything in great detail.  The main idea of this tutorial is not learn Java, but to help the viewers to gain experience in developing Java applications.  During the tutorial we explore various technologies, such as Maven.  These technologies are quite popular amongst developers communities and are considered as good traits when applying for jobs.
date: 2014-04-05 08:00:00 +0200
categories: swing, database, application
img: simple-java-database-swing-application/splash.png
permalink: simple-java-database-swing-application
author: Albert Attard
published: true
---

This tutorial demonstrates how to develop a simple Java application that connects with a database and provides a user interface using a series of videos compiled as a playlist ([Youtube](https://www.youtube.com/watch?v=-89CSE2RMlE&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB)).  The application is kept very simple while making use of various technologies, such as Maven ([Homepage](http://maven.apache.org/)).  This tutorial is targeted to college or university students who studied Java but lack experience and would like to do something productive.  The main idea of this tutorial is not learn Java, but to help the viewers to gain experience on various technologies that are commonly used in industry but not necessarily taught in schools or courses.  One of the greatest challenges a new developer has when starting a project from scratch is finding his or her way and make use of the right technology.  The playlist presented by this tutorial shows how to build a Java application that connects to a database and provides a user interface in a  step-by-step fashion.

Each video has its own project, with the exception of the first, and the viewers can download or view all code from the provided links.

## Overview

The tutorial comprises the following

1. [Part 1 - Overview](https://www.youtube.com/watch?v=-89CSE2RMlE&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Provides an overview about this tutorial and the application being developed.  No projects are associated with this video.

1. [Part 2 - Create Project](https://www.youtube.com/watch?v=tkqHSRvxdiI&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Creates a blank Maven project using the provided wizard.   ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-02))

1. [Part 3 - Connect with Database](https://www.youtube.com/watch?v=nOFaI56S_H4&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Describes how to manage the database libraries through Maven and establishes a database connection.   ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-03))

1. [Part 4 - DataSource and Flyway](https://www.youtube.com/watch?v=0oVM79TS6R8&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Introduces the concept of `DataSource` ([Java Doc](http://docs.oracle.com/javase/7/docs/api/javax/sql/DataSource.html)) and Flyway ([Homepage](http://flywaydb.org/)), a third party library that manages database migrations.  This video demonstrates how to manage the database updates while the application develops and evolves with minimal effort.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-04))

1. [Part 5 - Logging](https://www.youtube.com/watch?v=WrBNH0TxdEs&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Logging is fundamental part of any somewhat complex application as this can be used to provide detailed information about a problem.  This video focuses on two things.  It shows how to make use of logs using a popular logging facade.  Then it shows how to configure logging of third party libraries to work with the current logging framework.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-05))

1. [Part 6 - Domain](https://www.youtube.com/watch?v=rGJZ2Dz_mrc&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): So far we only dealt with technologies but very little about the project.  This video starts taking about the business logic related to the domain.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-06))

1. [Part 7 - Testing](https://www.youtube.com/watch?v=C_0CoxbDUKs&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): This video shows how to automate testing using JUnit ([Homepage](http://junit.org/)) test cases.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-07))

1. [Part 8 - Load From Database](https://www.youtube.com/watch?v=nr0Euix8WPE&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB):  So far we only saved data into the database.  This video demonstrates how to load data from the database and how to test this new functionality.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-08))

1. [Part 9 - Update and Delete](https://www.youtube.com/watch?v=1ztnam4ztkY&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): This video demonstrates how to update an existing record and how to delete one too.   ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-09))

1. [Part 10 - Simple User Interface](https://www.youtube.com/watch?v=HjzJaGPjAKw&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB):  This video shows how to build a simple user interface using Java Swing.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-10))

1. [Part 11 - Integration](https://www.youtube.com/watch?v=l7s5BvFYlAI&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Finally, the user interface is connected with the business logic.  This video shows how to connect the user interface code with the domain code and complete the application.   ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-11))

1. [Part 12 - Packaging and Distribution](https://www.youtube.com/watch?v=b4qC57D71uo&list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB): Once the program is ready, we need to package it and send it to our friends to use.  This video shows how to create an executable JAR which we can share with others.  ([GitHub](https://github.com/javacreed/simple-java-database-swing-application-part-07))</li>

The application shown in this tutorial is developed with Spring Tool Suite, also referred to as STS, ([Download](https://spring.io/tools3/sts/all)).  The viewers are free to use any integrated development environment, also referred to as IDE, ([Wiki](http://en.wikipedia.org/wiki/Integrated_development_environment)), they prefer.  In such event, please note that some of the commands or shortcuts may not work on a different IDE and some things may be organised differently.  If you are new to Java, it is highly recommended to use the same IDE as the one used in the videos as this will simplifies your learning experience.  Please note that by no means we are saying that the other IDE are not good or that STS is better than all others.  For more information about how to setup your environment please follow the [Setting up the Environment](http://www.javacreed.com/setting-up-the-environment/) tutorial.

### Resources and Technologies

Following is a list of resources technologies used in this tutorial.  These are shown in the order they appear.

1. Spring Source Tool Suite: [https://spring.io/tools3/sts/all](https://spring.io/tools3/sts/all)
1. Maven Repository: [http://mvnrepository.com/](http://mvnrepository.com/)
1. H2 Database: [http://www.h2database.com/html/main.html](http://www.h2database.com/html/main.html)
1. Flyway: [http://flywaydb.org/](http://flywaydb.org/)
1. SLF4J Logging Facade: [http://www.slf4j.org/](http://www.slf4j.org/)
1. Log4J Logger: [http://logging.apache.org/log4j/1.2/](http://logging.apache.org/log4j/1.2/)
1. JUnit: [http://junit.org/](http://junit.org/)

## Tutorial

[![Java Creed - Simple Java Database Swing Application]({{ "/assets/images/simple-java-database-swing-application/Video.png" | absolute_url }})](https://www.youtube.com/embed/-89CSE2RMlE?list=PLP7XIoztemXR1tzfnJSr9Z0odnBAHQ9DB)

Please note that the videos are recorded in HD (1080p).  Make sure to use the correct resolution to make the best out of these videos as shown below.

![Videos Resolution]({{ "/assets/images/simple-java-database-swing-application/Videos-Resolution.png" | absolute_url }})


## FAQs

1. I am not sure how to find path of the first argument in `getConnection()` method. Could you explain it please? I do not know if I should even call it a path?﻿

    The first argument of the `getConnection()` method is the database URL.  Here we are using H2 database ([Homepage](http://www.h2database.com/html/main.html)) which is saved in a local path relative to the project.  The path is "`target/db`", where "`target`" is the folder and "`db`" is the database name.  Say that you want to save the database under the directory called "`db`" with the name: "`app`".  Then the database connection URL will be "`jdbc:h2:db/app`".  Please make sure that this directory exists as otherwise you may get an error.

1. First, I use netbeans as my IDE and do not know where I should locate my sql file. I have put the sql file everywhere but I keep getting this error

    ```
    com.googlecode.flyway.core.api.FlywayException: Unable to determine URL for classpath location: db/migration     (ClassLoader: sun.misc.Launcher$AppClassLoader@1174ec5)
    ```

    how can I fix it?﻿

    Where are you saving the db/migration folder structure?  Are you using Maven?

    This path needs to be in your class path.  Say you package is "`com.mypkg`", then the folder "`db`" needs to be where the folder "`com`" is.  If you are using Maven, the "`db/migration`" folders can be organised as shown in the video.

1. How we can download the maven?﻿

   Please follow the tutorial titled: Setting Up The Environment ([Tutorial](http://www.javacreed.com/setting-up-the-environment/)), which shows you how to configure your    environment including Maven.  Maven is described at [Youtube - Maven](https://www.youtube.com/watch?v=zH0xcTDDTAs#t=916).

1. Where is the h2 database live exactly?  Does it live inside the hard-disk or in the net? Usually when connecting to MySQL database for example I use J/connector and a port 3308 to get a connection visa the local host.

    H2 is a pure Java database which can be embedded within a program.  In our example we are not installing a database but simply embedding it as part of our program.  Therefore our code include a database.  The tables and data are saved as files in a relative location to our project/program.  Our database is not actively listening on a particular port ﻿like MySQL.  Furthermore, only one program can work with such database.

1. `V1__initial_db.sql` has an unknown type.  I can't open it with eclipse as it's shown in your computer?﻿

    Can you please confirm that you are using Maven and that your project is correctly configured?  The file `V1__initial_db.sql`  is a simple text file and you should be able to open it.

Should you have any queries, please do not hesitate to contact me at: `albert@javacreed.com` or through [LinkedIn](https://www.linkedin.com/in/javacreed/).
