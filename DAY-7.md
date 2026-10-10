# DAY-7-SB: N+1, Validation, DTO, Logging, Profiles, Swagger and Maven


## 1. N+1 Query Problem

### 1.1 First, understand the example

Imagine you work in a company that has an Employee Management System.

We have two database tables.

Employee table

| id | name | department_id |
|---|---|---|
| 1 | Rahul | 10 |
| 2 | Priya | 20 |
| 3 | Amit | 10 |

Department table

| id | department_name |
|---|---|
| 10 | IT |
| 20 | HR |

Notice that each employee has a department_id that tells us which department they belong to.

For example:

- Rahul → IT
- Priya → HR
- Amit → IT

Suppose we want to fetch all employees and print their department names.

How many employees do we have? 3.

### 1.2 How does Hibernate fetch the data?

Suppose you write:

```text
List<Employee> employees = employeeRepository.findAll();

for (Employee employee : employees) {
    System.out.println(employee.getDepartment().getDepartmentName());
}
```

Let's understand this code.

Step 1: Fetch all employees

```text
employeeRepository.findAll();
```

Hibernate executes a query similar to:

```text
SELECT * FROM employee;
```

This is 1 query.

It retrieves Rahul, Priya, and Amit.

Step 2: Get each employee's department name

```text
employee.getDepartment().getDepartmentName();
```

Suppose the relationship is configured for lazy loading, and the departments haven't already been fetched.

When the code accesses an employee's department, Hibernate may execute another query to retrieve that department.

For Rahul:

```text
SELECT * FROM department WHERE id = 10;
```

For Priya:

```text
SELECT * FROM department WHERE id = 20;
```

For Amit, Hibernate may reuse the already-loaded IT department from its first-level cache, so another query might not be needed.

For this example, the queries could be:

| Operation | Number of queries |
|---|---|
| Fetch all employees | 1 |
| Fetch Rahul's department | 1 |
| Fetch Priya's department | 1 |
| Fetch Amit's department | 0 additional queries if IT is cached |
| Total | 3 |

### 1.3 So, what is the N+1 Query Problem?

Imagine the application has 100 employees instead of 3.

Hibernate might execute:

1. 1 query to fetch all 100 employees.
2. 100 additional queries to fetch their departments.

That gives us:

```text
1 + 100 = 101 queries!
```

This is called the N+1 Query Problem.

- 1 = the initial query that fetches the employees.
- N = the additional queries executed while fetching related departments. In this example, N is 100.

👑 Remember: N+1 means one initial query plus potentially N additional queries for related data.

It does not mean 1 query is executed 100 times. It means one query fetches the main records, and many more queries may be executed to retrieve their related records.

### 1.4 Why is this a problem?

Imagine fetching 10,000 employees.

Instead of executing one query to fetch employees and efficiently retrieve their departments, Hibernate could execute thousands of additional queries.

That can cause:

- Slower API responses.
- More database work.
- Increased network communication between your application and database.

This becomes particularly important in real-world Spring Boot applications.

### 1.5 How do we solve it?

One common solution is JOIN FETCH.

Instead of fetching employees first and then retrieving each department separately, we tell Hibernate to fetch employees and their departments together.

```text
@Query("""
    SELECT e
    FROM Employee e
    JOIN FETCH e.department
""")
List<Employee> findEmployeesWithDepartment();
```

Here:

- Employee e refers to the Employee entity.
- JOIN FETCH fetches the associated department as part of the same query.
- e.department refers to the department relationship in the Employee entity.

Conceptually, Hibernate can generate SQL similar to:

```text
SELECT e.*, d.*
FROM employee e
JOIN department d ON e.department_id = d.id;
```

Now Hibernate can retrieve the employees and their departments in a single SQL query instead of issuing a separate query for each department.

Important: JOIN FETCH is one solution; the best approach depends on the relationship and data you need. Also, if you want employees without a department, a LEFT JOIN FETCH may be more appropriate.

Without JOIN FETCH:

```text
1 query    → Fetch 100 employees
100 queries → Fetch their departments

Total = 101 queries
```

With JOIN FETCH:

