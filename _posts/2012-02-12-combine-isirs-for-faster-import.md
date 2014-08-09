---
layout: post
title: "Combine ISIRs for Faster Import"
modified:
categories: "Banner Financial Aid"
excerpt: Tired of importing 5 or 6 ISIR files everyday? Learn how to combine ISIRs and speed up your import process.
tags: [banner, "financial aid", isirs]
image:
  feature: berea-college-quad.jpg
date: 2012-02-12T18:01:32-04:00
---

Loading ISIRs into Banner can be an intensive process, if done manually. There are often many different file classes each day. Combining the files can make it easier to load the files. Here is the process that I development to load ISIRs into Banner.

## Overview

We use EDConnect to download the ISIRs each day. These ISIRs are saved on a network drive in the directory, N:\DATA. From there, we move the ISIRs that we are ready to load to another director, `N:\DATA\1112-ISIRS`. We then use combine the ISIRs into one file, which is then moved to our Banner server, mapped to Z:\. From there, we import the file into Banner. This process is broken into two different runs. One run is for standard daily and pushed ISIRs; the other is for corrections.

## 1112MoveISIRs.bat

Once the ISIRs are downloaded, we run a batch file, `MoveISIRs.bat`, to move all of the standard daily and pushed ISIRs to the `N:\DATA\1112-ISIRS` directory. This batch file should be located in the `N:\DATA` directory. Here is the `1112MoveISIRs.bat` file.

{% highlight bat %}
@ECHO OFF
ECHO Processing...
move /Y idsa12*.* N:\DATA\1112-ISIRS
move /Y igsa12*.* N:\DATA\1112-ISIRS
move /Y igsg12*.* N:\DATA\1112-ISIRS
move /Y igaa12*.* N:\DATA\1112-ISIRS
move /Y isrf12*.* N:\DATA\1112-ISIRS
ECHO Completed Successfully
{% endhighlight %}

## 1112ISIRCombine.bat

After the ISIRs are moved into the `N:\DATA\1112-ISIRS` directory, we run the `1112ISIRCombine.bat` file. This batch file accomplishes two purposes. The first is to combine all of the ISIRs into one file, `1112esar.tap`. The second purpose is to backup the ISIRSs into a timestamp directory in the `N:\DATA\1112-ISIRS` directory. The batch file accomplishes this via the filecat.exe program, provided by Sungard. Both `1112ISIRCombine.bat` and `filecat.exe` should be located in the `N:\DATA\1112-ISIRS` directory. Here is the `1112ISIRCombine.bat` file.

{% highlight bat %}
@ECHO OFF
ECHO Processing...
filecat.exe -i idsa12* igsa12* igsg12* igaa12* isrf12* -o 1112esar.tap
ECHO Completed Successfully
{% endhighlight %}

## 1112MoveESAR.bat

Once the ISIRs are combined into the `1112esar.tap`, we run the `1112MoveESAR.bat` to move the it to the Banner server. From here, the file can be loaded into Banner via the RCBTP12 process. The `1112MoveESAR.bat` should be located in the `N:\DATA\1112-ISIRS` directory. Here is the `1112MoveESAR.bat`.

{% highlight bat %}
@ECHO OFF
ECHO Processing...
move /Y 1112esar.tap Z:\
ECHO Completed Successfully
{% endhighlight %}

That covers the process for standard daily and pushed files. We run this process every day, but it is flexible enough to handle if days are skipped. Skipping days will increase the occurrence of duplicate suspended records that will need to be manually handled. For us, duplicate records has not been a big concern, even when skipping days.

The process for correction ISIRs is very similar. We use a separate process because we load correction ISIRs with different settings than the standard ISIRs.

## 1112MoveCORRs.bat

The `1112MoveCORRs.bat` is very similar to `1112MoveISIRs.bat`. It should be located in `N:\DATA`. Here is `1112MoveCORRs.bat`.

{% highlight bat %}
@ECHO OFF
ECHO Processing...
move /Y igco12*.* N:\DATA\1112-ISIRS
ECHO Completed Successfully
{% endhighlight %}

## 1112CORRCombine.bat

The `1112CORRCombine.bat` is very similar to `1112ISIRCombine.bat` and accomplishes the same two purposes. It should be located in the `N:\DATA\1112-ISIRS` directory. Here is the `1112CORRCombine.bat`.

{% highlight bat %}
@ECHO OFF
ECHO Processing...
filecat.exe -i igco12* -o 1112esar.tap
ECHO Completed Successfully
{% endhighlight %}

## 1112MoveESAR.bat

This is the same 1112MoveESAR.bat as above.

If you use different directories, you can modify the files to change the directory names. All of these files can be downloaded from (link: https://github.com/jpangborn/bannerfileimport). (There are also files there to automate the import of Direct Loan/Pell/Teach files. Watch for a blog post about those.)