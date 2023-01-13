---
layout: post
title:  "Spring Security"
date:   2023-01-10 11:54:21 -0600
categories: spring framework
---
Spring Security is a well made and powerful authentication and authorization framework used for securing Spring-based applications. The underlying value is how quickly and easily it can be configured and extended to meet most business requirements. However, because it's so configurable it can be daunting to set up and understand how to quickly get it running for your application. This 5 part series will take you through a real-world example from start to finish. 

# Table of Contents
* Part 1 - [Security configuration](/spring/framework/2023/01/spring-security)
* Part 2 - How to handle UserDetails gracefully
* Part 3 - Setting up User/Pass and OAuth authentication
* Part 4 - Handling GrantedAuthorities with custom roles/permissions.
* Part 5 - Security tags for Freemarker

# Overview
If you're new to Spring and you haven't set up a projet yet then take a look at [Getting started with Spring Framework for Web Apps](/spring/framework/2023/01/spring-initializer). 

Spring Security does a lot of work with a very little configuration. It's important to have a solid understanding how its architected with regard to web servlet based applications. Spring Security is based on the use of servlet filter chains. It injects itself into the servlet chain to determine if a request needs a user to be authentication, whether the user has authorization, etc. 

Servlet filters intercept a request from a client and can inspect or change the request before it gets to the servlet engine. Here's a high level architecture.

{% highlight ruby %}
Client => filter0 => filter1 => filter2 => filterN => Servlet
{% endhighlight %}

Spring Security injects its own `FilterChainProxy` into the servlet's filter chain which then executes a `SecurityFilterChain`. The security filter chain contains as many `Security Filters` as needed based on configuration.

{% highlight ruby %}
Client => filter0 => DelegatingFilterProxy =|                   ---> filter2 => filterN => Servlet
        v-----------------------------------|                   |
        FilterChainProxy => SecurityFilterChain -|              |  
        v----------------------------------------|              |  
        securityFilter0 => securityFilter1 => securityFilterN --^

{% endhighlight %}

Spring Security has a specific order it executes its filters in. From a security perspective, the order of filters matters. It typically is not necessary to understand or know the ordering of Spring Security filter instances. However, there are times that it is beneficial to know the ordering.

* ForceEagerSessionCreationFilter
* ChannelProcessingFilter
* WebAsyncManagerIntegrationFilter
* SecurityContextPersistenceFilter
* HeaderWriterFilter
* CorsFilter
* CsrfFilter
* LogoutFilter
* OAuth2AuthorizationRequestRedirectFilter
* Saml2WebSsoAuthenticationRequestFilter
* X509AuthenticationFilter
* AbstractPreAuthenticatedProcessingFilter
* CasAuthenticationFilter
* OAuth2LoginAuthenticationFilter
* Saml2WebSsoAuthenticationFilter
* UsernamePasswordAuthenticationFilter
* DefaultLoginPageGeneratingFilter
* DefaultLogoutPageGeneratingFilter
* ConcurrentSessionFilter
* DigestAuthenticationFilter
* BearerTokenAuthenticationFilter
* BasicAuthenticationFilter
* RequestCacheAwareFilter
* SecurityContextHolderAwareRequestFilter
* JaasApiIntegrationFilter
* RememberMeAuthenticationFilter
* AnonymousAuthenticationFilter
* OAuth2AuthorizationCodeGrantFilter
* SessionManagementFilter
* ExceptionTranslationFilter
* FilterSecurityInterceptor
* SwitchUserFilter

In part 1, we'll be taking advantage of the `SecurityFilterChain` where we'll configure and see the filters in action.

That's an overview of Spring Security filter architecture. For a more comprehensive review refer to the [Spring Security documentation](https://docs.spring.io/spring-security/reference/servlet/architecture.html). 
