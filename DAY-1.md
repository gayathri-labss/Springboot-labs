# DAY-1-SB: Spring Core Basics


## 1. Why was Spring created?

Spring helps us achieve loose coupling.

```text
Spring Container
      ↓
Creates Objects
      ↓
Manages Objects
      ↓
Injects Objects
```

Developers focus on business logic. Spring handles object creation and wiring.

> Spring is a lightweight Java framework that provides features such as Dependency Injection and Inversion of Control to build loosely coupled, maintainable, and testable applications.

### Today's Key Takeaway

```text
Before Spring:
Developer
   ↓
Creates Objects
   ↓
Connects Objects

After Spring:
Developer
   ↓
Defines Classes
   ↓
Spring Creates Objects
   ↓
Spring Connects Objects
```

This is the biggest idea in Spring.

---

## 2. IoC (Inversion of Control)

Inversion of Control (IoC) is a design principle in which the responsibility of creating and managing objects is transferred from the application to the Spring Container.

Without IoC

```java
EmployeeRepository repo = new EmployeeRepository();

EmployeeService service =
        new EmployeeService(repo);
```

You control object creation.

With IoC

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }
}
```

Spring controls the creation and management of these objects.

```text
You
 ↓
Define classes

Spring Container
 ↓
Creates & manages objects
```

Inversion: Reverse the control of object creation.

IoC is not the same as DI (Dependency Injection). Dependency Injection (DI) is one way Spring implements IoC.

```text
IoC
│
├── Dependency Injection
├── Bean Management
└── Bean Lifecycle
```

> Note: The Spring IoC Container creates, manages, configures, and injects objects (beans).

---

## 3. Dependency Injection

A technique where an object's required dependency is provided to it from outside, instead of the object creating the dependency itself.

### Types of Dependency Injection

#### 3.1 Constructor Injection

Dependency is provided through the constructor.

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }
}
```

Here:

- EmployeeRepository → dependency
- EmployeeService → class that needs the dependency
- EmployeeService(EmployeeRepository repository) → constructor
- Spring provides EmployeeRepository through the constructor.

Conceptually, Spring does:

```java
EmployeeRepository repository = new EmployeeRepository();

EmployeeService service =
        new EmployeeService(repository);
```

You don't write this wiring yourself; Spring handles it.

#### 3.2 Setter Injection

Spring provides the dependency through a setter method once the object is created.

```java
@Service
public class EmployeeService {

    private EmployeeRepository repository;

    @Autowired
    public void setRepository(EmployeeRepository repository) {
        this.repository = repository;
    }
}
```

The important difference:

Constructor Injection

```text
Create object + provide dependency
        ↓
EmployeeService(repository)
```

Setter Injection

```text
Create object
    ↓
Provide dependency through setter
    ↓
setRepository(repository)
```

- In constructor injection, the dependency is provided while creating the object. In setter injection, it is provided once the object is created.

Constructor injection:

```java
EmployeeService service =
        new EmployeeService(repository);
```

The dependency is given while creating the object.

Setter injection:

```java
EmployeeService service =
        new EmployeeService();

service.setRepository(repository);
```

The object is created first, and the dependency is provided afterward.

Full example:

```java
@Repository
public class EmployeeRepository {

    public void save() {
        System.out.println("Employee Saved");
    }
}
```

```java
@Service
public class EmployeeService {

    private EmployeeRepository repository;

    @Autowired
    public void setRepository(EmployeeRepository repository) {
        this.repository = repository;
    }

    public void saveEmployee() {
        repository.save();
    }
}
```

Spring essentially handles this for us:

```java
EmployeeRepository repository = new EmployeeRepository();

EmployeeService service = new EmployeeService();

service.setRepository(repository);
```

You don't manually write these lines when using Spring.

Key points

- In Setter Injection, the dependency is assigned after the object is created through a setter method, so the field cannot be final. In Constructor Injection, the dependency is assigned during object creation through the constructor, so we can make the field final and ensure it cannot be reassigned later.
- Mandatory dependency → use constructor injection. The class should not be created without that dependency. Constructor Injection ensures this because the dependency must be provided at the time of object creation.

  ```java
  public EmployeeService(EmployeeRepository repository) {
      this.repository = repository;
  }
  ```

  When creating the object, we must provide an EmployeeRepository:

  ```java
  EmployeeRepository repository = new EmployeeRepository();

  EmployeeService service =
          new EmployeeService(repository);
  ```

  If we don't provide the repository, we cannot create the EmployeeService object using this constructor.

- Optional dependency → use setter injection. The object can be created first and the dependency provided later through the setter method.

  ```java
  EmployeeService service = new EmployeeService();

  service.setRepository(repository);
  ```

  So there is a possibility that the EmployeeService object exists without its required dependency before setRepository() is called.