```text
1 query → Fetch employees and their departments together

Total = 1 query
```

"The N+1 Query Problem occurs when Hibernate executes one query to fetch a collection of entities and then executes additional queries to fetch related entities individually. This can cause performance issues. We can address it using techniques such as JOIN FETCH or @EntityGraph."

---

## 2. Validation in Spring Boot

Validation is the process of checking whether the data provided by a user is correct and meets certain rules before processing or saving it.

### 2.1 Example

Imagine you're building an Employee Management System with a registration API. A user sends this JSON to your Spring Boot application:

```text
{
  "name": "",
  "email": "abc",
  "age": -5
}
```

Here are three problems:

- name is empty.
- email is not a valid email address.
- age is negative.

Should your application save this employee in the database?

No! We should validate the input before saving it.

Validation works as follows:

```text
User sends request
        |
        v
Spring Boot Controller
        |
        v
    Validation
        |
        |---- Invalid data → Return error
        |
        |---- Valid data → Continue processing
                    |
                    v
              Service Layer
                    |
                    v
               Repository
                    |
                    v
                Database
```

### 2.2 Why do we need validation?

Three important reasons:

1. Data quality: Prevent invalid data from being saved.
2. Security and reliability: Reject unexpected input before processing it. Validation alone, however, does not replace security controls.
3. Better user experience: Return meaningful errors instead of allowing bad data to cause problems later.

---

## 3. Validation Annotations

### 3.1 @NotNull

It ensures that the value is not null.

Example:

```text
@NotNull
private String name;
```

Consider these values:

| Value | Valid? |
|---|---|
| null | ❌ No |
| "Gayathri" | ✅ Yes |
| "" (empty string) | ✅ Yes |
| " " (spaces only) | ✅ Yes |

This means name cannot be null.

Important: @NotNull checks only whether the value is null. It doesn't check whether a string is empty or contains only spaces.

### 3.2 @NotBlank

It ensures that a string is not null and contains at least one non-whitespace character.

```text
@NotBlank
private String name;
```

Consider these values:

| Value | Valid? |
|---|---|
| null | ❌ No |
| "" | ❌ No |
| " " | ❌ No |
| "Gayathri" | ✅ Yes |

### 3.3 @NotEmpty

@NotEmpty ensures that a value is not null and is not empty.

```text
@NotEmpty
private String name;
```

| Value | Valid? |
|---|---|
| null | ❌ No |
| "" | ❌ No |
| " " | ✅ Yes |
| "Gayathri" | ✅ Yes |

| Annotation | Rejects null | Rejects empty "" | Rejects spaces " " |
|---|---|---|---|
| @NotNull | ✅ | ❌ | ❌ |
| @NotEmpty | ✅ | ✅ | ❌ |
| @NotBlank | ✅ | ✅ | ✅ |

### 3.4 @Size

Suppose you want an employee's name to contain between 3 and 20 characters.

You can write:

```text
@Size(min = 3, max = 20)
private String name;
```

- min = 3 → At least 3 characters.
- max = 20 → At most 20 characters.

One important detail: @Size alone doesn't reject null. If the field is required, combine it with @NotBlank.

```text
@NotBlank
@Size(min = 3, max = 20)
private String name;
```

### 3.5 @Email

@Email checks whether a string has a valid email-like format.

| Email | Expected result |
|---|---|
| gayathri@gmail.com | ✅ Valid format |
| abc@company.com | ✅ Valid format |
| gayathri123 | ❌ Invalid format |
| abc.com | ❌ Invalid format |

Important: @Email checks the format, not whether the email address actually exists. Also, @Email alone does not reject null or an empty string. If the email is required, combine it with @NotBlank.

### 3.6 @Min and @Max

These annotations validate the minimum and maximum numeric values allowed for a field.

Example:

```text
@Min(18)
@Max(60)
private int age;
```

Let's understand each annotation.

- @Min(18) → The value must be at least 18.
- @Max(60) → The value must be at most 60.

```text
@Min(18)
private int age;
```

This checks the numeric value.

```text
@Size(min = 3, max = 10)
private String name;
```

