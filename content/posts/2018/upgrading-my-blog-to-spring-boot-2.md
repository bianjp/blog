---
title: Upgrading my blog to Spring Boot 2.0
date: 2018-03-06T14:51:26+08:00
tags: ["spring-boot"]
draft: false
---

Spring Boot 2.0 was finally released on May 1, 2018, and my blog has been upgraded to it from Spring Boot 1.5.9.

Spring Boot 2.0 is a major update of 17 months' work. It breaks some configurations due to refactoring and dependency updates along with it.

<!--more-->

__Package removal/rename:__

* `spring-boot-starter-mobile` starter is removed.
* `spring-session` should be replaced by `spring-session-data-redis`.

__Gradle plugin updates:__

* Dependency management plugin is no longer automatically applied, and should be explicitly enabled.
* The `bootRepackage` task is replaced by `bootJar`, and as a result `jar` task is no longer not invoked when building executable jars. This breaks configuration for `jar` task.

__Dependency updates:__

* Hibernate validator from 5.3.6 to 6.0.7: `org.hibernate.validator.constraints.NotEmpty` is deprecated and `javax.validation.constraints.NotEmpty` should be used.
* Flyway: Spring Boot's default `flyway.table` has been changed from `schema_version` to `flyway_schema_history`.
* `spring-data-commons` from 1.13 to 2.0: Configuration for 1-based pagination has to be changed.

__Refactoring__:

* `AbstractErrorController`, `ErrorViewResolver` are moved from package `org.springframework.boot.autoconfigure.web` to `org.springframework.boot.autoconfigure.web.servlet.error`.
* `FreeMarkerAutoConfiguration.FreeMarkerWebConfiguration` is replaced by a package-private class, breaking the configuration that extends the class.

__Configuation__:

* `management.security.*` is gone, and current solution is to configure web security programatically, making it harder to configure different strategies for different profiles.
* All actuator endpoints are moved to `/actuator` by default.

One problem that I found when upgrading is that not all changes are listed in Spring Boot 2.0 [release note](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes) and [migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide). You have to build the project to check errors and warning, and solve them one by one. Sometimes you have to check its source code for an alternative solution to broken configuration.