Summary

- Constructor Injection → dependency is required at object creation.
- Setter Injection → dependency can be provided after object creation.

#### 3.3 Field Injection

Field Injection is a type of Dependency Injection where Spring directly injects the dependency into a class field using @Autowired. Instead of passing the dependency through a constructor or setter, Spring directly puts the dependency into the variable.

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

}
```

Spring sees @Autowired and injects the EmployeeRepository object into repository.

> Note: Although Field Injection is simple and requires less code, Constructor Injection is generally preferred, especially for mandatory dependencies, because it makes dependencies explicit and allows fields to be final.

---

## 4. Difference between Dependency Injection & IoC

1. IoC is a principle; DI is a way/technique to achieve IoC.
2. IoC is a principle where control of object creation and management is transferred to the Spring Container. Dependency Injection is one technique through which Spring implements IoC by providing the required dependencies to a class instead of the class creating them itself.

| IoC | DI |
|---|---|
| Design principle | Technique/pattern |
| Broader concept | More specific |
| Says who controls object creation | Says how dependencies are provided |
| Spring Container takes control | Spring injects dependencies |
| Can be achieved in different ways | Constructor, Setter, Field injection |

---

## 5. Spring Core Concepts

### 5.1 Spring Bean

A Spring Bean is simply an object that is created, managed, and maintained by the Spring Container.

Key phrase to remember: Object managed by Spring = Spring Bean

```java
@Repository
public class EmployeeRepository {

    public void save() {
        System.out.println("Employee Saved");
    }
}
```

Because we have @Repository, Spring detects EmployeeRepository, creates an object of it, and manages it as a bean.

```text
@Repository → Spring Bean
@Service    → Spring Bean
@Controller → Spring Bean
@Component  → Spring Bean
```

These annotations tell Spring to create and manage objects of these classes.

---

## 6. How does Spring create a Bean?

Spring creates beans in 2 ways:

- Component scanning: using @Component, @Service, @Repository, @Controller
- Java configuration: using @Bean

### 6.1 Component Scanning

Component Scanning is the process by which Spring searches your application classes for specific Spring annotations and automatically creates and manages objects (Beans) for those classes.

Without Spring: you have to create objects, connect objects, and manage objects. Example:

Suppose you have three classes:

```java
public class EmployeeRepository {

    public void save() {
        System.out.println("Employee Saved");
    }
}
```

```java
public class EmployeeService {

    private EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

    public void saveEmployee() {
        repository.save();
    }
}
```

```java
public class EmployeeController {

    private EmployeeService service;

    public EmployeeController(EmployeeService service) {
        this.service = service;
    }
}
```

Without Spring, you have to create all these objects yourself:

```java
EmployeeRepository repository =
        new EmployeeRepository();

EmployeeService service =
        new EmployeeService(repository);

EmployeeController controller =
        new EmployeeController(service);
```

How Spring scans

When your application starts, it looks through a specific set of packages in your application.

```text
Start Spring Boot
        ↓
Find the main application package
        ↓
Scan classes inside that package/subpackages
        ↓
Look for Spring component annotations
        ↓
Find @Controller
Find @Service
Find @Repository
Find @Component
        ↓
Create objects
        ↓
Register them as Beans
        ↓
Manage them
```

Spring looks for classes marked with: @Component, @Service, @Repository, @Controller, @RestController.

> Note: Component scanning does NOT scan every class and turn every class into a Bean.

```java
public class Employee {
}
```

Here there is no component annotation, so Spring doesn't automatically treat this class as a component just because it exists.

```java
@Component
public class Employee {
}
```

Now Spring can discover it during component scanning and create a Bean.

### 6.2 Stereotype annotations

1. @Component: generic purpose, Spring-managed class. Spring can discover it during component scanning and create a Bean.

```java
@Component
public class EmailValidator {

    public boolean isValid(String email) {
        return email.contains("@");
    }
}
```

2. @Service: handles business logic.

```java
@Service
public class EmployeeService {

    public void createEmployee() {
        System.out.println("Creating employee...");
    }
}
```

3. @Repository: for database access.

```java
@Repository
public class EmployeeRepository {

    public void saveEmployee() {
        System.out.println("Saving employee to database...");
    }
}
```

4. @Controller: handles web requests in Spring MVC.

```java
@Controller
public class EmployeeController {

    @GetMapping("/employees")
    public String employees() {
        return "employees";
    }
}
```

### 6.3 Java Configuration (@Bean)

We use @Bean. It tells Spring: create and manage the object returned by this method as a Spring Bean.

```java
@Configuration
public class AppConfig {

