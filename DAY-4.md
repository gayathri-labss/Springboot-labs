# Day 4 – Spring Boot: REST Controller Annotations


## Annotations Covered

**REST Controller annotations**
1. @RestController
2. @RequestMapping
3. @GetMapping
4. @PostMapping
5. @PutMapping
6. @DeleteMapping

**Request data annotations**
1. @PathVariable
2. @RequestParam
3. @RequestBody

**Response**
1. ResponseEntity

---

## 1. @RestController

@RestController: This class will handle REST API requests and return the response directly to the client.

@Controller: A **Controller** is a class that receives HTTP requests and decides which method should handle them.

```text
Client
  │
  │ GET /employees
  ↓
Controller
  │
  ↓
Service
  │
  ↓
Repository
  │
  ↓
Database
```

```text
@RestController = @Controller + @ResponseBody
```

- @Controller – "This class is a web controller."
- @ResponseBody – "Take the return value of this method and put it directly into the HTTP response body."

### Example

```text
@RestController
public class EmployeeController {

    @GetMapping("/employees")
    public String getEmployees() {
        return "Employee List";
    }
}
```

### Flow

```text
Client
  │
  │ GET /employees
  ↓
EmployeeController
  │
  │ @GetMapping("/employees")
  ↓
getEmployees()
  │
  │ "Employee List"
  ↓
HTTP Response Body
```

**Request:**

```text
GET /employees
```

### Difference between @Controller and @RestController

**@Controller**: Typically used with Spring MVC when returning **views/pages**.

```text
@Controller
public class EmployeeController {

    @GetMapping("/employees")
    public String employees() {
        return "employees";
    }
}
```

Here "employees" can represent a view/template.

**@RestController**: "@RestController is a Spring annotation used to define a REST controller. It combines @Controller and @ResponseBody, so the methods in the class handle HTTP requests and their return values are written directly to the HTTP response body, typically as JSON."

```text
@RestController
public class EmployeeController {

    @GetMapping("/employees")
    public List<Employee> employees() {
        return employeeService.getEmployees();
    }
}
```

Here the return value becomes the **HTTP response body**, commonly JSON.

> **Remember:** HTTP method as well as path always need to match.
> Example: if we have @GetMapping("/employees"), it expects GET + /employees.

### Example – returning an object

```text
@RestController
public class EmployeeController {

    @GetMapping("/employee")
    public Employee getEmployee() {
        return new Employee(1L, "Rahul");
    }
}
```

- Client sends: GET /employee
- Java method returns: new Employee(1L, "Rahul")
- But the client doesn't receive a Java object.

It receives JSON:

```text
{
  "id": 1,
  "name": "Rahul"
}
```

**So what happened?**

```text
Java Object
    ↓
Spring
    ↓
JSON
    ↓
HTTP Response
    ↓
Client
```

The conversion from the Java object to JSON is commonly handled by **Jackson**.

---

## 2. @RequestMapping

@RequestMapping is a Spring annotation used to **map an HTTP request to a controller or controller method**.

```text
Request
   ↓
HTTP request from the client

Mapping
   ↓
Connecting that request to Java code
```

@RequestMapping(value = "/employees", method = RequestMethod.GET) & @GetMapping("/employees") both are the same. Both endpoints perform GET.

### Compare these two

**Longer form:**

```text
@RequestMapping(
    value = "/employees",
    method = RequestMethod.GET
)
public String getEmployees() {
    return "Employees";
}
```

**Shorter form:**

```text
@GetMapping("/employees")
public String getEmployees() {
    return "Employees";
}
```

Both express the same basic mapping:

```text
GET + /employees
```

The second one is simply much cleaner.

### Example – class level + multiple HTTP methods

```text
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public String getEmployees() {
        return "Get";
    }

    @PostMapping
    public String createEmployee() {
        return "Post";
    }
}
```

### Example – class level + method level

```text
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @RequestMapping("/details")
    public String getDetails() {
        return "Employee Details";
    }
}
```

So the HTTP request will be for /employees/details.

Hence, **@RequestMapping works both at class level as well as method level.**

> **NOTE:**
> **If @GetMapping already exists, why do we need @RequestMapping?**
> Because @RequestMapping is **more general** and can be used for different HTTP methods, while @GetMapping specifically represents GET.

---

## 3. @GetMapping

@GetMapping is a Spring annotation used to **map a GET HTTP request to a Java method**.

Example: *"When a GET request comes to /employees, map it to this Java method."*

### What if we have two methods?

```text
@RestController
public class EmployeeController {

    @GetMapping("/employees")
    public String getEmployees() {
        return "All Employees";
    }

    @GetMapping("/departments")
    public String getDepartments() {
        return "All Departments";
    }
}
```

Now Spring knows:

```text
GET /employees
      ↓
getEmployees()

GET /departments
      ↓
getDepartments()
```

