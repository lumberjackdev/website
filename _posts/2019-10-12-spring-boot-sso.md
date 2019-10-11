---
layout: post
title: "How To: Spring Security with SSO through Oauth2"
tags: [how-to, spring, sso, oauth2]
summary: "Learn how to setup Spring Security with an SSO provider using Oauth2"
published: false
---

### The Problem
Let's face it, authentication and user identity management isn't exactly easy. And I think I can safely say that for most developers it's not the most fun thing either. Building your own system can add maintenance costs and delay the path to production. So how can we solve this? Single Sign-On (SSO) of course! In this post, I'll walk you through securing a spring boot application using an SSO provider (Google in this case).

### Background
Before we get into the actual code, let's talk a little bit more about what SSO actually is. To do that, we'll first need to get into what Oauth2 is since you'll often hear the two terms used together a lot. 
- What is SSO
- What is Oauth2

- Why they're a good solution

### Steps
Spring Boot provides support for [several](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-security.html#boot-features-security-oauth2-common-providers) oauth2 providers, but for the purpose of this tutorial we'll be using Google.

1. Get a Client ID and Secret:
Before we get into the code, we'll need to register our application with Google. We can do that [here](https://developers.google.com/identity/sign-in/web/sign-in#before_you_begin). For the application name you can provide what you'd like. One thing to remember, however, is that the redirect uri should be `http://localhost:8080/login/oauth2/code/google`. In a deployed environment you'd need to do something besides localhost, but the rest of the uri would look the same. Once you've filled everything out, you'll be provided a client ID and secret, this is the only time you'll be given them so either copy the values somewhere else or download the provided file. 

1. Dependencies
In order to get this sample working, we'll need to setup the dependencies:

<pre><code style="language-kotlin">implementation("org.springframework.boot:spring-boot-starter-web")
implementation("org.springframework.boot:spring-boot-starter-actuator")
implementation("org.springframework.boot:spring-boot-starter-oauth2-client")
implementation("org.springframework.boot:spring-boot-starter-security")
</code></pre>

Here we're adding the web, actuator, security, and oauth2 client starters. This will give the application everything we need to get going.

1. Configure the Oauth2 Client
Using the client ID and secret from earlier, we'll next configure the client in our application configuration
<pre><code style="language-yaml"># application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            clientId: ${CLIENT_ID:}
            clientSecret: ${CLIENT_SECRET:}
            redirectUri: 'http://localhost:8080/login/oauth2/code/google'
</code></pre>
- Add yaml properties
-> Include snippet with obscuring clientId and Secret

- Security Config
-> Code snippet of SecurityConfig.kt

- Test
-> Code snippet of LoginTests.kt

### Things to Look Out For
-> You're relinquishing control of data
-> If the SSO provider has issues, then your users will have issues

### What's next?
I know not every team has the luxury of using an existing SSO provider like Google. So one of the next tutorials I'll be doing will be for setting up our own SSO provider using Spring Security's [Authorization Server](https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/html/boot-features-security-oauth2-authorization-server.html). That way your team or organization can setup their own provider for use across multiple applications.

Check out the source code [here](https://github.com/lumberjackdev/spring-boot-google-sso) if you're interested! I hope this tutorial can help you and your team, thanks for reading!