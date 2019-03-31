---
layout: post
title: Revoke Json Web Tokens in Vert.x applications
date: 2017-12-26T22:19:00.000Z
categories: blog
---

In this article we'll explore how to manage pseudo sessions in our Vert.x application. For this purpose, the revoked tokens must be stored in a fast read-access datasource. We'll see that Redis is well suited for this task.
<br>This is how Facebook (among others) manages its revoked sessions on multiple devices. 

## Introduction

There are two mainly used ways of managing authentication in web applications: [cookies][cookies-url] and [jwt's][jwt-url].
<br>The main difference between them is *state*. This means that while cookies are stored in the browser and linked to a 
session persisted in a database on the backend-side; jwt's are completely stateless (no information relying the browser 
is persisted on the backend-side).

In the last years jwt's have found great popularity since they offer a very lightweight token that can be embedded in 
every http request. This is very useful when dealing with mobile applications.

## Materials

Before starting, I strongly encourage you to read [this introduction to Vert.x][introvertx-url] which goes through the main
topics in a vert.x application as well as [this vert.x jwt tutorial][vertxjwt-url].

All the code can be found here [https://github.com/underscorenico/vertx-jwt-revoke].

### API

The code we will use to show how to revoke jwts is a very basic CRUD Api, in which the document objects are musical instruments.
Here is how the code is structured:

    .
    ├── pom.xml
    ├── src
    │   ├── main
    │   │   └── java
    │   │       └── com.github.underscorenico.vertx.jwt.revoke
    │   │           └── controller
    │   │                   ControllerVerticle.java
    │   │           └── services
    │   │                   DatabaseVerticle.java
    │   │                   Instrument.java
    │   │                   InstrumentRepository.java
    │   │                   InstrumentRepositoryImpl.java
    │   │                   package-info.java
    │   │               MainVerticle
    │   └── test
    │       └── java
    │           └── com.github.underscorenico.vertx.jwt.revoke
    │                   MainVerticleTest.java
    
The API routes code is found in the ControllerVerticle verticle.

### JWTs

In order to protect the API, we'll use the [Vert.x jwt auth provider][vertxjwt-url]. Before protecting the API, we need
to create a keystore, which will contain the encryption keys. All this is well explained in the provider documentation.

The entry door to our API is the ```/login``` route. This route checks if the username and password provided in the body
of the request match against some user profile persisted in a database. Here we check the username and password against 
a hard-coded profile since we do not care about how the profiles are persisted.
<br> The route will then generate and sign a token and retrieve it in the response:

{% highlight java %} 
   /**
    * This method authenticates a user (hard-coded check to avoid database boilerplate) and returns a JWT.
    *
    * @param routingContext
    */
   private void login(RoutingContext routingContext) {
     // Get username from body
     String username = routingContext.getBodyAsJson().getString("username");
     // Get password from body
     String password = routingContext.getBodyAsJson().getString("password");
     // Hard-coded user / pass checking
     if (!username.equals("userTest") || !password.equals("passTest")) {
       routingContext.response()
         .setStatusCode(404)
         .putHeader("content-type", "application/json; charset=utf-8")
         .end(Json.encodePrettily("Bad username or password!"));
     } else {
       String token = provider.generateToken(new JsonObject().put("username", "userTest"), new JWTOptions());
       routingContext.response()
         .putHeader("content-type", "application/json; charset=utf-8")
         .end(Json.encodePrettily(new JsonObject().put("token", token)));
     }
   }
{% endhighlight %}

The retrieved token must be stored in the target web (or mobile) application and added in the ```Authorization``` 
key of the header of each incoming request:    

    Authorization: Bearer <token>

Once we have the JWT (on the client-sied), we can add it in the header so that the backend-side can check its validity.
The first check in the chain is given by a single handler that performs the following tasks:
* Automatically extract the token present in the ```Authorization``` key in the header
* Decrypt the token using the keystore persisted in the backend-side
* Check the validity and expiration date (unless stated otherwise in the options)

This handler can be applied to a set of routes (via regex) respecting the routes chaining:

{% highlight java %}
    router.route("/api/*").handler(JWTAuthHandler.create(provider, "/login"));
{% endhighlight %}

As we can see, the second parameter allows us to provide the routes that will not check for a token present in the headers.

[cookies-url]: https://en.wikipedia.org/wiki/HTTP_cookie
[jwt-url]: https://jwt.io/
[introvertx-url]: http://vertx.io/blog/posts/introduction-to-vertx.html
[vertxjwt-url]: http://vertx.io/docs/vertx-auth-jwt/java/
