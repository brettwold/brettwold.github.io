---
layout: post
title:  "Automatic versioning of Android application APK files"
subtitle: ""
date:   2016-03-15
categories: gradle, android
tags: gradle android
author: Brett Cherrington
---

If you want to automatically version your android application APK files and android library AAR files then here's a handy way to do this using a couple of simple gradle scripts.

Firstly create a gradle script in the root of your project called `artifacts.gradle`

```groovy
android.applicationVariants.all { variant ->
    def appName
    if (project.hasProperty("applicationName")) {
        appName = applicationName
    } else {
        appName = parent.name
    }

    variant.outputs.each { output ->
        def newApkName
        if (output.zipAlign) {
            newApkName = "${appName}-${output.baseName}-${variant.versionName}.apk"
        } else {
            newApkName = "${appName}-${output.baseName}-${variant.versionName}-unaligned.apk"
        }
        output.outputFile = new File(output.outputFile.parent, newApkName)
    }
}
```

Then refer to this script from within your applications main build.gradle file using something like the following

```groovy
apply from: "../artifacts.gradle"

ext.applicationName="MyAwesomeApplication"

version = "1.2.3"

```

Then call your usual build command `./gradlew assembleRelease` and you will find that the output APK now has the version number of your application appended to the main APK filename.

You can do the same thing for a library using the script below:

```groovy
android.libraryVariants.all { variant ->
    def appName
    if (project.hasProperty("applicationName")) {
        appName = applicationName
    } else {
        appName = parent.name
    }

    variant.outputs.each { output ->
        def newApkName = "${appName}-${output.baseName}-${version}.aar"
        output.outputFile = new File(output.outputFile.parent, newApkName)
    }
}
```

This will append the verison number to your AAR file name.

<script src="https://raw.github.com/moski/gist-Blogger/master/public/gistLoader.js" type="text/javascript"></script>
