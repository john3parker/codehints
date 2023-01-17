---
layout: post
title:  "Spring Security part 4 - Custom GrantedAuthorities"
date:   2023-01-14 11:00:00 -0600
categories: spring framework
published: true
---
From a Spring Security perspective a GrantedAuthority is the method to provide authorization for a user. Spring Security has its own implementation (code and database) for authorization, but what if your application already has its own role-based authorization? This article will dive into one approach which may work for your needs.

# Other Articles
* Architecture - [Spring Security Architecture](/spring/framework/2023/01/spring-security-overview.html)
* Part 1 - [Security configuration](/spring/framework/2023/01/spring-security-part1-config.html)
* Part 2 - [How to handle UserDetails gracefully](/spring/framework/2023/01/spring-security-part2-UserDetails.html)
* Part 3 - [UserDetails for OAuth authentication](/spring/framework/2023/01/spring-security-part3-OidcUser.html)
* Part 4 - [Handling GrantedAuthorities with custom roles/permissions](/spring/framework/2023/01/spring-security-part4-GrantedAuthority.html)
* Part 5 - [Security tags for Freemarker](/spring/framework/2023/01/spring-security-part5-security-tags.html)

If you're new to Spring, you should take a look at [Getting started with Spring Framework for Web Apps](/spring/framework/2023/01/09/spring-initializer)


# GrantedAuthority
A granted authority is a fairly simple object with a big role to play in the securing of your application. There is no hard-and-fast rule how to implement it other than to return a string object with `String getAuthority()`. There are 6 subclasses in Spring Security which implement [GrantAuthority](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/GrantedAuthority.html) interface. You can use one these of build your own.

You'll need to return an immutble list of granted authorities at some point in the authentication process. Refer to my [UserDetails](/spring/framework/2023/01/spring-security-part2-UserDetails.html) or [OidcUser](/spring/framework/2023/01/spring-security-part3-OidcUser.html) discussion on specific areas where you can return them during user/password or OAuth authentication. 

A super easy and effective way to return a GrantedAuthority from your custom database security roles, is to simply write a helper class and method for it.

The code below is an example wher I'm using a `SimpleGrantedAuthority` to create the roles and specific permissions for a user. The `getRoles()` method calls a custom repository to return a list of roles for a given user. 

Keep in mind the default prefix for roles in Spring Security is `ROLE_`, so be sure to prefix it in the authority if you want to use the default implementation evaluator for `hasRole`. 

For permissions, our custom solution requires a triplet object containing a `OBJECT:ID:PERMISSION`. An asterisk (*) is used to denote all identifiers for a given object. In this example, a user with the `USER` role will have permission to read and write their own object identifier. Users with the `ADMIN` role will have permission to read and write all users. 


{% highlight java %}
	public List<GrantedAuthority> getAuthorities(User user, Unit unit) {

		List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
		List<UserUnitRoleDTO> roles = getRoles(user);
		
		for(UserUnitRoleDTO role : roles) {			
			authorities.add(new SimpleGrantedAuthority("ROLE_" + role.getName().toUpperCase()));
			
			if (role.getName().equalsIgnoreCase("USER")) {
				authorities.add(new SimpleGrantedAuthority(String.format("%s:%s:%s", role.getName(), user.getId(), "READ")));
				authorities.add(new SimpleGrantedAuthority(String.format("%s:%s:%s", role.getName(), user.getId(), "WRITE")));
			}
			if (role.getName().equalsIgnoreCase("ADMIN")) {
				authorities.add(new SimpleGrantedAuthority("USER:*:READ"));				
				authorities.add(new SimpleGrantedAuthority("USER:*:WRITE"));
			}
		}
		
		return authorities;
	}

{% endhighlight %}

## PermissionEvaluator
Now that we have the authorities set up, let's write our custom permissions by implementing `PermissionEvaluator`. There are two methods you can override. In the example below, I've only overridden the second one since that's the only one I want to use. 

The second `hasPermission` method expects 4 arguments but you only need to send 3 in since the first one is the `Authentication` object of the principal. 

1. Serializable targetId - in our case this is a Long identifier.
1. String targetType - the type of object being security. e.g (User, Document, etc.)
1. Object permission - the permission you wish to grant. e.g. (read, write, etc.)

After that it's just your custom code deciding whether to return `true` or `false`. The code below lays out all the permission options I wanted to support and check tham against the granted authorities.

{% highlight java %}
public class CustomPermissionEvaluator implements PermissionEvaluator {

	Logger logger = LoggerFactory.getLogger(CustomPermissionEvaluator.class);
	
	@Override
	public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
		logger.info("hasPermission " + targetDomainObject + " " + permission);
		return false;
	}

	@Override
	public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
		//logger.info("hasPermission " + targetId + " " + targetType + " " + permission);

		String perm = String.format("%s:%s:%s", targetType, targetId.toString(), permission.toString());
		String permAllIds = String.format("%s:*:%s", targetType, permission.toString());
		String permAllRoles = String.format("*:%s:%s", targetId.toString(), permission.toString());
		String permAllPerms = String.format("%s:%s:*", targetType, targetId.toString());
		String permAll = "*:*:*";
		
		logger.info("hasPermission checking for " + perm);
		for(GrantedAuthority a : authentication.getAuthorities()) {
			logger.info("authority=" + a.getAuthority());
			if (a.getAuthority().equalsIgnoreCase(perm) 
					|| a.getAuthority().equalsIgnoreCase(permAllIds) 
					|| a.getAuthority().equalsIgnoreCase(permAllRoles) 
					|| a.getAuthority().equalsIgnoreCase(permAllPerms) 
					|| a.getAuthority().equalsIgnoreCase(permAll)) {
				logger.info("matched " + a.getAuthority());
				return true;
			}
		}
		
		return false;
	}

}
{% endhighlight %}


## Implement in a controller
This is how it looks to implement in your web `Controllers`. 

{% highlight java %}
	@PreAuthorize(value = "hasPermission(#id, 'USER','READ')")
	@GetMapping(value = "/user/profile/{id}")
	public String viewUser(@PathVariable Long id, Model model) {
		...				
		return "user/profile";
	}
{% endhighlight %}