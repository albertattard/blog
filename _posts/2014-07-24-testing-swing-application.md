---
layout: post
title:  Testing Swing Application
description: This tutorial demonstrates how to develop a simple Java Swing (or user interface) application and test the UI related logic using automated tests, such as JUnit, using a series of videos compiled as a playlist.  The application is kept very simple while making use of various technologies, such as EasyMock.  This tutorial is targeted to college or university students who studied Java but lack experience and would like to do something productive.  The main idea of this tutorial is not to learn Java, but to help the viewers to build testable Swing applications.  The playlist presented by this tutorial shows how to build a Java Swing application that can be easily tested using automated tests in a step-by-step fashion.
date: 2014-07-24 08:00:00 +0200
categories: testing, ui
img: testing-swing-application/splash.png
permalink: testing-swing-application
author: Albert Attard
published: true
---

This tutorial demonstrates how to develop a simple Java Swing (or user interface) application and test the UI related logic using automated tests, such as JUnit ([Homepage](http://junit.org/)), using a series of videos compiled as a playlist ([Youtube](https://www.youtube.com/watch?v=naWIFcghJhg&amp;list=PLP7XIoztemXTOGlS4BWhLCvi2VFDtI2my)). The application is kept very simple while making use of various technologies, such as EasyMock ([Homepage](http://easymock.org/)). This tutorial is targeted to college or university students who studied Java but lack experience and would like to do something productive. The main idea of this tutorial is not to learn Java, but to help the viewers to build testable Swing applications. The playlist presented by this tutorial shows how to build a Java Swing application that can be easily tested using automated tests in a step-by-step fashion.

All code for this tutorial is available at: [https://github.com/javacreed/testing-swing-application](https://github.com/javacreed/testing-swing-application). Each video has its own project and the viewers can download or view all code from the above link. With the exception of the first video, all other videos start from where the previous one left and thus the viewer can use the previous videos’ code as a starting point to the next video. The first video starts by creating a project from scratch.

## Overview

The tutorial comprises the following

1. [Part 1 - Introduction](https://www.youtube.com/watch?v=naWIFcghJhg&amp;index=1&amp;list=PLP7XIoztemXTOGlS4BWhLCvi2VFDtI2my): Introduce the problem and shows the limitations found when we tightly integrate the UI logic within the UI itself.

1. [Part 2 - Refactor UI Logic](https://www.youtube.com/watch?v=XzgiwBbgCO4&amp;index=2&amp;list=PLP7XIoztemXTOGlS4BWhLCvi2VFDtI2my): Refactor the project and splits the UI logic away from the UI code. This is the first step in creating testable Swing/UI applications. This part introduces the concept of _View_ and _Presenter_ and provides a clear definition of their boundaries.

1. [Part 3 - Mocking](https://www.youtube.com/watch?v=NmM__5F3l_M&amp;index=3&amp;list=PLP7XIoztemXTOGlS4BWhLCvi2VFDtI2my): Describes how to test the UI logic and mocking the UI (referred to as the _view_).

1. [Part 4 - Multithreading](https://www.youtube.com/watch?v=cDOPXgpyiPQ&amp;list=PLP7XIoztemXTOGlS4BWhLCvi2VFDtI2my&amp;index=4): Shows how the introduction of multiple threads can tamper our tests and provides a simple solution to handle this issue.

The application shown in this tutorial is developed with Spring Tool Suite, also referred to as STS, ([Download](http://spring.io/tools/sts/all)). The viewers are free to use any integrated development environment, also referred to as IDE, ([Wiki](http://en.wikipedia.org/wiki/Integrated_development_environment)), they prefer. In such event, please note that some of the commands or shortcuts may not work on a different IDE and some things may be organised differently. If you are new to Java, it is highly recommended to use the same IDE as the one used in the videos as this will simplify your learning experience. Please note that by no means we are saying that the other IDE are not good or that STS is better than all others. For more information about how to setup your environment please follow the [Setting up the Environment]({{ "/setting-up-the-environment/" | absolute_url }}) tutorial.

### Resources and Technologies

Following is a list of resources technologies used in this tutorial. These are shown in the order they appear.

1. Spring Source Tool Suite: [https://spring.io/tools3/sts/all](https://spring.io/tools3/sts/all)
1. Maven: [http://maven.apache.org/](http://maven.apache.org/)
1. EasyMock: [http://easymock.org/](http://easymock.org/)
1. JUnit: [http://junit.org/](http://junit.org/)
1. Jenkins (Continuous Integration): [http://jenkins-ci.org/](http://jenkins-ci.org/)
1. Hudson (Continuous Integration): [http://hudson-ci.org/](http://hudson-ci.org/)

## Tutorial

[![Testing Swing Application]({{ "/assets/images/testing-swing-application/Video.png" | absolute_url }})](https://www.youtube.com/embed/naWIFcghJhg?list=PLP7XIoztemXTOGlS4BWhLCvi2VFDtI2my)

Please note that the videos are recorded in HD (1080p). Make sure to use the correct resolution to make the best out of these videos as shown below.

![Videos Resolution]({{ "/assets/images/testing-swing-application/Videos-Resolution.png" | absolute_url }})

## Help

Should you have any queries, please do not hesitate to contact me at: `albert@javacreed.com` or through [LinkedIn](https://www.linkedin.com/in/javacreed/).

## Recommended Books

1. [Accelerate](https://itrevolution.com/book/accelerate/) by Nicole Forsgren, Jez Humble and Gene Kim
1. [Refactoring: second edition](https://lnkd.in/dFQRFP9) by Martin Fowler