This checks the length of the string.

---

## 4. @Valid

@Valid tells Spring to validate an object against the validation rules defined on its fields.

Validation rules will be whatever we define: @NotNull etc.

- @NotBlank
- @Email
- @Size
- @Min
- @Max

These annotations define the rules. @Valid triggers their evaluation when used on a supported controller request parameter.

### 4.1 Without @Valid

Suppose you have an Employee class:

```text
public class Employee {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

These annotations define validation rules:

- name cannot be blank.
- email must have a valid email-like format.

Now imagine your controller:

```text
@PostMapping("/employees")
public String createEmployee(@RequestBody Employee employee) {
    return "Employee created";
}
```

Problem: without @Valid, Spring MVC does not automatically trigger these Bean Validation rules just because they are present on the fields.

The request could contain invalid data without these field constraints being checked at this point.

### 4.2 With @Valid

We add @Valid to the request body parameter:

```text
@PostMapping("/employees")
public String createEmployee(@Valid @RequestBody Employee employee) {
    return "Employee created";
}
```

Now Spring validates the employee's fields before proceeding with the controller method.

If the data is valid: the method proceeds.

If the data is invalid: Spring MVC normally raises a validation exception, and the request is rejected before the method body executes. We can customise the error response later.

### 4.3 Let's test it

```text
Client sends invalid JSON
        |
        v
Spring converts JSON to Java object
        |
        v
      @Valid
        |
        v
  Validation fails
        |
        v
MethodArgumentNotValidException
        |
        v
Return an error response
```

Quiz: your controller uses @Valid, and a user submits an invalid email address. Which exception is commonly raised by Spring MVC for this invalid @RequestBody?

A) NullPointerException
B) MethodArgumentNotValidException
C) SQLException
D) IOException

Answer: B) MethodArgumentNotValidException

---

## 5. @ExceptionHandler

For example, let's handle an employee-not-found exception.

```text
@ExceptionHandler(EmployeeNotFoundException.class)
public ResponseEntity<String> handleEmployeeNotFound(
        EmployeeNotFoundException ex) {

    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ex.getMessage());
}
```

Let's understand the important parts.

| Code | Meaning |
|---|---|
| @ExceptionHandler(...) | Specifies which exception this method handles |
| EmployeeNotFoundException.class | The exception type to handle |
| EmployeeNotFoundException ex | Receives the exception object |
| HttpStatus.NOT_FOUND | Sets HTTP status to 404 |
| ex.getMessage() | Retrieves the exception's message |

If the exception message is "Employee not found", the response body contains that message.

The HTTP response would look conceptually like this:

```text
HTTP/1.1 404 Not Found
Content-Type: text/plain

Employee not found
```

---

## 6. @RestControllerAdvice

@RestControllerAdvice provides centralised exception handling for REST controllers.

For example:

```text
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<String> handleEmployeeNotFound(
            EmployeeNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ex.getMessage());
    }
}
```

This class can handle EmployeeNotFoundException from multiple controllers.

Conceptually:

```text
EmployeeController ────┐
                       │
DepartmentController ──┼──→ GlobalExceptionHandler
                       │
SalaryController ──────┘
```

Instead of duplicating exception-handling logic across the controllers, you centralise it.

---

## 7. Custom Validation

Custom validation means creating your own validation rule when the built-in annotations don't meet your requirements.

Example: suppose a company ID has to start with "ID-". Then we have to use custom validation.

---

## 8. DTO (Data Transfer Object)

A DTO is a Java object used to transfer data between different parts of an application, or between an application and its client, without exposing the entire entity.

```text
     Database
         |
         v
   Employee Entity
   (All DB fields)
         |
         v
Convert to DTO [getters, setters]
         |
         v
 EmployeeResponseDTO
 (Only selected fields)
         |
         v
Frontend [json response]
```

### Example

Imagine an Employee Management System. Suppose you have an Employee entity that represents a database table.

```text
@Entity
public class Employee {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private String email;

    private Double salary;