    @Bean
    public EmployeeService employeeService() {
        return new EmployeeService();
    }
}
```

- @Bean public EmployeeService employeeService() {: we let Spring know that the object returned from this method has to be registered as a bean.
- return new EmployeeService();: here we explicitly return a new object.

> Note:
>
> @Component → class-level
> @Bean → method-level

Using @Component, we tell Spring to discover the class, and create & manage its objects.

```java
@Component
public class EmailService {
}
```

Using @Bean, we tell Spring to use this method to create the object and manage the returned object.

```java
@Configuration
public class AppConfig {

    @Bean
    public EmailService emailService() {
        return new EmailService();
    }
}
```

We need @Bean when we are using a third-party class that we cannot modify (we can't add annotations to it because we don't own its source code).

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new PaymentClient("my-api-key");
    }
}
```

---

## 7. Bean Scope

Bean scope defines the lifecycle and number of instances of a Spring Bean managed by the Spring container.

Example:

```java
@Service
public class EmployeeService {
}
```

Should Spring create one EmployeeService object, or a new one every time it is requested? This is what bean scope controls.

By default, bean scope is singleton: Spring creates one instance of the bean per IoC container.

Other bean scopes:

| Scope | Meaning |
|---|---|
| singleton | One instance per Spring container. This is a shared object, as we have only one object |
| prototype | New instance whenever requested. This is not a shared object |
| request | One instance per HTTP request |
| session | One instance per HTTP session |
| application | One instance per ServletContext |
| websocket | One instance per WebSocket session |

Syntax to define different scopes of a bean:

```java
@Service
@Scope("prototype")
public class EmployeeService {

    public EmployeeService() {
        System.out.println("EmployeeService object created");
    }
}
```

> "Singleton is the default Spring Bean scope, where one instance is created per Spring Container. Prototype scope creates a new Bean instance each time the Bean is requested from the container."

---

## 8. Bean Lifecycle

The Bean Lifecycle is the sequence of steps a Spring Bean goes through from the time Spring creates the Bean until Spring destroys it.

```text
Spring Container starts
        ↓
Create Bean object
        ↓
Constructor
        ↓
Inject dependencies
        ↓
@PostConstruct
        ↓
Bean is ready
        ↓
Application runs
        ↓
Application shuts down
        ↓
@PreDestroy
        ↓
Bean is destroyed
```

After the Bean is created, for example:

- Load configuration
- Open a connection
- Initialize resources
- Perform setup

Before the Bean is destroyed, for example:

- Close a connection
- Release resources
- Cleanup

So Spring provides lifecycle hooks that allow us to run code during these stages.

```java
@Service
public class EmployeeService {

    public EmployeeService() {
        System.out.println("Constructor called"); // Spring creates the object, so the constructor runs.
    }

    @PostConstruct
    public void init() {
        System.out.println("Initialization"); // Runs after Spring has created the Bean and injected its dependencies,
                                              // before the Bean is ready for normal use.
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Destroying Bean"); // Runs when Spring is destroying the Bean, typically during application shutdown.
    }
}
```

- @PostConstruct: runs after Bean creation and dependency injection. Used for initialization.
- @PreDestroy: runs before Spring destroys the Bean. Used for cleanup.

> Note: For @PreDestroy, remember that Spring manages destruction of normal singleton Beans; prototype Beans are a special case because Spring does not manage their full destruction lifecycle.

> "The Spring Bean lifecycle starts when the Spring Container creates the Bean. Spring then injects its dependencies and calls the initialization callback, such as @PostConstruct. The Bean is then available for use. When the container shuts down, Spring calls the destruction callback, such as @PreDestroy, before destroying the Bean."

---

## 9. BeanFactory

BeanFactory is a Spring IoC container interface which creates and manages beans.

```java
@Component
public class EmployeeService {

    public void hello() {
        System.out.println("Hello Employee");
    }
}
```

```java
EmployeeService service =
        beanFactory.getBean(EmployeeService.class);

service.hello();
```

We are not creating the object ourselves with new EmployeeService(). We're asking the Spring container for the object it manages.

```text
IoC
 ↓
Spring Container
 ↓
BeanFactory
 ↓
Creates / manages / provides Beans
```

## 10. BeanFactory vs ApplicationContext

| BeanFactory | ApplicationContext |
|---|---|
| Basic IoC container | Advanced IoC container |
| Basic Bean management | Bean management + many additional features |
| More lightweight/basic | More commonly used |
| Core Spring | Used extensively in Spring Boot |

### ApplicationContext

ApplicationContext is an advanced Spring IoC container that is responsible for creating, managing, and providing Spring Beans, along with additional features such as events, internationalization, and resource handling.

You can access the ApplicationContext like this:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

Then you can retrieve a Bean:

```java
EmployeeService service =
        context.getBean(EmployeeService.class);
```

BeanFactory uses lazy initialization, which means the Bean may be created only when you actually request it. But by default, singleton Beans are generally created eagerly during ApplicationContext startup.
