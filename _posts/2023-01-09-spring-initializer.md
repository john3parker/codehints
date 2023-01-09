---
layout: post
title:  "Getting started with Spring Framework for Web Apps"
date:   2023-01-09 08:39:21 -0600
categories: spring framework
---
Spring is a Java framework which allows you to quickly build many different types of applications. It provides the foundational layers of technology so you can focus on building business logic. Spring has a framework for microservices, reactive applications, cloud, web applications, serverless, event driven for streaming data and batch. 

# Getting Started
In ths example, we're going to use Spring 3.x, Java 17 and Maven to build a simple web application to serve up content using Apache Freemarker. We'll leverage Spring Boot, which encapsulates Apache Tomcat as the default embedded servlet container.

## Prerequistes
Make sure you have Java 17 and Eclipse installed for your operating system before proceeding. 

## Generating the project
The easiest and quickest way to get started with Spring is to use the [Spring Initializer](https://start.spring.io/). To follow along with the example, be sure to select `Maven` for project, `Java` as language, `3.0.1`, `War` for the packaging and `Java 17` version. Take all the remaining defaults for now.

![Github ](/assets/2023/spring/spring-initializer.png){:.img-fluid .w-25}


#### Add dependencies
Click the `Add Dependencies` button and select the following: 
* Spring Boot Dev Tools
* Spring Web
* Apache Freemarker

![Github ](/assets/2023/spring/spring-initializer-dependencies.png){:.img-fluid .w-50}

Click the `Generate` button and it'll download a starter project. Open the resulting zip and extract the results into a folder. 

## Setting up the project
Open Eclipse and set the workspace to the top level directory. Click on `Import projects` or use the menu `File => Import`. Open the `Maven` folder and select `Existing Maven Projects`. Browse to the folder containing your project. Give it a minute for Maven to build the project.

![Github ](/assets/2023/spring/spring-eclipse-import-project.png){:.img-fluid .w-50}

![Github ](/assets/2023/spring/spring-eclipse-maven-import-project.png){:.img-fluid .w-50}

![Github ](/assets/2023/spring/spring-eclipse-maven-imported-project.png){:.img-fluid .w-50}

## Validation
To validate everything is working, run the Spring Boot application. Open the `src/main/java` and locate the `DemoApplication.java` file. Right-click, Run As, Java Application.  

![Github ](/assets/2023/spring/spring-eclipse-run-project.png){:.img-fluid .w-50}


You should get output similar to this. You can browse to `localhost:8080` in your browser but no output will occur until we configure a web controller.

![Github ](/assets/2023/spring/spring-eclipse-running-project.png){:.img-fluid .w-50}

## Setting up a web controller
Create a new class and name it `HomeController`. Enter the following code into the new class. We are annotating the class with the Spring sterotype `@Controller`. Spring will automatically detect and set up a web servlet for us. The `GetMapping` value set up a context so the servlet will serve content when you navigate to root at `localhost:8080`. 

The return type will be a string which will translate to an Apache Freemarker template named `home.ftlh`. 

{% highlight java %}
package com.example.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

	@GetMapping("/")
	public String home(Model model) {
	
		return "home";
	}
}
{% endhighlight %}

![Github ](/assets/2023/spring/spring-eclipse-new-controller-class.png){:.img-fluid .w-50}


## Setting up Freemarker
Now locate the `src/main/resources` folder and create a new `Text` file in the `resources` folder named `home.ftlh`. Enter the HTML below and save the file. 

![Github ](/assets/2023/spring/spring-eclipse-freemarker-file.png){:.img-fluid .w-25}


{% highlight java %}
<html>
	<body>
		<h1>Hello World</h1>
	</body>
</html>
{% endhighlight %}

## Test your work
Now point your browser to [localhost:8080](http://localhost:8080/) and you should see the HTML displayed.