    private String password;
}
```

Imagine an employee has these details in the database:

| ID | Name | Email | Salary | Password |
|---|---|---|---|---|
| 101 | Gayathri | gayathri@gmail.com | 80000 | secret123 |

Now, suppose your frontend application requests the employee's details.

Should your API return every field, including the password?

Absolutely not! Passwords and other sensitive information should not be exposed in API responses.

We need a way to return only the information the client is supposed to see.

That's where a DTO helps.

### Create an Employee DTO

Let's create a separate class called EmployeeResponseDTO.

```text
public class EmployeeResponseDTO {

    private Long id;
    private String name;
    private String email;
    private Double salary;

    // Getters and setters
}
```

Notice that we haven't included the password.

Now the API can return:

```text
{
  "id": 101,
  "name": "Gayathri",
  "email": "gayathri@gmail.com",
  "salary": 80000
}
```

The password is not included in the response because our DTO doesn't contain that field.

Important: A DTO helps control what data you expose, but it isn't a complete security mechanism by itself.

Entity = Represents database data.
DTO = Represents data transferred between application components or to/from a client.

| Feature | Entity | DTO |
|---|---|---|
| Purpose | Represents database data | Transfers selected data |
| Database mapping | Usually mapped using @Entity | Not normally mapped as a database entity |
| Fields | Reflects the data model | Contains only the fields needed for a particular operation |
| Sensitive information | May contain sensitive fields | Can exclude sensitive fields |
| Used in API requests/responses | Can be, but often avoided in well-structured APIs | Commonly used |

Quick quiz: suppose your Employee entity has a salary field, but your EmployeeResponseDTO doesn't.

When you return the DTO, will the salary normally appear in the JSON response?

A) Yes, because it exists in the entity.
B) No, because the DTO doesn't contain the salary field.
C) Yes, because Spring automatically copies every entity field into the DTO.

Answer: B

---

## 9. Where should we convert Entity into DTO?

When retrieving an employee, the flow is:

1. The Controller receives the request.
2. The Service asks the Repository to fetch the employee.
3. The Repository retrieves the entity from the database.
4. The Service converts the entity into a DTO.
5. The Controller returns the DTO to the client.

Conceptually:

```text
Database
    |
    v
Employee Entity
    |
    v
Service converts Entity → DTO
    |
    v
Controller returns DTO
    |
    v
JSON response
```

Why do we usually convert in the Service layer?

The Service layer handles business logic and coordinates operations. Keeping entity-to-DTO mapping there, or in a dedicated mapper used by the service, helps keep the Controller focused on handling HTTP requests and responses.

There isn't one mandatory location for every mapping operation, but service-layer mapping or a dedicated mapper is a common approach.

### Request DTO vs Response DTO

| Feature | Request DTO | Response DTO |
|---|---|---|
| Direction | Client → Server | Server → Client |
| Purpose | Accept input | Return output |
| ID | Usually omitted when generated by the server | Usually included |
| Validation | Often includes @NotBlank, @Email, etc. | Usually focuses on response data |
| Sensitive fields | Only accepts the fields the operation needs | Excludes sensitive information |

---

## 10. Dependencies

| No. | Dependency | Maven dependency name | Why do we need it? |
|---|---|---|---|
| 1 | Spring Web | spring-boot-starter-web | Build REST APIs using @RestController, @GetMapping, @PostMapping, etc. |
| 2 | Spring Data JPA | spring-boot-starter-data-jpa | Use JpaRepository, CRUD operations, custom queries, and entity relationships. |
| 3 | Validation | spring-boot-starter-validation | Use @NotBlank, @Email, @Size, @Min, @Max, @Valid, and custom validation. |

| Concept | Separate dependency required? |
|---|---|
| IoC and Dependency Injection | ❌ No; provided by Spring |
| Beans and Bean Lifecycle | ❌ No |
| @SpringBootApplication | ❌ No |
| application.properties and application.yml | ❌ No |
| REST API annotations | ❌ No additional dependency beyond Spring Web |
| JpaRepository | ❌ No additional dependency beyond Spring Data JPA |
| Hibernate | ❌ Usually brought in by Spring Data JPA |
| DTOs | ❌ No |
| @ExceptionHandler | ❌ No additional dependency beyond Spring Web |
| @RestControllerAdvice | ❌ No additional dependency beyond Spring Web |
| Built-in validation annotations | ✅ Spring Boot Validation starter |
| Custom validation | ❌ No additional dependency beyond the validation starter |

---

## 11. Logging

Logging is the process of recording information about what an application is doing while it runs. Logging is especially important in production applications because it helps with debugging, monitoring, and troubleshooting.

A typical Spring Boot application uses:

- SLF4J — a logging facade that provides a common API for writing log messages.
- Logback — the default logging implementation in a typical Spring Boot setup.

| Feature | System.out.println() | Logging |
|---|---|---|
| Print messages | ✅ | ✅ |
| Different severity levels | ❌ | ✅ |
| Configure which messages appear | ❌ Not built in | ✅ |
| Configure output destinations | ❌ Not built in | ✅ |
| Include timestamps and contextual information | ❌ Not built in | ✅ |
| Commonly used in production applications | Generally avoided for application diagnostics | ✅ |

### Different types of log

| Log level | Purpose | Example |
|---|---|---|
| TRACE | Extremely detailed diagnostic information | Tracing each step inside a complex operation |
| DEBUG | Detailed information useful for debugging | Checking the employee ID received by a service |
| INFO | Normal application events | Employee created successfully |
| WARN | Something unexpected that may need attention | An API is taking longer than expected |
| ERROR | A failure that prevents an operation from completing successfully | Database connection failed |

Order of log:

```text
TRACE
  ↓
