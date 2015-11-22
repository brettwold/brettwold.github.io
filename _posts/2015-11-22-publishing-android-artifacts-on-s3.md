---
layout: post
title:  "Publishing Android Liraries to S3"
date:   2015-11-22
categories: android publishing
tags: android
author: Brett Cherrington
---

# Publishing Android Liraries to S3

If you want to share your java and Android libraries with other colleagues in your team but don't want to make them public then you need a private artifact repository. As always there are numerous ways of doing this most commonly using something like [Artifactory](https://www.jfrog.com/open-source/) by jFrog to host you artifacts. Whilst this is usually very easy to do with these artifact repos, usually if you want to make these artifacts private then you'll need to purchase a license or host your own version.

If you're just looking for a simple low cost way of sharing artifacts within your team then you can now do it really simply using S3. This post give details on how you can publish your Android libraries or java projects to S3 and then make use of them in other projects just like you do with public open source projects. Your library can be published to S3 as a maven artifact and will include the necessary POM files etc. You will need to use gradle 2.4 or above and I'll assume the user is familiar with the basic concepts of using AWS.

## Setting up S3

First you're going to need an S3 bucket to publish your artifacts into. Once you have created one you'll need to give access to it. You can of course make this bucket public for reads and writes but that kind of defeats the point of what we are doing. So next you'll need to create a new user in IAM specifically for uploading and downloading artifacts to/from this bucket. Make sure of course you keep a note of the user access id and secret and also make a note of the new users ARN (usually at the top of the page). Then switch back to the S3 and add the following policy against your new bucket.

```xml
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Stmt1428433844452",
			"Effect": "Allow",
			"Principal": {
				"AWS": "arn:aws:iam::[{ARN_NUMBER_HERE]:user/my-release-user"
			},
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": "arn:aws:s3:::my-releases.example.com/*"
		}
	]
}
```
This allows the user to authenticate to the bucket and get and put files to/from it.

## Setting up Gradle to publish your library

Firstly import the maven-publish plugin

```groovy

apply plugin: 'maven-publish'

```

Next we will setup our bucket location as a new publishing repository using the code below. Notice that the access kay and secret have been entered as gradle variables. You can of course put your id and key in this file directly but I stringly recommend against that. Instead place them in your gradle.properties file or set them as environment variables so that they are not checked into your source control.

```groovy
publishing {
    repositories {
        maven {
            url "s3://my-releases.example.com/maven2"
            credentials(AwsCredentials) {
                accessKey "${aws_accessid}"
                secretKey "${aws_accesskey}"
            }
        }
    }
}
```

All that's left to do now is tell gradle what our artifacts are to publish. As per the Gradle docs for a java library all we need to do is add the following.

```groovy
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
}
```
However, for an Android library this is a bit more involved as we need to create a jar file from the Android sources to publish. This is shown in the block below, as you can see we need to add a new task that firstly creates a jar file from our library and then tell our maven publisher about it.

```groovy
publishing {
    publications {
        mavenJava(MavenPublication) {
            artifact jarRelease
        }
    }
}

android.libraryVariants.all { variant ->
    def name = variant.buildType.name
    if (name.equals(com.android.builder.core.BuilderConstants.DEBUG)) {
        return; // Skip debug builds.
    }
    def task = project.tasks.create "jar${name.capitalize()}", Jar
    task.dependsOn variant.javaCompile
    task.from variant.javaCompile.destinationDir
    artifacts.add('archives', task);
}

```

Now we can publish our artifact to S3 by simply running

```
gradlew publish
```

If all goes well you should see your jar file and the required maven xml files uploaded in your S3 bucket.

## Using your new artifact

Now you want to make use of your shiny new library. This is a simple process of telling gradle about your new repo and then depending on your library at compile time as you usually do. The code below can be used in your top level gradle file to add a reference to your new repo. Again make sure the id and key values are kept in a properties file outside of your source control.

```groovy
allprojects {
    repositories {
        jcenter()
        mavenCentral()
        maven {
            url "s3://my-releases.example.com/maven2"
            credentials(AwsCredentials) {
                accessKey aws_accessid
                secretKey aws_accesskey
            }
        }
    }
}
```

Now you can simply depend on your new library as you usually do like so.

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    
    compile "com.example.myamazingproject:Magic:1.0.0"
    
    ...
    
    }
````