Because each @GetMapping is attached to a **different method**.

> **NOTE:** The annotation is placed above a method because it tells Spring which HTTP request should trigger that particular method.

### @GetMapping with a class-level @RequestMapping

```text
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public String getEmployees() {
        return "Employees";
    }
}
```

```text
Class path:  /employees
Method path: [nothing]
        ↓
Final path:  /employees
```

So this request will call getEmployees().

### Path combinations

```text
@GetMapping("/employees")
```

→ GET /employees

and:

```text
@RequestMapping("/employees")
@GetMapping
```

→ GET /employees

and:

```text
@RequestMapping("/employees")
@GetMapping("/all")
```

→ GET /employees/all

---

## 4. @PostMapping

**POST** is generally used when the client wants to **send data to the server to create a new resource**.

## 5. @RequestBody

A request body is the part of an HTTP request that contains the data being sent from the client to the server.

### Client sends this request

```text
POST /employees
Content-Type: application/json

{
  "name": "Ananya",
  "department": "IT"
}
```

Here:

```text
POST /employees
        ↓
Request URL + method

{
  "name": "Ananya",
  "department": "IT"
}
        ↓
Request Body
```

Now in Spring Boot, we use @RequestBody to receive that data:

```text
@PostMapping("/employees")
public String createEmployee(@RequestBody Employee employee) {
    return "Employee Created";
}
```

The important part is:

```text
@RequestBody Employee employee
```

It means:

> Take the data from the HTTP request body and convert it into an Employee object, then give it to the employee parameter.

**"Take the request body and give it to me as an Employee object called employee."**

### How is this performed?

When the user clicks **Create**, the frontend application takes those values and creates a JSON object:

```text
{
  "name": "Ananya",
  "department": "IT"
}
```

Then the frontend sends an HTTP request to your backend:

```text
POST /employees
Content-Type: application/json

{
  "name": "Ananya",
  "department": "IT"
}
```

### FLOW

```text
User fills form
      ↓
Frontend application
      ↓
Creates JSON
      ↓
HTTP POST request
      ↓
Spring Boot API
      ↓
@RequestBody
      ↓
Employee Java object
      ↓
Service
      ↓
Repository
      ↓
Database
```

**Jackson** is the library commonly used by Spring Boot to convert between JSON and Java objects.

- For request data, JSON → Java object is called **deserialization**.
- For response data, Java object → JSON is called **serialization**.

```text
Client
   │
   │ POST /employees
   │
   │ JSON
   │ {
   │   "name": "Ananya"
   │ }
   ↓
Spring Boot
   │
   │ @RequestBody
   ↓
Jackson
   │
   │ JSON → Java
   ↓
Employee object
   ↓
createEmployee(employee)
```

**@RequestBody**
→ tells Spring to take data from the HTTP request body

**Jackson**
→ performs the JSON ↔ Java object conversion

---

## 6. @PutMapping

**PUT** is generally used to **update/replace an existing resource**.

Example:

```text
@PutMapping("/employees/10")
public String updateEmployee() {
    return "Employee Updated";
}
```

**"When a PUT request comes to /employees/10, execute updateEmployee()."**

```text
@PutMapping("/employees/10")
public String updateEmployee() {
    return "Updated";
}
```

**What determines that PUT /employees/10 should call updateEmployee()?**

- A. updateEmployee method name
- B. @PutMapping("/employees/10")
- C. "Updated"
- D. String

**Answer is B**

---

## 7. @DeleteMapping

**This Java method should handle an HTTP DELETE request.**

Example:

```text
@DeleteMapping("/employees")
public String deleteEmployee() {
    return "Deleted";
}
```

### How does delete work?

```text
User clicks Delete
      ↓
Frontend
      ↓
DELETE /employees/10
      ↓
Spring Controller
      ↓
@PathVariable gets 10
      ↓
Service
      ↓
Repository
      ↓
Database
      ↓
DELETE employee WHERE id = 10
```

---

## 8. @PathVariable

**@PathVariable is used to extract a value from the URL and give it to a Java method parameter.**

Example:

```text
@DeleteMapping("/employees/{id}")
public String deleteEmployee(@PathVariable Long id) {
    return "Employee Deleted";
}
```

So Spring does this:

```text
URL
/employees/10
      ↓
{ id = 10 }
      ↓
@PathVariable Long id
      ↓
Java variable id = 10
```

Example (explicit name):

```text
@GetMapping("/employees/{employeeId}")
public String getEmployee(@PathVariable("employeeId") Long id) {
    ...
}
```

