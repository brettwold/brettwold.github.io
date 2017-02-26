---
layout: post
title:  "Angular2 Http Authentication Interceptor"
subtitle: ""
date:   2017-02-26
categories: angular2 javascript typescript ionic2
tags: angular2 ionic2
author: Brett Cherrington
---

This post shows how you can write an `Interceptor` for the Angular2 Http class
and use it to help when accessing an authenticated API. The interceptor can be
used to catch any call to an API that is un-authenticated (i.e. returns a 401)
and then act appropriately to authenticate the application and then repeat the
original call.

The code below shows the main interceptor class which extends from the Angular2
Http class. The main aim of the class is to intercept any HTTP calls that are sent
and if a 401 response is returned then obtain a new authentication token for the
API and repeat the original call. This should be done without any impact to the
code using the interceptor, in other words the interceptor should behave as per
the standard Http class.

<script src="https://gist.github.com/brettwold/f94ec2de1355dab62619052c4b38e86e.js"></script>

Diving into the code above you can see that the main `request` method of the Http
is overridden so that a token can be added to the headers of every request sent
by the class. The `requestWithToken` method adds a standard HTTP `Authorization`
header item to every request. This header will contain a token that has been
previously obtained by the application. If no token has yet been obtained then
by default this implementation will throw an Error, but this can be overridden
by changing the configuration parameters passed to this interceptor on
construction (more about that later). As you can also see the token is obtained
by whatever `Promise<string>` the `getToken()` method returns which had to be
provided by the concrete implemetation of this abstract base class.

Also as you can see all the standard HTTP method calls are overridden and make
a call to the `intercept()` method. This method add a catch observer to catch
any HTTP call this has a response code of 401. If a 401 is observed then a call
is made to the `refreshToken()` method which should make an attempt to obtain a
new token. Also the `intercept()` method takes care of storing the original HTTP
request details so that once an authorisation token has been obtained the original
call cen be sent automatically and the original subscriber to the call is completely
unaware anything different has happened.

As you can see this class is abstract so in order to use it you have to extend it
with your own concrete implementation. This implementation must provide the
`getToken` , `saveToken` and `refreshToken` methods. An example implementation
is shown below. This example is using Ionic2 and it's Storage class to store
an authentication token but the class should work for any Angular2 framework.

<script src="https://gist.github.com/brettwold/711e23cf26c18cbb0054aa3a5985223f.js"></script>

In the `refreshToken` method above you can see that a token is obtained by calling
posting to an authenticate URL. In order to avoid being trapped in a 401 loop
usually a call to this type of authenticate URL does not need to be authenticated
itself by means of an Authorization header. This would be true for OAuth and
authentication URLs where you may wish to obtain a JWT style token. Therefore
when calling this method we need to skip the intercept part of our Http
implementation. This is why the interceptor class adds an extra boolean flag to
all HTTP methods, so that the intercept can be skipped.

Finally we need to make sure that our `HttpAuthInterceptor` is used throughout
our application instead of the normal Angular2 Http class. To do this we simply
need to define it as a new provider in the @NgModule definition.


```typescript

export function getHttpAuth(backend: ConnectionBackend, defaultOptions: RequestOptions, storage: Storage) {
  return new HttpAuth(backend, defaultOptions, storage);
}

@NgModule({
  ...

  providers: [
    {
      provide: Http,
      useFactory: getHttpAuth,
      deps: [XHRBackend, RequestOptions, EnvService, Storage]
    },
    ...
  ]

  ...
})
```

By providing a different implementation of the Http class but sticking to the
same interface means we can now use the class as we normally would for HTTP calls.
If the client application is currently un-authorised or the token has expired
our HttpAuthInterceptor implementation will take care of refreshing the token and
repeating the original call for us. So an example of calling an API URL would be
a standard call using the normal Angular2 Http class e.g.

```typescript
return this.http.get(this.api_url)
                .map(res => this.extractData(res))
                .catch(this.handleError);
```

## Summary

This post shows how to implement an interceptor extending the standard Angular2 
Http class, that is capable of intercepting any requests ended with 401 failures
and automatically refreshing an authentication token.

This examples above are taken from a real Ionic2 application connecting to a
bespoke API but this class should be capable of being used in conjunction with
many modern APIs requiring authentication including OAuth or JWT style APIs.
