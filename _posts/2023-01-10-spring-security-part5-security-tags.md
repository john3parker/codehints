---
layout: post
title:  "Spring Security part 5 - Freemarker Security Tags"
date:   2023-01-15 11:00:00 -0600
categories: spring framework
published: true
---
In this series of Spring Security articles, I've walked you through the configuration, authentication and authorization cycles. In this final article, I'm going to show you how to set up security tags in Free Marker. The concepts I'm going to show you work equally well in JSP, Thymeleaf and Tiles. 

# Other Articles
* Architecture - [Spring Security Architecture](/spring/framework/2023/01/spring-security-overview.html)
* Part 1 - [Security configuration](/spring/framework/2023/01/spring-security-part1-config.html)
* Part 2 - [How to handle UserDetails gracefully](/spring/framework/2023/01/spring-security-part2-UserDetails.html)
* Part 3 - [UserDetails for OAuth authentication](/spring/framework/2023/01/spring-security-part3-OidcUser.html)
* Part 4 - [Handling GrantedAuthorities with custom roles/permissions](/spring/framework/2023/01/spring-security-part4-GrantedAuthority.html)
* Part 5 - [Security tags for Freemarker](/spring/framework/2023/01/spring-security-part5-security-tags.html)

If you're new to Spring, you should take a look at [Getting started with Spring Framework for Web Apps](/spring/framework/2023/01/09/spring-initializer)

{% highlight java %}
{% endhighlight %}

# 1 Security Tags
Implementing security tags with server-side rendering technologies is fairly common, but Spring Security doesn't ship with any out of the box. Thankfully, setting these up in Free Marker (and others) is fairly straight-forward. This article will leverage knowledge and code exposed in prior articles so it would be a good idea to review those first. 


## 1.2 Creating the tags
The code I'm using has been adapted from https://github.com/jagregory/shiro-freemarker-tags for Freemarker running with Spring Security. After copying these classes into my project, I modified them to work with Spring Security, GrantedAuthority and my user contexts. 

The `SecurityTags` class is the entry point from your FreeMarker template. The names you give your tags here is how you'll use it in your template. 

{% highlight java %}
public class SecurityTags extends SimpleHash {
    public SecurityTags() {
        put("authenticated", new AuthenticatedTag());
    	
        put("guest", new GuestTag());
        put("hasAnyRole", new HasAnyRolesTag());
        put("hasPermission", new HasPermissionTag());
        put("hasRole", new HasRoleTag());
        put("lacksPermission", new LacksPermissionTag());
        put("lacksRole", new LacksRoleTag());
        put("notAuthenticated", new NotAuthenticatedTag());
        put("principal", new PrincipalTag());
        put("user", new UserTag());
        
    }
}
{% endhighlight %}

## 1.3 Customizing your tags
The `PermissionTag` is a good example to see a custom [SecurityUtils](/spring/framework/2023/01/spring-security-part3-OidcUser.html) in action. 

{% highlight java %}
public abstract class PermissionTag extends SecureTag {
    String getName(Map params) {
        return getParam(params, "name");
    }
    
    @Override
    protected void verifyParameters(Map params) throws TemplateModelException {
        String permission = getName(params);

        if (permission == null || permission.length() == 0) {
            throw new TemplateModelException("The 'name' tag attribute must be set.");
        }
    }

    @Override
    public void render(Environment env, Map params, TemplateDirectiveBody body) throws IOException, TemplateException {
        String p = getName(params);

        boolean show = showTagBody(p);
        if (show) {
            renderBody(env, body);
        }
    }

    protected boolean isPermitted(String p) {
    	UserContext userContext = SecurityUtils.getUserContext();
    	
    	if (userContext != null) {
    		return userContext.hasAnyRole(p);
    	}
    	return false;
    }

    protected abstract boolean showTagBody(String p);
}
{% endhighlight %}

## 2 FreeMarkerConfigurer
Finally, you'll want to create a new bean in your `Configuration` that returns a [FreeMarkerConfigurer](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/view/freemarker/FreeMarkerConfigurer.html). Add in a shared variable called **security** and set the object to a new `SecurityTags` object.

{% highlight java %}
    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() throws IOException {
    	FreeMarkerConfigurer freeMarkerConfigurer = new FreeMarkerConfigurer();

    	// Create a new configuration 
    	freemarker.template.Configuration config = new freemarker.template.Configuration(freemarker.template.Configuration.DEFAULT_INCOMPATIBLE_IMPROVEMENTS);
    	
    	// Set the templateLoader based on the ClassTemplateLoader
    	config.setTemplateLoader( new ClassTemplateLoader(getClass(), "/templates"));
    	
    	// Set up tags to assist with security in the templates
    	config.setSharedVariable("security", new SecurityTags());

    	// Set the configuration and return the configurer
    	freeMarkerConfigurer.setConfiguration(config);
    	
    	return freeMarkerConfigurer;
    }
{% endhighlight %}


## 2.1 Using the security tags
Now, in your FreeMarker template file you can use the security tags by using `<@ TAGNAME>`. 

{% highlight java %}
<@security.authenticated>
	<p>IS AUTHENTICATED</p>
</@security.authenticated>

<@security.hasRole name="USER">
	<h3>You have the USER role</h3>
</@security.hasRole>

<@security.hasAnyRole name="ADMIN,WEBMASTER">
	<h3>Has at least one of these roles: ADMIN,WEBMASTER</h3>
</@security.hasAnyRole>

{% endhighlight %}