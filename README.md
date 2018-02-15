# Microservices with SpringCloud

#### Links

- [Spring Boot reference](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/)
- [Spring Cloud reference](http://projects.spring.io/spring-cloud/docs/1.0.1/spring-cloud.html)
- [Lab1](https://github.com/kennyk65/Microservices-With-Spring-Student-Files/blob/master/LabInstructions/Lab%201.md#lab-1---spring-boot)

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

#### Spring Cloud

###### Goals of Spring Cloud

- Provide libraries to apply common patterns needed in distributed applications:
  - Distributed / Versioned / Centralized Configuration Management
  - Service Registration and Discovery
  - Load Balancing
  - Service-to-service Calls
  - Circuit Breakers
  - Routing
  - ...

###### Dependencies

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Edgware.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId></groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId></groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```

##### Spring Cloud Config

- centralized 
- secured
- easy to reach
- server
  - configuration can backed by source control
- clients 
  - connect over http and retrieve their confoguration settings - in addition to their own internal sources of configuration

###### Server implementation

- configuration: application.yml

  ```yaml
  spring:
    cloud:
      config:
        server:
          git: # path to git repository where we kepp config files
            uri: https://github.com/kennyk65/Microservices-With-Spring-Student-Files
            searchPaths: ConfigData # subdirectory
          # "native" is used when the native profile is active,  
          # for local tests with a  classpath repo:
          native:
            searchLocations: classpath:offline-repository/
  server:
    port: 8001
  ```

- java setup: `@SpringBootApplication`

  ```java
  @SpringBootApplication
  @EnableConfigServer
  public class Application {

      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
      }
  }
  ```

###### Client implementation

- dependency

  ```xml
  <dependency>
      	<groupId>org.springframework.cloud</groupId>
        	<artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
  ```

- configuration - `bootstrap.properties / yml`

  ```properties
  # boostrap.properties - is loaded before other properties
  spring.application.name: lucky-word
  spring.cloud.config.uri: http://localhost:8001
  ```

Spring creates 2 contexts:

- first downloads the configuration form a server
- ​