DEBUG
  ↓
INFO
  ↓
WARN
  ↓
ERROR
```

How does this ordering work?

Suppose you configure your application to display logs at the INFO level.

The application will generally display:

- INFO ✅
- WARN ✅
- ERROR ✅

But it won't display:

- TRACE ❌
- DEBUG ❌

### SLF4J

SLF4J stands for Simple Logging Facade for Java.

It provides a common interface for writing log messages without tying your application code to one specific logging implementation. These methods are provided through the SLF4J API. It is the standard interface for logging.

### Logback

Logback is a logging framework that implements the SLF4J API.

It handles the actual logging work, such as:

- Processing log messages.
- Filtering messages according to configured log levels.
- Writing logs to the console or files.
- Applying formatting and other logging configurations.

```text
   Your Java Code
         |
         v
       SLF4J
    (Logging API)
         |
         v
      Logback
  (Implementation)
         |
         v
 Console / Log Files
```

| Feature | SLF4J | Logback |
|---|---|---|
| Type | Logging facade/API | Logging implementation |
| Main purpose | Provides methods for writing logs | Processes and outputs logs |
| Writes log messages through its API | Yes | Implements the logging backend |
| Controls file output and formatting | Not by itself | Yes |
| Default in typical Spring Boot applications | Common logging API | Common default implementation |

Spring Boot commonly uses them together, so you can write logging code through SLF4J while Logback handles the actual output.

Quiz: in a typical Spring Boot application, which component is responsible for actually processing log messages and writing them to the console or log files?

A) SLF4J
B) Logback
C) @RestController
D) Spring Data JPA

Answer: B

### SLF4J usage

```text
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {

    private static final Logger logger =
            LoggerFactory.getLogger(EmployeeService.class);

    public void createEmployee() {
        logger.info("Creating employee");
        logger.info("Employee created successfully");
    }
}
```

| Code | Meaning |
|---|---|
| Logger | Interface used to write log messages |
| LoggerFactory | Creates a logger |
| getLogger(EmployeeService.class) | Associates the logger with EmployeeService |
| private | Restricts access to the class |
| static | Shares one logger reference across instances of the class |
| final | Prevents reassignment of the logger reference |

Other log levels:

```text
logger.debug("Fetching employee details");
logger.warn("Employee record was not found");
logger.error("Failed to connect to the database");
```

Notice this part:

```text
logger.info("Fetching employee with ID {}", id);
```

The {} is a placeholder. SLF4J substitutes it with the value of id.

If id is 101, the log message looks like:

```text
INFO - Fetching employee with ID 101
```

This is called parameterized logging. It's generally preferable to concatenating strings in log statements.

### Logging configuration

In Spring Boot, we can configure which log levels appear using application.properties.

```text
logging.level.root=INFO
logging.level.com.example=DEBUG
```

- logging.level.root=INFO — show INFO, WARN, and ERROR logs across the application.
- logging.level.com.example=DEBUG — show DEBUG and higher-level logs for classes in the com.example package.

This enables DEBUG-level logging for classes in the com.example.service package and its subpackages, without changing the root logging level.

```text
logging.level.root=INFO
logging.level.com.example.service=DEBUG
```

- Root level = INFO: the rest of the application displays INFO, WARN, and ERROR logs.
- Service package = DEBUG: classes in that package display DEBUG, INFO, WARN, and ERROR logs.

### 2 ways to create Logger

#### 1. Without an annotation — SLF4J

This is the approach we just learned.

```text
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class EmployeeService {

    private static final Logger logger =
            LoggerFactory.getLogger(EmployeeService.class);

    public void createEmployee() {
        logger.info("Creating employee");
    }
}
```

Here, we explicitly create the logger using LoggerFactory.

Advantages:

- No additional library is required.
- You have direct control over logger creation.
- It's a standard approach that works well in Spring Boot.

#### 2. With an annotation — Lombok's @Slf4j

Yes, we can use an annotation to avoid writing the logger declaration manually!

First, add Lombok to your project if it isn't already included.

Maven dependency:

```text
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

