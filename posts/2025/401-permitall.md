---
title: Skille enhets- og integrasjonstester i maven
date: 2025-03-27
author: Bjørn
tags:
  - maven
  - test
---

# Enhetstesting av controllere i Spring (Boot)

Det er mulig å teste RestControllere i Spring Boot som en enhetstest, hvor man ikke trenger å laste hele spring-konteksten og ha en database tilgjengelig.

`@WebMvcTest` vil laste den angitte controlleren, andre bønner som spesifiseres og relaterte MVC-bønner som:
* @ControllerAdvice
* @JsonComponent
* Filter
* WebMvcConfigurer
* HandlerMethodArgumentResolver

Avhengigheter som controlleren trenger mocker en. i testklassen med @MockitoBean

Dersom 





https://stackoverflow.com/questions/39554285/spring-test-returning-401-for-unsecured-urls

@WebMvcTest will auto-configure the Spring MVC infrastructure and limit scanned beans to @Controller, @ControllerAdvice, @JsonComponent, Filter, WebMvcConfigurer and HandlerMethodArgumentResolver. Regular @Component beans will not be scanned when using this annotation.


The @WebMvcTest by default auto configure spring security if spring-security-test is present in the class path (which in my case is).

So since WebSecurityConfigurer classes aren't picked, the default security was being auto configured, that is the motive I was receiving the 401 in url's that was not secured in my security configuration. Spring security default auto configuration protects all url's with basic authentication.