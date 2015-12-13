---
layout: post
title:  "Using RxJava to Implement Asynchronous Loops"
date:   2015-12-13
categories: android rxjava
tags: rxjava, android
author: Brett Cherrington
---

Recently I was working on a project that required me to replace a while loop that contained a blocking method. This post shows how this can be done using RxJava.

Firstly lets look at the original loop.

```java

  final List<String> supportedApps = getListForSelection();

  String candidate = null;
  for(String selectedCandiate : supportedApps)
    try {
      selectApplication(selectedCandidate);
      userSelectAccount();
      candidate = selectedCandidate;
    } catch(Exception e) {
      // failed to select this application move onto the next
    }
  }

  // handle candidate or null!
}

```

As you can see the loop above goes through a list of `supportedApps` and attempts
to select one. The problem lies in the`selectApplication(...)` and
`userSelectAccount()` methods which are both blocking methods that wait for some
kind of asynchronous input.

One of these methods perfoms interaction with some hardware the second needs to wait for some
input from the user. To make it worse the only way we know these methods aren't
successful is they throw some kind of exception. Potentially of course we could
use a listener pattern but we have two things to listen to asynchronously which
would result in *callback hell!*

## How can RxJava help?

Firstly we need to start thinking in Obserables (or streams). The first step towards
this is to make the list of `supportedApps` an `Observable`. This is pretty straight
forward like so.

```java
    final List<String> supportedApps = getListForSelection();
    final Observable<String> observableSupportedApps = Observable.from(supportedApps);
```

Now that we have our candidates as a stream we can subscribe to them and perform the
first validation method from above.

```java
    observableSupportedApps.takeWhile(new Func1<String, Boolean>() {
      @Override
      public Boolean call(String candidate) {
        try {
          return selectApplication(candidate);
        } catch (Exception e) {
          return false;
        }
      }
    }).subscribe(...)

```
Above I am using the `takeWhile` method which will take items from the stream as long
as they return true ignoring any that return false. Here we handle the exception
and return appropriately.

Next we need to figure out how to perform our second step and stop when we get a valid
response to our `userSelectAccount()` method. To do this we can simply create another
observable which will emit when the user has selected.

```java

    final PublishSubject<Integer> stop = PublishSubject.create();

    final List<String> supportedApps = getListForSelection();
    final Observable<String> observableSupportedApps = Observable.from(supportedApps);

    // go through candidates until we find one that is supported by terminal and is valid
    observableSupportedApps.takeWhile(new Func1<String, Boolean>() {
      @Override
      public Boolean call(String candidate) {
        try {
          return selectApplication(candidate);
        } catch (Exception e) {
          return false;
        }
      }
    })
    .takeUntil(stop)
    .subscribe(new Action1<String>() {
      @Override
      public void call(String candidate) {
        try {
          userSelectAccount();

          // here we have selected our application finally
          // all is well so we want to stop
          stop.onNext(1);

        } catch (Exception e) {
          resultPublishSubject.onError(e);
        }
      }
    });
```

So here we have ensured the methods are called in order and we can use the normal
RxJava methods to run the subscription/observation on whatever threads we choose.

I'm sure there are many other ways to achieve this and it is quite specific to our
application. However, this method helped with our application and I hope someone
out there also finds it useful.
