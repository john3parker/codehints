---
layout: post
title:  "Github Pages Custom Domain"
date:   2023-01-09 09:00:00 -0600
categories: github config
---

Github Pages supports using a custom domain for your hosted site. Custom domains allow you to serve your site from a domain other than the default at github.io. After registering a domain name, follow these steps to complete the configuration. 

## Configuration
You should first secure a domain name by registering with a domain name authority. There are several available including: Amazon AWS Route53, Google Domains and GoDaddy. 

### Step 1
Before configuring your domain with your DNS provider, make sure to add your custom domain name to Github Pages. It will generate a DNS error initially until you set up the CNAME record in step 2. 

![Github ](/assets/2023/github-customdomain/github-pages-custom-domain.png){:.img-fluid}


This action will also create a CNAME file in the root of your repository. Leave this file here, its how Github Pages identifies and routes your website properly.

![Github ](/assets/2023/github-customdomain/github-pages-cname-entry.png){:.img-fluid .w-50}

### Step 2
Using your DNS provider's website, set up a CNAME record. This image below is from AWS Route53. Fill in each marked area. 

![Github ](/assets/2023/github-customdomain/github-pages-cname-entry-creation.png){:.img-fluid .w-50}


### Complete
That's it. In just a few minutes you'll be able to start using your custom domain. 