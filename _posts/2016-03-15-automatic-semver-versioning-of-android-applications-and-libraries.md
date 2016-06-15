---
layout: post
title:  "Automatic SemVer versioning of Android applications"
subtitle: ""
date:   2016-03-15
categories: gradle, android
tags: gradle android
author: Brett Cherrington
---

If you want to automatically version your android application APK files and android library AAR files then here's a handy way to do this using a couple of simple gradle scripts.

Firstly create a gradle script in the root of your project called `artifacts.gradle`

<div class="gistLoad" data-id="89939fe171e8698a65ca95365a806f1" data-file="artifacts.gradle" id="gist-89939fe171e8698a65ca95365a806f1">Loading https://gist.github.com/89939fe171e8698a65ca95365a806f1....</div>

Then refer to this script from within your applications main build.gradle file using the following

```groovy
apply from: "../artifacts.gradle"

ext.applicationName="MyAwesomeApp"

```

Then call your usual build command `./gradlew assembleRelease` and you will find that the output APK now has the version number of your application appended to the main APK filename.

You can do the same thing for a library using the script below:

<div class="gistLoad" data-id="89939fe171e8698a65ca95365a806f1e" data-file="artifacts.gradle" id="gist-89939fe171e8698a65ca95365a806f1e">Loading https://gist.github.com/89939fe171e8698a65ca95365a806f1e....</div>

This will append the verison number to your AAR file name.

<script src="https://raw.github.com/moski/gist-Blogger/master/public/gistLoader.js" type="text/javascript"></script>
