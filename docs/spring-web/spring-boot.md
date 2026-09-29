# Spring Boot

## Spring Boot

| Spring | Spring Boot |
| :---- | :---- |
| Spring is an open-source lightweight framework widely used to develop enterprise applications. | Spring Boot is built on top of the conventional spring framework, widely used to develop REST APIs. |
| The most important feature of the Spring Framework is dependency injection. | The most important feature of the Spring Boot is Autoconfiguration. |
| It helps to create a loosely coupled application. | It helps to create a stand-alone application. |
| To run the Spring application, we need to set the server explicitly. | Spring Boot provides embedded servers such as Tomcat and Jetty etc. |
| To run the Spring application, a deployment descriptor is required. | There is no requirement for a deployment descriptor. |
| To create a Spring application, the developers write lots of code. | It reduces the lines of code. |
| It doesn’t provide support for the in-memory database. | It provides support for the in-memory database such as H2. |
| Developers need to write boilerplate code for smaller tasks. | In Spring Boot, there is reduction in boilerplate code. |
| Developers have to define dependencies manually in the pom.xml file. | pom.xml file internally handles the required dependencies. |

Ioc & dependency injection

SpringBoot
Advantage
Annotations
Circuit Breaker
Spring Cloud
Spring MVC
DevTools

Java env variable precedence (springbboot too)
Property order - https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config
Spring profiles
Multiple db connection from Java spring
Spring boot profiles
Transaction - all or nothing =- spring boot

[https://www.geeksforgeeks.org/spring-mvc-framework/](https://www.geeksforgeeks.org/spring-mvc-framework/)

[https://medium.com/@TechiesSpot/mastering-mvc-in-java-spring-boot-a-comprehensive-guide-f7353a06fd61](https://medium.com/@TechiesSpot/mastering-mvc-in-java-spring-boot-a-comprehensive-guide-f7353a06fd61)

[https://www.jrebel.com/blog/spring-annotations-cheat-sheet](https://www.jrebel.com/blog/spring-annotations-cheat-sheet)

[https://www.baeldung.com/spring-qualifier-annotation](https://www.baeldung.com/spring-qualifier-annotation)

## Hystrix circuit breaker

[https://www.geeksforgeeks.org/implementing-a-basic-circuit-breaker-with-hystrix-in-spring-boot-microservices/](https://www.geeksforgeeks.org/implementing-a-basic-circuit-breaker-with-hystrix-in-spring-boot-microservices/)
[https://www.baeldung.com/spring-cloud-netflix-hystrix](https://www.baeldung.com/spring-cloud-netflix-hystrix)
[https://cloud.spring.io/spring-cloud-netflix/multi/multi__circuit_breaker_hystrix_clients.html](https://cloud.spring.io/spring-cloud-netflix/multi/multi__circuit_breaker_hystrix_clients.html)

The Hystrix circuit breaker is an open-source Java library that helps prevent cascading failures in distributed systems. It's a fault tolerance technique that monitors services and temporarily rejects calls when they're not behaving normally.

**How it works**

* Hystrix monitors services for failures
* When a service fails above a certain threshold, the circuit breaker opens
* The circuit breaker prevents further calls to the failing service
* The developer can provide a fallback, such as another Hystrix protected call, static data, or an empty value

**Hystrix benefits**

* Prevents cascading failures: Stops failures from spreading throughout a system
* Improves resilience: Helps systems recover quickly from failures
* Provides fault tolerance: Protects against latency and failure from dependencies
* Enables monitoring and alerting: Provides near real-time monitoring and alerting