Then write:

```text
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class EmployeeService {

    public void createEmployee() {
        log.info("Creating employee");
        log.error("Employee creation failed");
    }
}
```

That's it! Notice that we don't declare Logger or call LoggerFactory.

How does @Slf4j work?

When Lombok processes this annotation, it generates a logger field for your class, conceptually equivalent to:

```text
private static final Logger log =
        LoggerFactory.getLogger(EmployeeService.class);
```

The generated logger uses SLF4J, and Logback can handle the messages as the logging implementation.

The annotation doesn't replace SLF4J or Logback. It simply reduces the amount of code you need to write.

---

## 12. Profiles and Configuration

A Spring Boot Profile allows you to define different configurations for different environments.

### How do we implement profiles?

Step 1: Create application.properties

Location:

```text
src/main/resources/application.properties
```

Add:

```text
spring.profiles.active=dev
```

This tells Spring Boot to activate the dev profile by default.

Step 2: Create application-dev.properties

```text
server.port=8081
logging.level.root=DEBUG
```

When the dev profile is active, the application uses port 8081 and enables DEBUG-level logging.

Step 3: Create application-prod.properties

```text
server.port=8080
logging.level.root=INFO
```

When the prod profile is active, the application uses port 8080 and enables INFO-level logging.

Your project structure will look like this:

```text
src/main/resources/
├── application.properties
├── application-dev.properties
└── application-prod.properties
```

### How do you switch profiles?

You can activate a profile in several ways.

Option 1: In application.properties

```text
spring.profiles.active=dev
```

Option 2: When running the application

```text
java -jar employee-app.jar --spring.profiles.active=prod
```

This activates the prod profile for that application run, overriding the active profile specified in the packaged configuration.

Important: avoid hardcoding production credentials in configuration files committed to Git. Use environment variables or a secrets-management solution for sensitive values.

### @Profile

@Profile is used to load different Java classes or beans depending on the active environment.

Example:

```text
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

@Service
@Profile("dev")
public class MockPaymentService {

    public void processPayment() {
        System.out.println("Processing mock payment");
    }
}
```

Here:

- @Service registers the class as a Spring bean.
- @Profile("dev") tells Spring to register this bean only when the dev profile is active.

You could create a separate production implementation:

```text
@Service
@Profile("prod")
public class RealPaymentService {

    public void processPayment() {
        System.out.println("Processing real payment");
    }
}
```

Important: in a real application, ensure the appropriate implementation is selected and that payment processing is properly secured.

| Annotation/property | Purpose |
|---|---|
| @Profile("dev") | Register a bean only when the dev profile is active. |
| @Profile("prod") | Register a bean only when the prod profile is active. |
| spring.profiles.active=dev | Activate the dev profile. |

