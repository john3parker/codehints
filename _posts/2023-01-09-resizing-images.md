---
layout: post
title:  "Quickly Resize Images"
date:   2023-01-09 9:30:00 -0600
categories: github config
---

Want to quickly resize a large number of images? This useful command will save your hours of effort. 

## Resize images
This command works on most Ubuntu installations. Navigate to the directory where your images are stored, change the percentage to whatever you'd like and presto. Your images are quickly and easily reduced in size.

``
find . -name "*.png" | xargs --replace=file convert -resize 25% file file
``
