---
layout: post
title: "Install EDConnect 7 on Windows 7"
modified:
categories: "EDConnect"
excerpt: EDConnect 7 doesnt' officially support Windows 7. Learn how to work around the issues to get it installed and working.
tags: [edconnect, "financial aid", windows]
comments: true
image:
  feature: berea-college-draper.png
date: 2012-02-14T00:00:00-05:00
---

EDConnect is not currently supported on Windows 7 by the Department of Education. In spite of the lack of support, EDConnect will work on Windows 7 if installed in a particular manner. Read on for installation instructions and details of what does not work.

## Installation

Installing EDConnect on Windows 7 requires a few prerequisites and a few extra steps. The main prerequisite is that the user be an Administrator. I was unable to find a method to install EDConnect as a standard user. It may be possible if the EDConnect database will reside on the local PC, but not if the database is located on a Network drive.

Even with the user as an Administrator, you must still right click on the EDConnect installation file and select “Run as Administrator”. You will be asked to confirm this action. This will start the installation process. You can proceed through the installation process normally, except that you must change the location of the install. Installing in “Program Files” will cause EDConnect to not work. I have installed in C:\EDConnect. This works without problem. Continue through the completion of installation. You may receive a error message at completion indicating that several files could not be registered. In spite of these errors, EDConnect will work properly.

One part of EDConnect that does not work properly is setting up the connection to your Mailbox. Fortunately, it is easy to manually adjust the configuration file. Open Notepad. Assuming you installed EDConnect in C:\EDConnect, open the EDConnect.ini from that directory. Once open, change the line “Connection=” to “Connection=Direct Connection”. Save the file.

EDConnect can now be launched used normally. I have not run into any problems with day to day usage. These installation instructions were tested with an already created database. I am not sure if there would be any differences if creating the database.