Active profile: an active profile is the profile Spring Boot currently uses to determine which profile-specific configurations and beans should be loaded.

---

## 13. @Value and @ConfigurationProperties

These are 2 ways to read values from configuration files into Java classes.

### A. @Value

Definition: @Value injects an individual property value into a Spring-managed bean.

Suppose application.properties contains:

```text
app.name=Employee Management System
app.version=1.0
```

You can read the application name like this:

```text
@Component
public class AppInfo {

    @Value("${app.name}")
    private String appName;
}
```

Spring retrieves the value of app.name and injects it into appName.

The value will be:

```text
Employee Management System
```

Use @Value when you need to inject a small number of individual properties.

### B. @ConfigurationProperties

Definition: @ConfigurationProperties binds a group of related configuration properties to a Java object.

Suppose you have:

```text
app.name=Employee Management System
app.version=1.0
app.owner=Engineering
```

Instead of injecting each property individually, you can group them.

```text
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private String version;
    private String owner;

    // Getters and setters
}
```

Spring binds the properties to the corresponding fields.

For example:

- app.name → name
- app.version → version
- app.owner → owner

---

## 14. Swagger / OpenAPI

Swagger is a collection of tools used to document, visualize, and interact with REST APIs.

OpenAPI is a specification that describes how a REST API works in a standardized format.

Swagger is a collection of tools that work with the OpenAPI specification.

- OpenAPI Specification: Describes your API's endpoints, parameters, request bodies, and responses.
- Swagger UI: Displays the API documentation in a browser and lets you try API requests.
- Swagger Editor: Helps you write and edit OpenAPI definitions.

Interview tip: OpenAPI is the specification; Swagger UI is a tool that presents that specification interactively.

### 14.1 Why do we need Swagger?

Imagine you create this endpoint:

```text
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable Long id) {
    return employeeService.getEmployee(id);
}
```

Without API documentation, another developer might need to inspect your code to understand how to call this endpoint.

With Swagger UI, the developer can see something like this:

GET /employees/{id}

- Description: Retrieve an employee by ID.
- Parameter: id — employee ID.
- Example input: 101.
- Response: Employee details in JSON format.

Swagger UI can also let the developer enter 101, click Execute, and send the request to your running application.

This makes API development and testing easier.

Main benefits:

1. Automatic documentation: API documentation can be generated from your Spring application.
2. Interactive testing: Send requests directly from Swagger UI.
3. Better collaboration: Frontend and backend developers can understand the API contract.
4. Less manual documentation: You don't have to maintain a separate document for every endpoint.

One important distinction: Swagger UI doesn't automatically implement your business logic. Your Spring Boot endpoints still need to work correctly.

### 14.2 How does Swagger work in Spring Boot?

The typical flow is:

```text
Spring Boot REST Controllers
            |
            v
      springdoc-openapi
            |
            v
    OpenAPI Specification
            |
            v
        Swagger UI
            |
            v
Developer views and tests APIs
```

For example, your controller contains:

```text
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public String getEmployee(@PathVariable Long id) {
        return "Employee ID: " + id;
    }
}
```

A compatible OpenAPI integration can discover the endpoint and expose its documentation.

When the application runs, you can open Swagger UI in your browser and explore the available endpoints.

### 14.3 How do we add Swagger to Spring Boot?

For a typical Spring Boot application, we can use springdoc-openapi.

It integrates OpenAPI documentation with Spring MVC and provides Swagger UI.

For a Spring Boot 3.x application using the standard Spring MVC stack, add this Maven dependency to pom.xml:

```text
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.13</version>
</dependency>
```

This version is an example; use a version compatible with your Spring Boot version and project requirements.

After adding the dependency, Maven downloads the required libraries.

You generally don't need to create a separate Swagger configuration class just to get basic API documentation.

What happens next?

Start your Spring Boot application and open:

Swagger UI

```text
http://localhost:8080/swagger-ui/index.html
```

OpenAPI JSON

```text
http://localhost:8080/v3/api-docs
```

