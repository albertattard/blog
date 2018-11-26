---
layout: post
title:  How to install Subversion (SVN) in Eclipse (or Spring Tool Suite)
description: Subclipse is an Eclipse Plugin for SVN developed by Tigris that enables the developer to commit changes right from the IDE.
date: 2015-03-30 08:00:00 +0200
categories: IO
permalink: how-to-install-subversion-svn-in-eclipse
author: Albert Attard
published: true
---

Subclipse is an Eclipse Plugin for SVN developed by Tigris ([Homepage](http://subclipse.tigris.org/)).  Tigris have moved their site to [Git Hub](https://github.com/subclipse/subclipse/) as indicated on their website.  Subclipse [Wiki](https://github.com/subclipse/subclipse/wiki) provides detailed information about the Subclipse Eclipse Plugin and all available versions.  Note that different versions of the IDE may require a different version of the plugin.

## SVN Plugin Releases

The latest plugin update paths are listed in the following table require Eclipse 4.2 or later.

| IDE       | Update Link                                                   |
| --------- | ------------------------------------------------------------- |
| Latest    | `https://dl.bintray.com/subclipse/releases/subclipse/latest/` |
| 4.2.x     | `https://dl.bintray.com/subclipse/releases/subclipse/4.2.x/`  |

These old releases only require Eclipse 3.2 or later.

| SVN       | Update Link                                                   |
| --------- | ------------------------------------------------------------- |
| SVN 1.9.x | `https://dl.bintray.com/subclipse/archive/release/1.12.x/`    |
| SVN 1.8.x | `https://dl.bintray.com/subclipse/archive/release/1.10.x/`    |
| SVN 1.7.x | `https://dl.bintray.com/subclipse/archive/release/1.8.x/`     |

## Installation

Note that plugins installation requires the IDE to be restarted once the plugin is installed.  Kindly make sure that you are in a position to restart the IDE before staring the plugin installation.  Furthermore, the plugin is downloaded from the internet, thus the IDE needs to have access to the provided link.

New plugins can be installed by selecting the _Install new Software..._ menu option under the _Help_ menu.

![Install new Software]({{ "/assets/images/how-to-install-subversion-svn-in-eclipse/install-new-software.png" | absolute_url }})

Enter the plugin name and link

```
SVN Plugin - https://dl.bintray.com/subclipse/releases/subclipse/latest/
```

![Add Plugin]({{ "/assets/images/how-to-install-subversion-svn-in-eclipse/add-plugin.png" | absolute_url }})

Then press the _Add..._ button.  A dialogue will appear showing the plugin name and link.  These can be modified.  Click the _OK_ button.  The IDE will retrieve the information from the provided link.  Once the IDE retrieves the information, kindly select the items to be installed.  Note that you do not have to install all items.  With that said, the required items needs to be installed as other items depend on them.  In the following example, I selected all items.

![Select Items]({{ "/assets/images/how-to-install-subversion-svn-in-eclipse/select-items.png" | absolute_url }})

Click the _Next_ once ready.  read through the license agreements and accept only if you agree.  Before the installation starts, the IDE will show a security confirmation dialog.

![Security Warning]({{ "/assets/images/how-to-install-subversion-svn-in-eclipse/security-warning.png" | absolute_url }})

This dialogue is shown as this plugin is not coming from secure sources.  You need to click the _Install anyway_ button in order install this plugin.  Once installed, the IDE will show another dialogue confirming that the plugin is installed and the IDE needs to be restarted.  Click the _Restart Now_ button.
