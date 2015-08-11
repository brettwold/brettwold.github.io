---
layout: post
title:  "Create your own personal maven repo"
subtitle: "Deploying your Android libraries to Bintray"
date:   2015-07-15
categories: cloud, bintray
tags: cloud bintray gradle
author: Brett Cherrington
---

This blog post discusses how you can add your Android (or plain old Java) library to your own personal online repository and then use them in your own projects by referring to them as dependencies. 

There are several choices of online repository available but for this blog I chose to use [Bintray](https://bintray.com/). Bintray allows you to upload source files for a variety of distribution types such as maven, docker, rpm etc. However, the main reason for choosing Bintray in this case is that it's also the home of the main Android library distributions called jcenter. This means if we want to deploy our library from our own repository to jcenter its as simple as clicking a button.

For the purposes of this blog post I am assuming you have an existing Android or Java project and are ready to deploy the binaries to Bintray. I am also assuming you are using Gradle as the build tool.

## Setting up gradle

Firstly we need to add the bintray plugin to our main build.gradle file

```groovy

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        ...
        classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:0.6"
    }
}

```

Next we need to apply the plugin to our project (or module within the project)

```groovy

	apply plugin: 'com.jfrog.bintray'
    apply plugin: 'maven-publish'

    project.version = "0.5"

```

As you can see I've also added a plugin called `maven-publish` which handles creating all the files we need for a maven release (e.g. pom.xml etc). Also notice I've defined the `project.version` so we can use it later.

Now that we have the plugin in our modules build.gradle file we can go ahead and configure it for our setup. The first thing we will need to add is our Bintray `username` and `key`. Obviously you'll need to register with Bintray and when you do so you'll create a username. Once registered and you've jumped through the usual email authentication hoops, the API key can be found under your profile on Bintray by navigating to Profile -> Edit -> API Key.

```groovy
bintray {
    user = System.getenv('BINTRAY_USER')
    key = System.getenv('BINTRAY_KEY')
	...
}
```
Once you've got them I recommend you create them as environment variables on your machine. That way you can refer to them in your gradle script as above but they are never checked into your source respository.

Next we need to setup some details about the package we are going to deploy to bintray.

```groovy

bintray {
    user = System.getenv('BINTRAY_USER')
    key = System.getenv('BINTRAY_KEY')
    pkg {
        repo = 'maven'
        name = 'MyAwesomeLibrary'
        licenses = ['Apache-2.0']
        vcsUrl = 'https://github.com/brettwold/MyAwesomeLibrary.git'
        version {
            name = "${project.version}"
            desc = 'MyAwesomeLibrary initial alpha release'
            released  = new Date()
            vcsTag = 'v0.1'
        }
    }
}

```

As you can see this includes some naming details about our library and some details about where it can be found on github. Above we have also defined the repo "type" which in this case is maven (you can use other types, but if you want to share it to jcenter later then you'll need a maven repo).

Because this is a maven repository we need to define some maven specific details in our gradle file as well. These should be defined outside of bintray block above but we do need to refer to this block from within the bintray block using the `publications` parameter. The code below shows everything we need to add to complete our release deploy.

```groovy

bintray {
    user = System.getenv('BINTRAY_USER')
    key = System.getenv('BINTRAY_KEY')
    publications = ['MyAwesomeLibrary']
    pkg {
        repo = 'maven'
        name = 'MyAwesomeLibrary'
        licenses = ['Apache-2.0']
        vcsUrl = 'https://github.com/brettwold/MyAwesomeLibrary.git'
        version {
            name = "${project.version}"
            desc = 'MyAwesomeLibrary initial alpha release'
            released  = new Date()
            vcsTag = 'v0.1'
        }
    }
}

publishing {
    publications {
        MyAwesomeLibrary(MavenPublication) {
            groupId 'com.brettwold'
            artifactId 'MyAwesomeLibrary'
            version "${project.version}"
            from components.java
        }
    }
}
```

And that is it! Now that we have defined all of the above we can create a release to bintray using the following gradle task

```bash

./gradlew bintrayUpload

```
> _Note:_ The example above assume you have created a standard java library not an Android 
> library. If your project is an Android library you will need to change the `from components.java` line. See the snippet below for the maven-publish details you would use to publish 
> an Android library (i.e. an .aar file).
>

```groovy

publishing {
    publications {
        MyAwesomeLibrary(MavenPublication) {
            groupId 'com.brettwold'
            artifactId 'MyAwesomeLibrary'
            version "${project.version}"
            artifact "${project.buildDir}/outputs/aar/${project.name}-release.aar"
        }
    }
}

```

If all is well this should compile and archive your library and upload it to bintray. You can check it by loggin into Bintray and navigating to your project. There are some other details you can then fill out such as description etc if you want to. 

Once uploaded to Bintray sucessfully we can go ahead and start using our library in other applications. This is done by firstly adding our new maven library to our list of repositories and then using the standard gradle dependencies command to import the library. As we have released it as a maven release this will also fetch any dependencies your library itself may have.

```groovy

    repositories {
        mavenCentral()
        ...
        maven {
            url 'https://dl.bintray.com/brettwold/maven/'
        }
    }

	dependencies {

	    compile com.brettwold:MyAwesomeLibrary:0.5

	}
```

So now you have your own personal release repository to hold all your libraries and potentially share with your teammates. Of course when you are ready for public release you can then add your release to jcenter by simply clicking on the "Add to jcenter" button from bintray. This will then (subject to approval) make your library available for everyone from the main jcenter repo.