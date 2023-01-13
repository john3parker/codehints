---
layout: post
title:  "Spring Security part 1"
date:   2023-01-10 11:54:21 -0600
categories: spring framework
published: false
---
Spring Security is a well made and powerful authentication and authorization framework used for securing Spring-based applications. The underlying value is how quickly and easily it can be configured and extended to meet most business requirements. However, because it's so configurable it can be daunting to set up and understand how to quickly get it running for your application. This 5 part series will take you through a real-world example from start to finish. 

# Overview
* [Spring Security](/spring/framework/2023/01/spring-security)
* Part 1 - Security configuration
* Part 2 - [How to handle UserDetails gracefully](/spring/framework/2023/01/spring-security-part2)
* Part 3 - Setting up User/Pass and OAuth authentication
* Part 4 - Handling GrantedAuthorities with custom roles/permissions.
* Part 5 - Security tags for Freemarker

# Prerequistes
This series on Spring Security uses `Spring 6`. Enable Spring Security by adding the following dependency into your `pom.xml`. 

{% highlight yaml %}
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
{% endhighlight %}

### HomeController
We're going to need something to secure. Create a simple web controller. 

{% highlight java %}
@Controller
public class HomeController {

	@GetMapping("/")
	public ResponseEntity<String> home(Model model) {
	
		return new ResponseEntity<String>("Hello World", HttpStatus.OK);
	}
	
	@GetMapping(value = "/user/view", produces = MediaType.TEXT_PLAIN_VALUE)
	public ResponseEntity<String> userTest(Model model) {
	
		return new ResponseEntity<String>("Hello User", HttpStatus.OK);
	}	
}

{% endhighlight %}

# Security Configuration

Create a new base class and give it meaningful name. You'll need to annotate it with `@Configuration` so Spring knows this class contains configuration for your application. 

You'll need to annotate it with `@EnableWebSecurity` to have the Spring Security configuration defined by exposing a `SecurityFilterChain` bean. 

{% highlight java %}
@Configuration
@EnableWebSecurity
public class SecurityConfig  {

}
{% endhighlight %}

Immediately after bringing in the Spring Security dependencies, without any configuration, you'll notice your web application forward you to a login page for all requests. With a little configuration we can allow some requests to go through without authentication.

![Login Page ](/assets/2023/spring-security/default-login-page.png){:.img-fluid}

When we added `@EnableWebSecurity`, it automatically enables [CSRF](https://owasp.org/www-community/attacks/csrf) protection. This is activated by default when using EnableWebSecurity's default constructor. This also populates the `CsrfFilter`. 

You can disable CSRF with this code:
{% highlight java %}
  http.csrf().disable()
{% endhighlight %}

### Basic configuration
Let's configure the `/` mapping so its avaiable to everyone and configure the `/user` mapping so only authenticated users can view it. The `authorizeHttpRequests()` method enables the `requestMatchers` which allows us to easily configure each mapping. Keep in mind, order will matter here. When a filter matches it will stop processing the remaining ones. If you have `requestMatchers("/**")` above the user mapping, it will allow everyone to see both URLs. 

{% highlight java %}
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    	
    	http
    		.authorizeHttpRequests()
    		.requestMatchers("/user/**").authenticated()
    		.requestMatchers("/**").permitAll()
    		.and()
    		.formLogin();
    	
    	return http.build();
    }
{% endhighlight %}

### Set up users & in-memory database
To keep things simple we will use an in-memory database and clear text password. In a production environment you'll want to use a [PasswordEncoder](https://docs.spring.io/spring-security/site/docs/6.0.1/api/org/springframework/security/crypto/password/PasswordEncoder.html) that encrypts the password.

{% highlight java %}
    @Bean
    public static PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    
    @Bean
    public UserDetailsService users() {
    	
    	UserDetails user = User.builder()
    		.username("user")
    		.password("user")
    		.roles("USER")
    		.build();
    	UserDetails admin = User.builder()
    		.username("admin")
    		.password("admin")
    		.roles("USER", "ADMIN")
    		.build();
    	return new InMemoryUserDetailsManager(user, admin, user2);
    }
    {% endhighlight %}