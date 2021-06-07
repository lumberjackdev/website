---
layout: post
title: "How To: Spring Security with SSO using Oauth2"
tags: [how-to, spring, sso, oauth2]
summary: "Learn how to setup Spring Security with an SSO provider using Oauth2"
---

### The Problem
Let's face it, authentication and user identity management isn't exactly easy. Also, I think I can safely say that for most developers it's not the most fun thing either. Building our own system can add maintenance costs and delay the path to production. So how can we solve this? Single Sign-On (SSO) of course! In this post, we secure a spring boot application using an SSO provider (Google in this case).

### Background
Before we get into the actual code, let's talk a little bit more about what SSO actually is. To do that, we first need to get into what [Oauth2](https://oauth.net/2/) is since we often hear the two terms used together. Oauth is the underlying standard used by providers when clients (our application) request access to users' information on their behalf _without_ using their password, but rather by sharing tokens.

At this point, you might be thinking that Oauth and SSO are the same thing, but there are differences. Specifically, SSO is using one set of credentials against multiple applications. Oauth doesn't necessarily need to be used for login, but that's what it is being used for here. It's also worth pointing out that there are other ways of accomplishing SSO, for example [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language).

### Steps
Spring Boot provides support for [several](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-security.html#boot-features-security-oauth2-common-providers) oauth2 providers, but for the purpose of this tutorial let's use Google.

#### 1. Get a Client ID and Secret

Before we get into the code, we need to register our application with Google. We can do that [here](https://developers.google.com/identity/sign-in/web/sign-in#before_you_begin). For the application name we can provide what we'd like. One thing to remember, however, is that the redirect uri should be `http://localhost:8080/login/oauth2/code/google` for local development. In a deployed environment we would need to do something besides localhost, but the rest of the uri would look the same. Once everything is filled out, we are provided a client ID and secret, this is the only time we are given them so either copy the values somewhere else or download the provided file. 

#### 2. Configure Dependencies

In order to get this sample working, we need to setup the dependencies:

<pre><code class="language-kotlin">implementation("org.springframework.boot:spring-boot-starter-web")
implementation("org.springframework.boot:spring-boot-starter-actuator")
implementation("org.springframework.boot:spring-boot-starter-oauth2-client")
implementation("org.springframework.boot:spring-boot-starter-security")
</code></pre>

Here we're adding the web, actuator, security, and oauth2 client starters. This gives the application everything we need to get going. We can also use the [Spring Initializr](https://start.spring.io/) with web, actuator, oauth2, and security options. 

#### 3. Configure the Oauth2 Client

Using the client ID and secret from earlier, we next configure the client in our application configuration
<pre><code class="language-yaml"># application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            clientId: ${CLIENT_ID}
            clientSecret: ${CLIENT_SECRET}
            redirectUri: 'http://localhost:8080/login/oauth2/code/google'
</code></pre>

#### 4. Configure Spring Security with Oauth2 Login

<pre><code class="language-kotlin">@Configuration
@EnableWebSecurity
class SecurityConfig : WebSecurityConfigurerAdapter() {
    override fun configure(http: HttpSecurity?) {
        http!!.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .oauth2Login()
    }
}
</code></pre>

In this security configuration, we're going to enforce authentication on any request to the application with an oauth2 login. If you've ever set up a similar configuration the only new piece should be the call to `oauth2Login()`.

#### 5. Try it out

Now that we have everything configured, go ahead and run the application through gradle. When we try to access something (like one of the actuator endpoints: `http://localhost:8080/actuator/health`), we are taken to the Google login page and then redirected back where we were trying to go.

#### 6. Testing
Unfortunately with SSO unless we're able to create a test account with the provider it can be difficult to write tests. At this point, the best thing I can recommend is having a separate Security Configuration for tests that relies on basic authentication instead. Keep in mind that we should attempt to make that the only difference for our test configuration.

### Things to Look Out For

While SSO is a convenient way of performing using authentication, there are a couple of things to watch out for.
 * We're relinquishing control of data. Unless we plan on persisting some of the data we receive from the provider, we won't have much control over user data. If the provider changes something, our application needs to react. 
 * If the SSO provider has issues, then our users might have issues. This shouldn't really be much of an issue with a common provider, but it's something to keep in mind.

### What's next?
I know not every team has the luxury of using an existing SSO provider like Google. So one of the next tutorials I'm considering doing is for setting up our own SSO provider using Spring Security's [Authorization Server](https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/html/boot-features-security-oauth2-authorization-server.html). That way your team or organization can setup their own provider for use across multiple applications.

Check out the source code [here](https://github.com/lumberjackdev/spring-boot-google-sso)! I hope this tutorial can help you and your team, thanks for reading!