# DAY-2-SB: Spring Boot Basics

*Monday, 28 September 2026*

## 1. Spring Boot

Spring Boot is a framework built on top of Spring that simplifies the development of Spring applications by providing auto-configuration, starter dependencies, and embedded servers.

Without Spring Boot:

```text
Spring
 ↓
Lots of configuration
 ↓
Configure dependencies
 ↓
Configure server
 ↓
Configure application
 ↓
Run application
```

With Spring Boot:

```text
Spring Boot
 ↓
Auto Configuration
 ↓
Starter Dependencies
 ↓
Embedded Server
 ↓
Run Application
```

---

### 1.1 Autoconfiguration

Spring Boot automatically configures many things based on the dependencies you've added.

Spring Boot Auto Configuration automatically configures your application based on the dependencies you have added and the configuration available in your application.

Auto Configuration is a Spring Boot feature that automatically configures application infrastructure based on the dependencies present on the classpath, existing configuration, and conditions. It is enabled through @EnableAutoConfiguration, which is included in @SpringBootApplication.

Imagine you want to build a REST API.

You add:

```text
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot sees that you're using the web starter.

It can then configure things needed for a web application, such as:

```text
Web application
Spring MVC
Embedded Tomcat
JSON support
```

You don't have to manually configure every one of these pieces.

---

### 1.2 Starter Dependencies

A Spring Boot Starter is a convenient dependency that groups together the commonly required libraries for a particular feature.

```text
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
-> This gives you the dependencies needed for building web applications.
```

Example

Suppose you want to create a REST API.

Instead of manually adding multiple libraries, you add:

```text
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This is:

```text
spring-boot-starter-web
        ↓
Web-related dependencies
        ↓
Spring MVC
        ↓
REST API support
        ↓
Embedded web server support
        ↓
etc.
```

---

### 1.3 Embedded Server

Spring Boot can run with an embedded server such as Tomcat.

So you don't necessarily need to separately install and configure a web server just to run your application.

Example:

```text
@SpringBootApplication
public class EmployeeApplication {

    public static void main(String[] args) {

        SpringApplication.run(
            EmployeeApplication.class,
            args
        );
    }
}
```

Output:

```text
SpringApplication.run()
    ↓
Spring Boot starts
    ↓
Creates ApplicationContext
    ↓
Creates Spring Beans
    ↓
Starts embedded Tomcat
    ↓
Tomcat listens for HTTP requests
    ↓
Application is ready
```

Then you can access: http://localhost:8080

You don't have to:

1. Install Tomcat separately
2. Configure Tomcat
3. Deploy your application manually
4. Start Tomcat separately

Instead:

```text
Run Spring Boot application
        ↓
Embedded Tomcat starts
        ↓
Application starts
```

That's why Spring Boot applications are often described as standalone applications.

An embedded server is a web server that is packaged with and started by the Spring Boot application itself. For example, Spring Boot uses embedded Tomcat by default for many web applications, so we don't need to install and deploy the application to an external Tomcat server.

---

## 2. Spring Framework vs Spring Boot

| Spring Framework | Spring Boot |
|---|---|
| Core framework | Built on top of Spring |
| More configuration may be required | Less manual configuration |
| Doesn't itself provide the same opinionated setup | Provides auto-configuration |
| External server commonly used traditionally | Embedded server support |
| Dependencies can require more manual selection | Starter dependencies |
| More setup | Faster application development |

Spring is a comprehensive framework that provides features such as IoC, DI, MVC, Data, and Security. Spring Boot is built on top of Spring and simplifies application development by providing auto-configuration, starter dependencies, embedded servers, and sensible defaults.

---

## 3. Starter Dependency

A dependency is a library that your application needs to use some functionality.

Without Starter:

```text
Find dependencies individually
        ↓
Add them
        ↓
Manage versions
        ↓
Make sure they work together
```

With starter:

```text
spring-boot-starter-web
        ↓
Common web dependencies
        ↓
Ready to build web application
```

Spring starter web brings dependencies like Spring MVC, embedded Tomcat, JSON support.

It is called a starter because it brings the commonly needed dependencies for building a web application. Maven resolves dependencies for you.

---

## 4. Spring Initializr

Spring Initializr is a web-based tool that helps you quickly create the initial structure of a Spring Boot project with the required dependencies and configuration.

Instead of manually creating: pom.xml, src/main/java, src/main/resources, main application class, Spring Boot configuration — you select your requirements and Initializr generates the project.

You choose things like:

```text
Project: Maven
Language: Java
Spring Boot version
Group: com.example
Artifact: employee-management
Packaging: Jar
Java: 21
```

Then you select dependencies such as:

```text
Spring Web
Spring Data JPA
Validation
Spring Security
PostgreSQL Driver
```

Spring Initializr generates the project with those dependencies already configured.

Initializr → creates the project
Spring Boot → runs the application

---

## 5. @SpringBootApplication

@SpringBootApplication is a convenience annotation that combines three important annotations: @Configuration, @EnableAutoConfiguration, and @ComponentScan.

So instead of writing all three separately, we simply write @SpringBootApplication.

- @Configuration: This class can contain configuration and Bean definitions.
- @EnableAutoConfiguration: Automatically configure the application based on the dependencies and environment.
- @ComponentScan: Finds your Beans.

```text
@SpringBootApplication
        │
        ├──────────────┐
        ↓               ↓
 @ComponentScan   @EnableAutoConfiguration
        │               │
        ↓               ↓
 Find your Beans   Configure infrastructure
        │               │
        └───────┬───────┘
                ↓
       Spring ApplicationContext
                ↓
          Beans are created
                ↓
        Dependency Injection
                ↓
          Application starts
```

---

## 6. application.properties & application.yml

These files are used for configuring your Spring Boot application.

For example, you may want to configure:

- Server port
- Database connection
- Logging
- Application-specific settings
- Profiles

So these things we add in application.properties or application.yml.

### 6.1 application.properties

This uses a simple key=value format.

Example: server.port = 8081 -> this tells Spring Boot to run on port 8081 instead of default.

Stored in:

```text
src
└── main
    └── resources
        └── application.properties
```

### 6.2 application.yml

YAML uses hierarchical/structured indentation.

```text
server:
  port: 8081

spring:
  application:
    name: employee-management
```

Stored in:

```text
src
└── main
    └── resources
        └── application.yml
```

---

## 7. Difference between @Component and Component Scanning

@Component is placed on a class.

```text
@Component
public class EmailService {

    public void sendEmail() {
        System.out.println("Email sent");
    }
}
```

Tells Spring:

"This class is a Spring-managed component. Create and manage its object as a Bean."

```text
@Component
    ↓
Marks a class
```

@ComponentScan is the process performed by Spring to search your application packages and find classes marked with annotations such as @Component, @Service, @Repository, and @Controller.

```text
Component Scanning
        ↓
Searches packages
        ↓
Finds @Component / @Service / @Repository / @Controller
        ↓
Spring creates and manages their Beans
```

@Component is an annotation used to mark a class as a Spring-managed component, whereas component scanning is the process by which Spring searches application packages for such annotated classes and registers them as Beans.
