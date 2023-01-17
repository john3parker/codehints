---
layout: post
title:  "Spring Security part 2 - UserDetails"
date:   2023-01-12 11:00:00 -0600
categories: spring framework
published: true
---
We've all seen the infamous in-memory database with UserDetails as the base, but that's just a high level example and isn't useful in a production application. In this segment, I'm going to show you how to turn `UserDetails` into a useful security object.

# Other Articles
* Architecture - [Spring Security Architecture](/spring/framework/2023/01/spring-security-overview.html)
* Part 1 - [Security configuration](/spring/framework/2023/01/spring-security-part1-config.html)
* Part 2 - [How to handle UserDetails gracefully](/spring/framework/2023/01/spring-security-part2-UserDetails.html)
* Part 3 - [UserDetails for OAuth authentication](/spring/framework/2023/01/spring-security-part3-OidcUser.html)
* Part 4 - [Handling GrantedAuthorities with custom roles/permissions](/spring/framework/2023/01/spring-security-part4-GrantedAuthority.html)
* Part 5 - [Security tags for Freemarker](/spring/framework/2023/01/spring-security-part5-security-tags.html)

If you're new to Spring, you should take a look at [Getting started with Spring Framework for Web Apps](/spring/framework/2023/01/09/spring-initializer)


# UserDetails
You see and hear quite a bit about [UserDetails](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetails.html) but very few ways to actually implement it in production code. UserDetails is an interface you can add to your own `User` object, but in order to get access to it you need to also implement `UserDetailsService` as a component bean. 

Spring Security will trigger this bean's `loadUserByUsername` method after the user submits user/password but before it's authenticated via the password. You return your own `UserDetails` object and allow spring security to determine if the user credentials match. 

{% highlight java %}
@Component
public class MyUserDetailsService implements UserDetailsService {

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		// TODO Auto-generated method stub
		return null;
	}

}
{% endhighlight %}


To implement this you'll want to implement the `UserDetails` interface on your own `UserContext` object. It's recommended that you don't implement this interface on your entity object since it's mutable, so it's better to create your own immutable object. 

{% highlight java %}
public class UserContext implements UserDetails {

    private final User user; // this is the entity bean
    ...
	private final List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();

	public UserContext() {}
	public UserContext(User user, List<GrantedAuthority> authorities) { 
		this.user = user;
		this.authorities = authorities;
	}

    ...
}
{% endhighlight %}

Now just implement your business logic to find the `User` in your database, set up the granted authorities for access and return your new `UserContext`. 

{% highlight java %}
@Component
public class MyUserDetailsService implements UserDetailsService {

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        User user = userRepository.findByUsername(username);
        // check for user not found
        ...
        List<GrantedAuthority> grantedAuthoritites = roleRepository.findRoles(user.getId());
        UserContext userContext = new UserContext(user, grantedAuthoritites);

        return userContext;
	}

}
{% endhighlight %}

You now have your own `UserContext` on the SecurityContext which is much more useful and relevant to your application. Below I've created a helper class to quickly get the `UserContext` from Spring Security's `SecurityContextHolder`. 

{% highlight java %}
public class SecurityUtils {
	private SecurityUtils() {}
		
	public static UserContext getUserContext() {
		Authentication authentication = getAuthentication();
		if (authentication.isAuthenticated()) {
			if (authentication.getPrincipal() instanceof UserContext) {
				UserContext loggedInUserContext = (UserContext) authentication.getPrincipal();
				return loggedInUserContext;
			}
		}
		return null;
	}

	public static Authentication getAuthentication() {
		SecurityContext context = SecurityContextHolder.getContext();		
		Authentication authentication = context.getAuthentication();
		if (null == authentication || (authentication.getPrincipal() instanceof String)) {
			authentication = new UsernamePasswordAuthenticationToken(null, null);
			authentication.setAuthenticated(false);
		}
		return authentication;
	}
	
	public static boolean isAuthenticated() {
		return getAuthentication().isAuthenticated();
	}
}
{% endhighlight %}