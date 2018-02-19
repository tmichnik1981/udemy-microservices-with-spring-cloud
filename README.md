# Microservices with SpringCloud

#### Links

- [Spring Boot reference](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/)
- [Spring Cloud reference](http://projects.spring.io/spring-cloud/docs/1.0.1/spring-cloud.html)
- [Lab1](https://github.com/kennyk65/Microservices-With-Spring-Student-Files/blob/master/LabInstructions/Lab%201.md#lab-1---spring-boot)
- [Lab4](https://github.com/kennyk65/Microservices-With-Spring-Student-Files/blob/master/LabInstructions/Lab%204.md)
- [Lab5](https://github.com/kennyk65/Microservices-With-Spring-Student-Files/blob/master/LabInstructions/Lab%205.md)
- [Lab6](https://github.com/kennyk65/Microservices-With-Spring-Student-Files/blob/master/LabInstructions/Lab%206.md)

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

###### Spring creates 2 contexts:

- first downloads the configuration from a server
- the second stars main app

###### Spring Cloud Config uses `EnvironmentRepository` 

- Git implementation
- Native (local files) implementation

###### Naming convention

- **<SPRING_APP_NAME>-<PROFILE>.yml**
- or .properties
- **SPRING_APP_NAME** is configureed in bootstarap.yml on client site

###### Obtaining settings from server

- http://<SERVER>:<PORT>/<SPRING-APP-NAME>/<PROFILE>

Spring apps have their **Environment** object.

**Environment**  contains multiple PropertySources (system, JNDI etc). Spring Cloud Config Clients adds another **PropertySource**. 

Client app control policy of how to handle missing config server

- `spring.cloud.config.failFast=true`
- config server override local settings
- Tip: provide local fallback settings

##### Service Discovery - Eureka

- provides a lookup server
- highly available by running multiple copies
- copies replicate state of registered services
- "Client" services register with Eureka
  - provide metadata on host , port, health indicator URL, etc.
  - Eureka removes services without heartbeats

###### Setting up the Eureka server

1. Dependencies

   ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-eureka-server</artifactId>
   </dependency>
   ```

2. Annotations

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class Application {

       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }
   }
   ```

3. Common configuration

   ```yaml
   server:
     port: 8011  
   eureka:
     instance: # exposing some info
     	statusPageUrlPath: ${management.contextPath}/info
     	healthCheckUrlPath: ${management.contextPath}/health 
       hostname: localhost 
     client: # true - on production
       registerWithEureka: false
       fetchRegistry: false
       serviceUrl: # other Eureka servers
         defaultZone: http://server:port/eureka/,http://server:port/eureka/

   ```

###### Setting up the Eureka client

1. Dependencies

   ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   ```

2. Annotations

   - @EnableDiscoveryClient - sends application name, host, port to eureka server.

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class MyApplication {

       public static void main(String[] args) {
           SpringApplication.run(MyApplication.class, args);
       }
   }
   ```

3. Configurations

   ```
   eureka:
     client:
       serviceUrl:
         defaultZone: http://server:8010/eureka/
   ```

Client after registration becomes a consumer of service registry. 

###### Config First Boostrap

- Spring app will pull information about eureka server from the config server  


- Implies spring,cloud.config.uri

###### Eureka First Bootstrap

- Config server is just another client
- Client will get url of Config Server from Eureka
- Implies `spring.cloud.config.discovery.enabled=true` and `eureka.client.serviceUrl.defaultZone` configured in each app

##### Spring Cloud Ribbon

###### Client Side Load Balancer

- Based on some criteria
- Part of client software
- Server can still employ its own load balancer

###### Ribbon

- Client side load balancer - clients selects a server based on some criteria
- Automatically integrates with service discovery (Eureka)
- Built in failure resiliency (Hystrix)
- Caching / Batching
- Multiple protocols (HTTP, TCP, UDP)

1. List of services of each type
   - static - from a config file
   - dynamic - from eureka servers
2. Filtered list of servers
   - criteria by which you wish to limit the total list
   - by default - limits servers to those from the same zone
3. Ping
   - used to test if the server is up or down
   - by default  -delegate to Eureka to determine if server is up or down
   - it might happen that server is down but Eureka was not aware of it yet
4. Load Balancer
   - routes the calls to the servers in the filtered list
5. Rule
   - single module of inlelligence that makes the decisions on whether to call or not
   - Spring Clouds Default: ZoneAvoidanceRule

###### Setting up

- Dependencies

  ```xml
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
  	
  <dependency>
  	<groupId>org.springframework.cloud</groupId>
  	<artifactId>spring-cloud-starter-ribbon</artifactId>
  </dependency>
  ```

- Low level usage. Ribbon API.

  ```java
  public class MyClass{
    @Autowired
    LoadBalancerClient loadBalancer;
    
    public void doStuff(){
      ServiceInstance instance = loadBalancer.choose("subject");
      URI subjectUri = URI.create(String.format("http://%s:%s", instance.getHost(), instance.getPort()));
      
      // ...
    } 
    
  }
  ```

- Customizing

  - [ribbon java-docs](http://netflix.github.io/ribbon/ribbon-core-javadoc/index.html)
  - [ribbon source code ](https://github.com/Netflix/ribbon)

  ```java
  // declare ribbon client and config
  //DO NOT COMPONENT SCAN
  @Configuration
  @RibbonClient(name="subject", configuration=SubjectConfig.class)
  public class MainConfig{
    
  }

  @Configuration
  public class SubjectConfig{
    @Bean
    public IPing ribbonPing(IClientConfig config){
      return new PingURL();
    }
  } 
  ```

##### Feign

- Declarative REST client from NetFlix
- Alternative to RestTemplate
- How doest it work?
  - Define interfaces for your REST client code
  - Annotate interface with Feign annotation
  - Annotate methods with String MVC annotations
    - Other implementations like JAX/RS pluggable

1. Add dependecy

   ```xml
   <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-feign</artifactId>
   </dependency>
   ```

2. Create an Interface

   ```java
   @FeignClient(url="localhost:8080/warehouse") //will be replaced with Eureka client ID 
   public interface InventoryClient{
     @RequestMapping(method = RequestMethod.GET, value = "/inventory")
     List<Item> getItems();
     
     @RequestMapping(method = RequestMethod.POST,
                    value = "/inventory/{sku}",
                    consumes = "application/json")
     void update(@PathVariable("sku") Long sku, @RequestBody Item item)
   }
   ```

3. Enable Feign

   ```java
   @SpringBootApplication
   @EnableFeignClients
   public class Application{
     //...
   }
   ```

   ​