*(The above and below are the same for @PathVariable, but the way it's called is different.)*

---

## 9. @RequestParam

@RequestParam is used to get a value from the query parameter of a URL and put it into a Java method parameter.

Example:

```text
@GetMapping("/employees")
public String getEmployees(@RequestParam String department) {
    return department;
}
```

If client sends: GET /employees?department=IT

- Spring takes: department=IT
- And puts: department = "IT"
- So the method returns: IT

### Multiple query parameters

We can have multiple query parameters.

```text
@GetMapping("/employees")
public String getEmployees(
        @RequestParam String department,
        @RequestParam String city) {

    return department + " " + city;
}
```

Spring gets:

```text
department = "IT"
city = "Bangalore"
```

### Difference between @PathVariable and @RequestParam

- **@PathVariable**: Used when the value is part of the **URL path** and usually identifies a specific resource.
- **@RequestParam**: Used when the value is a **query parameter**, commonly for filtering, searching, sorting, pagination, etc.

```text
@PathVariable
/employees/25
           ↑
      WHICH employee?


@RequestParam
/employees?department=IT
                    ↑
              WHAT filter?
```

**Real-time example:**

@PathVariable – the number identifies **which specific resource** you want.

```text
GET /orders/123
GET /customers/456
GET /products/789
GET /accounts/1001
```

@RequestParam – based on filtering etc.

```text
/products?category=shoes&size=9
```

---

## 10. ResponseEntity

ResponseEntity represents the complete HTTP response that your Spring controller sends back to the client.

ResponseEntity lets your controller control the **HTTP response**, especially:

- **Status code**
- **Response body**
- **Headers** (when needed)

### One important thing: why not just return Employee?

You might have:

```text
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable Long id) {
    return employeeService.getEmployee(id);
}
```

This is perfectly fine when the operation is straightforward.

But with:

```text
@GetMapping("/employees/{id}")
public ResponseEntity<Employee> getEmployee(@PathVariable Long id) {

    Employee employee = employeeService.getEmployee(id);

    if (employee == null) {
        return ResponseEntity.notFound().build();
    }

    return ResponseEntity.ok(employee);
}
```

you can tell the client:

> "The employee exists → **200**"

or

> "The employee doesn't exist → **404**"

### Why is it useful?

Because APIs need to communicate what happened.

| Situation | HTTP status |
|---|---|
| Request successful | 200 OK |
| Employee created | 201 Created |
| Resource not found | 404 Not Found |
| Bad request | 400 Bad Request |
| Unauthorized | 401 Unauthorized |
| Forbidden | 403 Forbidden |
| Server error | 500 Internal Server Error |

```text
          Backend
             ↓
     Did employee exist?
        /          \
      YES           NO
       ↓             ↓
ResponseEntity.ok()  notFound()
       ↓             ↓
    200 OK          404
       ↓             ↓
  Employee JSON   No employee
```

But ResponseEntity lets you explicitly say:

> "The operation succeeded, and here is the data."

or:

> "The requested employee doesn't exist."

---

## Controller (Summary)

A **Controller** is the layer in a Spring Boot application that **receives requests from the client and sends responses back to the client**. It is for MVC.

```text
UI / Frontend
      ↓
   Request
      ↓
 Controller
      ↓
   Service
      ↓
 Repository
      ↓
  Database
```

The Controller is **not normally responsible for business logic or directly talking to the database**.

@RestController: Combination of controller and response body.

### @Controller vs @RestController

| Feature | @Controller | @RestController |
|---|---|---|
| **Purpose** | Handles web requests, usually for returning **views/pages** | Handles **REST API requests** and returns data |
| **Used for** | MVC web applications | REST APIs / backend APIs |
| **Response** | Usually returns a **view name** | Usually returns **JSON/data** |
| **Example return** | "employees" | Employee object / List<Employee> |
| **@ResponseBody needed?** | Usually yes, if you want to return data directly | **No** |
| **Equivalent to** | @Controller | @Controller + @ResponseBody |
| **Common with** | Thymeleaf, JSP, server-side HTML | React, Angular, mobile apps, Postman, other clients |
| **Typical URL** | /employees | /api/employees |
| **Main purpose** | Return a webpage | Return API data |

### Example 1 – @Controller

```text
@Controller
public class EmployeeController {

    @GetMapping("/employees")
    public String employees() {
        return "employees";
    }
}
```

Here:

```text
GET /employees
      ↓
Controller
      ↓
"employees"
      ↓
employees.html
```

So the browser gets a **webpage/view**.

### Example 2 – @RestController

```text
@RestController
public class EmployeeController {

    @GetMapping("/employees")
    public List<Employee> employees() {
        return employeeService.getEmployees();
    }
}
```

The response could be:

```text
[
  {
    "id": 101,
    "name": "Ananya"
  },
  {
    "id": 102,
    "name": "Rahul"
  }
]
```

So:

```text
GET /employees
      ↓
@RestController
      ↓
List<Employee>
      ↓
JSON
      ↓
Frontend
```
