---
layout: post
title:  "Spring Security part 3 - OidcUser"
date:   2023-01-13 11:00:00 -0600
categories: spring framework
published: true
---
In the last article, I showed you how to return your own `UserDetails` object via a bean interface for username/password authentication. But what if you want to use OAuth authentication too? Spring Security works differently with OAuth but with a little engineering we can bring it all together into a cohesive solution.

# Other Articles
* Architecture - [Spring Security Architecture](/spring/framework/2023/01/spring-security-overview.html)
* Part 1 - [Security configuration](/spring/framework/2023/01/spring-security-part1-config.html)
* Part 2 - [How to handle UserDetails gracefully](/spring/framework/2023/01/spring-security-part2-UserDetails.html)
* Part 3 - [UserDetails for OAuth authentication](/spring/framework/2023/01/spring-security-part3-OidcUser.html)
* Part 4 - [Handling GrantedAuthorities with custom roles/permissions](/spring/framework/2023/01/spring-security-part4-GrantedAuthority.html)
* Part 5 - [Security tags for Freemarker](/spring/framework/2023/01/spring-security-part5-security-tags.html)

If you're new to Spring, you should take a look at [Getting started with Spring Framework for Web Apps](/spring/framework/2023/01/09/spring-initializer)


# OidcUser
You've already learned about `UserDetails` but do you know about its cousin [OidcUser](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/core/oidc/user/OidcUser.html?

Oidc is a representation of a user Principal that is registered with an OpenID Connect 1.0 Provider. An OidcUser contains claims about the authentication of the End-User. The claims are aggregated from the OidcIdToken and the OidcUserInfo (if available).

If your application is authenticating via OAuth you'll need to implement the `OAuth2UserService` to gain access. Bring in the OAuth libraries with this Maven dependency.

{% highlight java %}
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-oauth2-client</artifactId>
</dependency>
{% endhighlight %}

Spring Security will trigger this bean's `loadUser` method after the user is authenticated with the OAuth service. You return your own `OidcUser` object and granted authorities.  

{% highlight java %}
import org.springframework.security.oauth2.client.oidc.userinfo.OidcUserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.oidc.user.OidcUser;

@Component
public class MyOAuth2UserService implements OAuth2UserService<OidcUserRequest, OidcUser> {

	@Override
	public OidcUser loadUser(OidcUserRequest userRequest) throws OAuth2AuthenticationException {
		// TODO: override details
	}
}
{% endhighlight %}

To implement this you'll want to implement the `OidcUser` interface on your own `UserContext` object. It's recommended that you don't implement this interface on your entity object since it's mutable, so it's better to create your own immutable object. We're going to re-use the same object we created to support user/password authentication by implementing `UserDetails` and `OidcUser`.

{% highlight java %}
public class UserContext implements UserDetails, OidcUser {

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

Now implement your business logic to find the `User` in your database, set up the granted authorities for access and return your new `UserContext`. You may also want to take the time to investigate what the OAuth service returned in `OidcUserRequest`. 

{% highlight java %}
@Component
public class MyOAuth2UserService implements UserDetailsService {

	@Override
	public OidcUser loadUser(OidcUserRequest userRequest) throws OAuth2AuthenticationException {

		OidcUserInfo userInfo = OidcUserInfo.builder()
		.email(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.EMAIL))
		.familyName(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.FAMILY_NAME))
		.givenName(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.GIVEN_NAME))
		.gender(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.GENDER))
		.birthdate(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.BIRTHDATE))
		.phoneNumber(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.PHONE_NUMBER))
		.picture(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.PICTURE))
		.profile(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.PROFILE))
		.address(getMapValue(userRequest.getIdToken().getClaims(), StandardClaimNames.ADDRESS))
		.build();

        User user = userRepository.findByUsername(userInfo.getEmail());
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