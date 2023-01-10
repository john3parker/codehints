---
layout: post
title:  "Max Upload File Size in Spring Framework"
date:   2023-01-10 11:54:21 -0600
categories: spring framework
---
Spring framework comes preconfigured with default upload file size and max request size. You can modify the sizes with application settings. 

# Settings
* `max-file-size` specifies the maximum size permitted for uploaded files. The default is 1MB
* `max-request-size` specifies the maximum size allowed for multipart/form-data requests. The default is 10MB.

## Error exceeding max file size
Java with Spring 6 will throw a `MaxUploadSizeExceededException` exception when the max size is exceeded.

{% highlight java %}
threw exception [Request processing failed: org.springframework.web.multipart.MaxUploadSizeExceededException: Maximum upload size exceeded] with root cause

org.apache.tomcat.util.http.fileupload.impl.FileSizeLimitExceededException: The field image exceeds its maximum permitted size of 1048576 bytes.
	at org.apache.tomcat.util.http.fileupload.impl.FileItemStreamImpl$1.raiseError(FileItemStreamImpl.java:117) ~[tomcat-embed-core-10.1.1.jar:10.1.1]
	at org.apache.tomcat.util.http.fileupload.util.LimitedInputStream.checkLimit(LimitedInputStream.java:76) ~[tomcat-embed-core-10.1.1.jar:10.1.1]
	at org.apache.tomcat.util.http.fileupload.util.LimitedInputStream.read(LimitedInputStream.java:135) ~[tomcat-embed-core-10.1.1.jar:10.1.1]
	at java.base/java.io.FilterInputStream.read(FilterInputStream.java:106) ~[na:na]
{% endhighlight %}

## Solution
Add the following settings to the `application.properties`.
{% highlight java %}
spring.servlet.multipart.max-request-size = 10MB
spring.servlet.multipart.max-request-size = 20MB
{% endhighlight %}

Or if you're using yaml in the `application.yml`. 
{% highlight java %}
spring:
  servlet:
	multipart:
	  max-file-size:  10MB
	  max-request-size:  20MB
{% endhighlight %}

