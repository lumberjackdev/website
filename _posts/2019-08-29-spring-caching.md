---
layout: post
title: "How To: Spring Caching with Two Cache Providers"
tags: [how-to, spring]
summary: "Learn how to setup Spring Caching with two separate cache providers (redis and ehcache)"
---

### The Problem
In most cases with a Spring Boot app, we'll likely only ever have one cache provider. Spring Boot provides [quite a few](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html#boot-features-caching-provider) cache providers out of the box. But what happens if we want to use multiple providers in our application? We'll be exploring a solution to this problem in today's post. 

_Note: This post assumes working knowledge of spring boot and spring caching_

### Setting Things Up
The first thing we'll do is setup Ehcache using JCache. To do this we'll need to add a few dependencies to the build file for spring caching, ehcache, and the java cache api:

<pre><code class="language-kotlin">implementation("org.springframework.boot:spring-boot-starter-cache")
implementation("org.ehcache:ehcache:3.8.0")
implementation("javax.cache:cache-api")</code></pre>

Next, we'll let spring know where to find our jcache config in the application.yml:

<pre><code class="language-yaml">spring:
  cache:
    jcache:
      config: classpath:ehcache.xml</code></pre>

Lastly, we'll enable caching and setup a small class that uses our configured cache:

<pre><code class="language-kotlin">@Configuration
@EnableCaching 
class CacheConfig

// Found in BookService.kt
@Cacheable(cacheNames = ["books"])
fun getBooks(author: String) = books.getOrDefault(author, emptyList())
</code></pre>

Now that we've got ehcache configured, let's set up redis. Like before, we'll start off by modifying the dependencies:

<pre><code class="language-kotlin">implementation("org.springframework.boot:spring-boot-starter-data-redis")</code></pre>

By including this starter, Spring Boot will go ahead and configure all of the necessary components in order to use redis. With our application in this state, if we `bootRun` and check out the auto-configuration report (http://localhost:8080/actuator/conditions) we'll see something like this:

<pre><code class="language-javascript">
{
  "RedisCacheConfiguration": {
    "notMatched": [
      {
        "condition": "OnBeanCondition",
        "message": "@ConditionalOnMissingBean (types: org.springframework.cache.CacheManager; SearchStrategy: all) found beans of type 'org.springframework.cache.CacheManager' cacheManager"
      }
    ]
  }
}
</code></pre>

Unfortunately, this means that Spring Boot won't be autoconfiguring our RedisCacheManager bean for us since it already sees a Bean of type `CacheManager` from the Ehcache/JCache configuration. 

### The Solution
So how do we fix this? Well this is one of those cases where we need to take over the configuration from Spring Boot. To do this we'll create a more complex `CacheConfig` class that declares two `CacheManager` Beans:

<pre><code class="language-kotlin">@Bean
@Primary
override fun cacheManager(): CacheManager =
        RedisCacheManager.builder(redisConnectionFactory).build()

@Bean
fun jCacheCacheManager(): JCacheCacheManager {
    val cachingProvider = Caching.getCachingProvider()
    val configLocation = cacheProperties().resolveConfigLocation(cacheProperties().jcache.config)
    val cacheManager = cachingProvider.getCacheManager(configLocation.uri, classLoader)
    jCacheManagerCustomizers.orderedStream().forEach { it.customize(cacheManager) }
    return JCacheCacheManager(cacheManager)
}
</code></pre>

Now that we have the two cache managers defined, we'll update the `BookService` to include a new cached method:

<pre><code class="language-kotlin">@Cacheable(cacheNames = ["books"], cacheManager = "jCacheCacheManager")
fun getBooks(author: String) = books.getOrDefault(author, emptyList())

@Cacheable(cacheNames = ["authors"], key = "'all-authors'")
fun getAuthors() = listOf(books.keys.toTypedArray())
</code></pre>

Notice that `getBooks` specifies its `CacheManager` while `getAuthors` does not. This means that `getBooks` will use the `jCacheCacheManager` Bean while `getAuthors` will use the Primary `CacheManager` Bean which we defined to be a `RedisCacheManager`. We can see this in action in the [tests](https://github.com/lumberjackdev/spring-caching-with-multiple-providers/blob/master/src/test/kotlin/com/lumberjackdev/caching/demo/service/BookServiceTests.kt). And there we have it, a working Spring Boot app with two cache providers!

### Things to Look Out For
* If we're migrating from previously having only one provider to multiple providers, then we may have specified the application property `spring.cache.type` in order to force the usage of a specific cache type. When combining multiple providers, we'll want to avoid that. 
* We're now owning the configuration of our `CacheManager` Beans, so this means some of the behavior might differ from the vanilla autoconfiguration. We'll especially want to be careful if we rely on [Customizers](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/cache/CacheManagerCustomizer.html) because we'll now need to manually apply those.
* We'll probably want to make sure that whichever provider we would like to have as the default is the Primary `CacheManager` Bean to save ourselves the effort of having to constantly specify the manager. This way various cached items need to opt into the secondary manager.

Check out the example code on [github](https://github.com/lumberjackdev/spring-caching-with-multiple-providers) to see the complete example!

Thanks for reading!
