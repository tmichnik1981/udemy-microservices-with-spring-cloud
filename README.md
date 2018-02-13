# Microservices with SpringCloud

#### Links

- [Spring Boot reference](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/)

#### Introduction

##### Breaking a Monolith into Microservices

###### Primary considerations: business functionality:

- Noun-based (catalog, cart, customer)
- Verb-based (search, checkout, shipping)
- Single Responsibility Principle
- Bounded Context (best guidence)

###### Size is not the compelling factor

#### Spring Boot

##### Converting JAR to WAR

1. Change POM packaging

2. Extend SpringBootServletInitializer

   ```java
   public class Application extends SpringBootServletInitializer{
     public static void main(String[] args){
       SpringApplication.run(Config.class, args);
     }
     
     protected SpringApplicationBuilder configure(SpringApplicationBuilder application){
       return application.sources(Config.class);
     }
   } 
   ```

3. Deploy to app server

##### Adding JPA Capability

###### Adding the spring-boot-starter-data-jpa Dependency

- Adds Spring JDBC / Transaction Management
- Adds Spring ORM
- Adds Hibernate / entity manager
- Adds Spring Data JPA subproject

###### Does NOT add a Database Driver

- Add one manualy