The first URL displays the interactive documentation. The second returns the API specification in JSON format.

These URLs assume your application runs locally on port 8080 and uses the default springdoc paths.

---

## 15. Maven

Apache Maven is a build automation and dependency management tool primarily used for Java projects.

### Maven has three main responsibilities

1. Dependency management

Downloads and manages the libraries your application needs.

For example, you can declare Spring Web in your project, and Maven resolves its required dependencies.

2. Build automation

Automates tasks such as:

- Compiling Java source code.
- Running tests.
- Packaging your application into a JAR file.

3. Project standardization

Uses a standard project structure and configuration format, making Java projects easier to understand and maintain.

### 15.1 POM.xml

POM means Project Object Model.

The pom.xml file contains information about your Maven project, including its dependencies, build configuration, and project coordinates.

Example pom.xml

Let's look at a simplified example:

```text
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>employee-management</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <!-- Project dependencies go here -->
    </dependencies>
</project>
```

Let's understand the important fields.

| Element | Meaning |
|---|---|
| modelVersion | Specifies the POM model version. |
| groupId | Identifies the organization or project group. |
| artifactId | Identifies this particular project or artifact. |
| version | Specifies the project's version. |
| dependencies | Lists the libraries the project needs. |

### 15.2 Dependency

A dependency is an external library or module that your project needs to use certain functionality.

You can declare Spring Web as a Maven dependency:

```text
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Let's understand each part.

- dependency: Declares a library that the project needs.
- groupId: Identifies the library's publisher or group.
- artifactId: Identifies the specific library.

What happens when we add a dependency?

```text
You add a dependency to pom.xml
            |
            v
      Maven resolves it
            |
            v
   Downloads required JARs
            |
            v
 Libraries become available
      to your project
```

### 15.3 Maven Repository

Maven needs a place to download dependencies from. It uses repositories to store and retrieve project artifacts, such as JAR files.

| Repository | Purpose |
|---|---|
| Local repository | Stores downloaded dependencies on your computer. |
| Central repository | A shared repository from which Maven can download publicly available artifacts. |
| Remote repository | An additional repository hosted elsewhere, such as a company's internal artifact repository. |

Example: when Maven downloads Spring Web, it normally retrieves the required artifacts from a remote repository and caches them in your local repository.

On a typical computer, the local repository is located at:

```text
~/.m2/repository
```

On Windows, it is usually:

```text
C:\Users\<username>\.m2\repository
```

Maven can reuse cached artifacts rather than downloading them again whenever possible.

### 15.4 Maven Build Lifecycle

A Maven build lifecycle is a sequence of phases that Maven executes to build, test, and package your application.

| Command | Purpose |
|---|---|
| mvn validate | Checks that the project structure and required information are valid. |
| mvn compile | Compiles the main Java source code. |
| mvn test | Runs tests after compiling the relevant code. |
| mvn package | Packages the compiled application, typically into a JAR or WAR. |
| mvn install | Runs the preceding default lifecycle phases and installs the artifact into your local repository. |
| mvn clean | Removes generated build files, usually from the target directory. |
| mvn clean install | Cleans the previous build, then builds, tests, packages, and installs the artifact. |

Example

Suppose you run:

```text
mvn clean install
```

Maven first removes the previous build output and then executes the required build phases through install.

The simplified flow is:

```text
clean
  |
  v
validate
  |
  v
compile
  |
  v
test
  |
  v
package
  |
  v
install
```

Important: clean belongs to a separate lifecycle. The other phases shown belong to Maven's default lifecycle.

- Dependency = functionality your application uses.
- Plugin = tool that performs a build-related task.

### 15.5 Plugins

Plugins perform tasks during the build process, such as compiling code, running tests, or packaging a Spring Boot application.

For example, the Spring Boot Maven Plugin helps package and run your Spring Boot application.

### 15.6 Maven Wrapper

It provides scripts to run Maven for the project without requiring a separate Maven installation.

Common files include:

- mvnw — for Linux and macOS.
- mvnw.cmd — for Windows.
- .mvn/wrapper/ — contains wrapper configuration files.
