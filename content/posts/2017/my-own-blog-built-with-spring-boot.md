---
title: My own blog built with Spring Boot
date: 2017-10-05T22:50:57+08:00
tags: ["blog", "spring-boot"]
draft: false
---

I have been planning to build my own blog (previously it was powered by [WordPress](https://wordpress.org/)) for a long time, and now it finally comes.

The new blog is build with Spring Boot.

<!--more-->

Why Spring Boot? Because I have switched to Java for several months. In addition to powering my blog site, the new blog also serves as a learning project.

In addition to Spring Boot, it also uses the following technologies or services:

* PostgreSQL - Mainly for its [partial index](https://www.postgresql.org/docs/current/static/indexes-partial.html) feature
* [Flyway](https://flywaydb.org/) - Manage database migration
* [Freemarker](http://freemarker.org/) - Powerful template engine for Java
* [flexmark](https://github.com/vsch/flexmark-java) - Java Markdown parser
* [Disqus](https://disqus.com/) - Popular comment service
* [Broccoli.js](http://broccolijs.com/) - Manage frontend assets
* [Semantic UI](https://semantic-ui.com/) - Beautiful UI framework

Source code is available on [GitHub](https://github.com/bianjp/blog-spring).

The features are quite limited for now, but it will be iterated gradually.
