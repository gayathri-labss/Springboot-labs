# DAY-5-SB: Spring Data JPA


## 1. Request and Response Flow

Request:

```text
GET /employees/101
        ↓
┌──────────────────────┐
│      Controller      │
│    Receives HTTP     │
│    request           │
└──────────────────────┘
        ↓
┌──────────────────────┐
│       Service        │
│    Business logic    │
└──────────────────────┘
        ↓
┌──────────────────────┐
│      Repository      │
│  Database interaction│
└──────────────────────┘
        ↓
    Database
```

Response:

```text
Database
 ↓
Repository
 ↓
Service
 ↓
Controller
 ↓
JSON response
 ↓
UI
```

```text
Controller
→ Handles HTTP

Service
→ Handles business logic

Repository
→ Handles database
```

---

## 2. Spring Data JPA

### 2.1 JPA

JPA is the Java Persistence API. JPA is a specification that defines how Java objects can be stored in and retrieved from a relational database.

Why do we need it?

Without JPA, you could write SQL manually:

```text
SELECT * FROM employee WHERE id = 101;
```

Then you'd have to manually take the database result and create an Employee object.

JPA allows us to work more naturally with Java objects.

For example, eventually you'll be able to write something like:

```text
employeeRepository.findById(101L);
```

and get an Employee object.

You don't have to write the basic SQL yourself.

### 2.2 Hibernate

Hibernate: JPA is a specification, Hibernate is an implementation of this specification.

JPA = interface/contract
Hibernate = implementation

```text
JPA
 ↓
Specification

Hibernate
 ↓
Implementation of JPA

Spring Data JPA
 ↓
Spring's convenient way of working with JPA
```

```text
Your Controller
   ↓
Your Service
   ↓
Spring Data JPA
   ↓
JPA
   ↓
Hibernate
   ↓
Database
```

### 2.3 Why is it called Persistence?

Persistence means making data survive beyond the lifetime of the application/memory, typically by storing it in a database.

What happens if it is not persisted: if the application stops, memory clears and the employee object is gone.

So we persist data in the database.

Without persistence:

```text
Create Employee
      ↓
Employee object in RAM
      ↓
Application stops
      ↓
Employee gone ❌
```

With persistence:

```text
Create Employee
      ↓
Save to database
      ↓
Application stops
      ↓
Start application again
      ↓
Employee still exists ✅
```

---

## 3. Entity

In JPA, an Entity is a Java class that represents data that we want to store in the database.

For example, suppose our database has:

```text
employee
------------------------
id
name
department
salary
```

We can create a Java class:

```text
@Entity
public class Employee {

    private Long id;
    private String name;
    private String department;
    private Double salary;
}
```

The @Entity annotation tells JPA: "This Java class should be mapped to a database table."

```text
Java                        Database

Employee class      →       employee table

id                  →       id
name                →       name
department          →       department
salary              →       salary
```

### ORM (Object Relational Mapping)

It means mapping Java objects to relational database tables.

For example:

```text
Java Object                        Database

Employee object        ←→          employee row

employee.id            ←→          id
employee.name          ←→          name
employee.department    ←→          department
```

### Note

Just adding:

```text
@Entity
public class Employee {
}
```

doesn't mean an employee is automatically inserted into the database.

It means: "JPA knows that Employee is a persistent entity and can map it to a database table."

---

## 4. @Table

How does JPA know the table name?

```text
@Entity
public class Employee {
}
```

And our database has employee.

By default, JPA/Hibernate can map the entity to a table based on the class name and naming strategy.

But if we want to explicitly provide the table name, then we do as below:

```text
@Entity
@Table(name = "employee")
public class Employee {
}
```

"Map this Java class to the employee database table."

```text
@Entity
@Table(name = "employee_details")
public class Employee {
}
```

Ans: Tells JPA to map Employee to the employee_details database table.

---

## 5. @Id

```text
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;
    private String department;
}
```

@Id means: this field is the primary key of the entity. It does not generate the ID automatically. It just says that this field is a primary key.

---

## 6. @GeneratedValue

The value of this primary-key field should be generated automatically.

Example: when the UI sends:

```text
{
  "name": "Ananya",
  "department": "IT"
}
```

There is no ID in the request. We want the system to generate one:

```text
New Employee
     ↓
Save to database
     ↓
id = 101
```

So we use @GeneratedValue:

```text
@Id
@GeneratedValue
private Long id;
```

It means: id is the primary key, and its value should be generated automatically.

### Types of generated values

| Strategy | Basic idea |
|---|---|
| AUTO | JPA chooses an appropriate strategy |
| IDENTITY | Database generates the ID, commonly using auto-increment |
| SEQUENCE | Uses a database sequence to generate IDs |
| TABLE | Uses a separate database table to generate IDs |

#### 1. GenerationType.AUTO

```text
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

We let JPA choose the appropriate ID generation; we don't explicitly tell it to use sequence, identity, etc.

#### 2. GenerationType.IDENTITY

```text
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Here the database generates the ID, commonly using an auto-increment/identity column.

Example:

```text
Insert employee
      ↓
Database
      ↓
ID generated → 101

Insert employee
      ↓
Database
      ↓
ID generated → 102
```

#### 3. GenerationType.SEQUENCE

```text
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE)
private Long id;
```

Here JPA uses a database sequence to generate IDs. Sequences are especially common with databases such as PostgreSQL and Oracle.

#### 4. GenerationType.TABLE

This uses a separate database table to keep track of ID values.

```text
ID generator table
------------------
next_id = 101
```

### Difference between Identity and Sequence

With identity/auto-increment, the ID generation is associated directly with the table column:

```text
Database
   │
   └── employee table
        ├── id ← auto-generated
        ├── name
        └── department
```

With sequence:

```text
Database
   │
   ├── employee table
   │    ├── id
   │    ├── name
   │    └── department
   │
   └── employee_id_seq ← generates IDs
```

That's the main difference.

| | IDENTITY | SEQUENCE |
|---|---|---|
| Who generates ID? | Database's identity/auto-increment mechanism | Database sequence |
| Where is the generator? | Part of the table/column mechanism | Separate database sequence object |
| Example | id auto-increments: 1, 2, 3... | Sequence produces: 1, 2, 3... |
| Commonly associated with | MySQL, SQL Server | PostgreSQL, Oracle |
| Can configure allocation/caching? | More limited | More flexible |
| JPA annotation | GenerationType.IDENTITY | GenerationType.SEQUENCE |

> Note: @GeneratedValue → AUTO by default → Hibernate decides how to generate the ID.

---

## 7. How do fields map to columns?

Consider:

```text
@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private String department;
    private Double salary;
}
```

JPA/Hibernate can map these fields to columns:

| Java Entity | Database |
|---|---|
| id | id |
| name | name |
| department | department |
| salary | salary |

So you don't necessarily need an annotation on every field.

### @Column

@Column lets you customize how a Java field maps to a database column.

If your database column has the name: employee_name
Java field name: private String name;

Then we explicitly tell JPA:

```text
@Column(name = "employee_name")
private String name;
```

```text
@Column(nullable = false)
private String name;
```

The above tells JPA that the column should not allow NULL.

---

## 8. JpaRepository

JpaRepository is a Spring Data JPA interface that provides ready-made methods for performing common database operations.

It provides us methods like:

- save()
- findById()
- findAll()
- deleteById()

Syntax:

```text
public interface RepositoryName extends JpaRepository<EntityType, IDType> {
}
```

Example:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

### Real example

Suppose our Entity is:

```text
@Entity
public class Employee {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private String department;
}
```

Our primary key is:

```text
private Long id;
```

So our repository becomes:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

### Why is JpaRepository an interface?

Because it is defining a contract for repository operations.

You write:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

You haven't written:

```text
save()
findById()
findAll()
```

So who implements them?

Spring Data JPA creates/provides the implementation for you at runtime.

Conceptually:

```text
You
 ↓
EmployeeRepository
(interface)
 ↓
Spring Data JPA
 ↓
generated/provided implementation
 ↓
JPA/Hibernate
 ↓
Database
```

You interact with the interface, while Spring handles the implementation.

> Note: "Why is JPA an interface?" The answer is: JPA itself isn't the interface. JpaRepository is an interface.

---

## 9. save()

Save an Entity object to the database.

Example:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

Because we extended JpaRepository, Spring Data JPA gives us a ready-made method: save()

### 1. Create an Employee object

Suppose we receive employee data from the UI:

```text
{
    "name": "Ananya",
    "department": "IT"
}
```

Spring converts that JSON into:

```text
Employee employee = new Employee();

employee.setName("Ananya");
employee.setDepartment("IT");
```

At this point:

```text
Employee object
----------------
id         → null
name       → Ananya
department → IT
```

It is currently just a Java object in memory.

It is NOT yet in the database.

### 2. Call save()

In the Service:

```text
@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public Employee createEmployee(Employee employee) {

        return employeeRepository.save(employee);
    }
}
```

The important line is:

```text
employeeRepository.save(employee);
```

You're saying: "Spring Data JPA, save this Employee."

### 3. What happens behind the scenes?

Conceptually:

```text
Employee object
      ↓
employeeRepository.save(employee)
      ↓
Spring Data JPA
      ↓
JPA
      ↓
Hibernate
      ↓
SQL
      ↓
Database
```

Hibernate may generate SQL similar to:

```text
INSERT INTO employee
(name, department)
VALUES ('Ananya', 'IT');
```

And because we have:

```text
@Id
@GeneratedValue
private Long id;
```

the ID can be generated automatically.

---

## 10. findById()

Example: employeeRepository.findById(101L);

So it means: find the employee whose id is 101.

It returns Optional<Employee>.

### Why Optional?

Because employee 101 might exist or might not exist.

For example:

```text
Database

ID    Name
101   Rahul
102   Ananya
103   Priya
```

If we do:

```text
findById(101L)
```

➡️ Employee exists → Optional contains Rahul.

If we do:

```text
findById(999L)
```

➡️ Employee doesn't exist → Optional.empty().

So instead of returning null, Spring gives us an Optional that represents:

```text
Employee found      → Optional<Employee>
Employee not found  → Optional.empty()
```

Example:

```text
Employee emp = employeeRepository.findById(101L).orElse(null);
```

Find employee 101. If found, give me the Employee. If not found, give me null.

---

## 11. findAll()

Example:

```text
List<Employee> employees =
        employeeRepository.findAll();
```

It means give all the employees from the database.

Suppose your database has:

| ID | Name | Department |
|---|---|---|
| 101 | Rahul | IT |
| 102 | Ananya | HR |
| 103 | Priya | Finance |

When you call:

```text
List<Employee> employees =
        employeeRepository.findAll();
```

you get a List<Employee> containing all 3 employees.

```text
findAll()
    ↓
List<Employee>
    ↓
[ Rahul, Ananya, Priya ]
```

---

## 12. deleteById()

Example: employeeRepository.deleteById(101L);

Delete the employee whose ID is 101 from the database.

```text
CREATE → save()
READ   → findById() / findAll()
UPDATE → save()
DELETE → deleteById()
```

---

## 13. save() for Create and Update

save() can be used to create/update as follows.

### 1. Create

Creating a new employee. Suppose:

```text
Employee employee = new Employee();

employee.setName("Rahul");
employee.setDepartment("IT");
```

The ID is initially:

```text
id = null
```

Then:

```text
employeeRepository.save(employee);
```

Spring Data JPA treats it as a new entity and typically results in:

```text
INSERT INTO employee ...
```

So:

```text
id = null
   ↓
save()
   ↓
CREATE / INSERT
```

### 2. Update

Suppose the database already has:

| ID | Name | Department |
|---|---|---|
| 101 | Rahul | IT |

You retrieve employee 101:

```text
Employee employee =
        employeeRepository.findById(101L).orElseThrow();
```

Then change something:

```text
employee.setDepartment("Finance");
```

Then:

```text
employeeRepository.save(employee);
```

Now you're working with an existing employee, so JPA can update the existing record:

```text
UPDATE employee
SET department = 'Finance'
WHERE id = 101;
```

```text
New entity
id = null
 ↓
save()
 ↓
INSERT


Existing entity
id = 101
 ↓
save()
 ↓
UPDATE
```

---

## 14. Spring Data JPA Derived Methods / Custom Queries

We write a repository method using a specific pattern, so Spring Data JPA automatically creates database queries for you.

Example: instead of writing SQL queries as:

```text
SELECT * FROM employee
WHERE department = 'IT';
```

We can write as:

```text
List<Employee> findByDepartment(String department);
```

### 2. Where do we write it?

Inside our repository:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    List<Employee> findByDepartment(String department);
}
```

Notice that we don't write the implementation.

Spring Data JPA sees:

```text
findByDepartment(...)
```

and understands:

```text
find
  +
by
  +
Department
```

Meaning: find Employee records where the department field matches the given value.

### 3. How do we call it?

From the service:

```text
List<Employee> employees =
        employeeRepository.findByDepartment("IT");
```

Suppose the database contains:

| ID | Name | Department |
|---|---|---|
| 101 | Rahul | IT |
| 102 | Ananya | HR |
| 103 | Priya | IT |
| 104 | Kiran | Finance |

Then:

```text
findByDepartment("IT")
```

returns:

```text
Rahul
Priya
```

So the important pattern is: findBy + EntityFieldName

### Complete example: Find employees by city

Suppose our database has this table:

| ID | Name | Department | City |
|---|---|---|---|
| 101 | Rahul | IT | Bangalore |
| 102 | Ananya | HR | Mumbai |
| 103 | Priya | IT | Bangalore |
| 104 | Kiran | Finance | Delhi |

We want an API:

```text
GET /employees/city/Bangalore
```

which should return Rahul and Priya.

#### Step 1: Entity

Our Employee class represents the database table.

```text
@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private String department;

    private String city;

    // getters and setters
}
```

The important part for our example is:

```text
private String city;
```

Because that's the field we want to search by.

#### Step 2: Repository

Now we create our repository:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    List<Employee> findByCity(String city);
}
```

This line is the important one:

```text
List<Employee> findByCity(String city);
```

Spring Data JPA reads the method name:

```text
find + By + City
```

and understands: find all Employee records where the city field matches the given value.

We don't write SQL ourselves.

#### Step 3: Service

The service calls the repository:

```text
@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public List<Employee> getEmployeesByCity(String city) {

        return employeeRepository.findByCity(city);
    }
}
```

So when we call:

```text
employeeRepository.findByCity("Bangalore");
```

the repository searches for employees whose city is Bangalore.

#### Step 4: Controller

Now we expose this functionality through a REST API:

```text
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping("/city/{city}")
    public List<Employee> getEmployeesByCity(
            @PathVariable String city) {

        return employeeService.getEmployeesByCity(city);
    }
}
```

Now the frontend/Postman can send:

```text
GET /employees/city/Bangalore
```

---

## 15. Derived Queries: AND

Find employees who are in the IT department AND located in Bangalore.

Our entity has:

```text
private String name;
private String department;
private String city;
```

Repository method:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    List<Employee> findByDepartmentAndCity(
            String department,
            String city
    );
}
```

Spring JPA reads this as:

```text
find
 ↓
By
 ↓
Department
 ↓
And
 ↓
City
```

### Calling the method

From our service:

```text
public List<Employee> getEmployees(
        String department,
        String city) {

    return employeeRepository
            .findByDepartmentAndCity(department, city);
}
```

We could call:

```text
employeeService.getEmployees("IT", "Bangalore");
```

Which eventually calls:

```text
employeeRepository
    .findByDepartmentAndCity("IT", "Bangalore");
```

### What does the database do?

Suppose we have:

| ID | Name | Department | City |
|---|---|---|---|
| 101 | Rahul | IT | Bangalore |
| 102 | Ananya | HR | Bangalore |
| 103 | Priya | IT | Bangalore |
| 104 | Kiran | IT | Delhi |
| 105 | Sneha | Finance | Bangalore |

We call:

```text
findByDepartmentAndCity("IT", "Bangalore");
```

The result is:

```text
Rahul
Priya
```

Because both conditions must be true.

Conceptually, the query is:

```text
SELECT *
FROM employee
WHERE department = 'IT'
AND city = 'Bangalore';
```

Pattern to remember is: findByField1AndField2(...)

---

## 16. Derived Queries: OR

Example: find all employees who work in HR OR Finance.

Ans: findByDepartmentOrDepartment(...)

> Note: Spring Data JPA reads the repository method name and derives the query automatically.

---

## 17. Containing

Containing: search inside a field.

Example: we want to find employees whose name contains "an".

### 1. Repository

We can write:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    List<Employee> findByNameContaining(String name);
}
```

The important part:

```text
findByNameContaining(String name)
```

means: find employees where the name contains the given text.

### 2. Calling it

```text
List<Employee> employees =
        employeeRepository.findByNameContaining("an");
```

Spring Data JPA conceptually creates a query similar to:

```text
SELECT *
FROM employee
WHERE name LIKE '%an%';
```

The % means there can be characters before or after "an".

So:

```text
"Ananya"  → contains "an" ✅
"Anand"   → contains "an" ✅
"Kiran"   → contains "an" ✅
"Rahul"   → contains "an" ❌
"Priya"   → contains "an" ❌
```

```text
Frontend
 ↓
User types: "an"
 ↓
GET /employees/search?name=an
 ↓
Controller
 ↓
Service
 ↓
findByNameContaining("an")
 ↓
Database
```

| Method | Meaning |
|---|---|
| findByName("Ananya") | Exact match |
| findByNameContaining("an") | Contains "an" |

---

## 18. IgnoreCase

Example:

```text
List<Employee> findByNameContainingIgnoreCase(String name);
```

Output:

```text
"raj" → Rajesh ✅
"raj" → RAJESH ✅
"raj" → rajesh ✅
```

| Method | Meaning |
|---|---|
| findByName() | Match name |
| findByNameContaining() | Name contains text |
| findByNameIgnoreCase() | Match name, ignore case |
| findByNameContainingIgnoreCase() | Contains text + ignore case ⭐ |

---

## 19. GreaterThan, LessThan and more

GreaterThan

Example: find employees whose salary is greater than ₹10,00,000.

Suppose our Employee entity has:

```text
private Double salary;
```

We want: find employees whose salary is greater than ₹10,00,000.

In the repository:

```text
List<Employee> findBySalaryGreaterThan(Double salary);
```

Then:

```text
employeeRepository.findBySalaryGreaterThan(1000000.0);
```

Conceptually, Spring Data JPA creates:

```text
SELECT *
FROM employee
WHERE salary > 1000000;
```

LessThan:

```text
List<Employee> findBySalaryLessThan(Double salary);
```

GreaterThanEqual:

```text
findBySalaryGreaterThanEqual(Double salary);
```

LessThanEqual: similarly.

| Method | Meaning |
|---|---|
| GreaterThan | > |
| GreaterThanEqual | >= |
| LessThan | < |
| LessThanEqual | <= |

---

## 20. Between

Find employees whose salary is between ₹5 lakh and ₹10 lakh.

### 1. Repository

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    List<Employee> findBySalaryBetween(
            Double minSalary,
            Double maxSalary
    );
}
```

The important part is:

```text
findBySalaryBetween(...)
```

Spring Data JPA understands: find employees whose salary is between the two given values.

### 2. Calling it

```text
List<Employee> employees =
        employeeRepository.findBySalaryBetween(
            500000.0,
            1000000.0
        );
```

> Note: Between is inclusive at both ends.

---

## 21. OrderBy

Find all IT employees and show them from highest salary to lowest salary.

### 1. Repository method

```text
List<Employee> findByDepartmentOrderBySalaryDesc(
        String department
);
```

Let's break the method name:

```text
findBy
   ↓
Department
   ↓
OrderBy
   ↓
Salary
   ↓
Desc
```

Meaning: find employees where department matches, and sort them by salary in descending order.

### 2. Calling it

```text
List<Employee> employees =
        employeeRepository
            .findByDepartmentOrderBySalaryDesc("IT");
```

Result:

```text
Priya → ₹15L
Kiran → ₹10L
Rahul → ₹8L
```

Because Desc means: descending → highest to lowest.

### Asc vs Desc

You can also use:

```text
findByDepartmentOrderBySalaryAsc(String department);
```

Asc means ascending:

```text
₹8L
₹10L
₹15L
```

Desc means descending:

```text
₹15L
₹10L
₹8L
```

---

## 22. @Query

@Query allows us to write the query ourselves inside the repository.

Syntax:

```text
@Query("query here")
ReturnType methodName();
```

Example:

```text
@Query("SELECT e FROM Employee e WHERE e.department = :department")
List<Employee> findEmployeesByDepartment(
    @Param("department") String department
);
```

Suppose we have:

```text
@Entity
public class Employee {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private String department;

    private Double salary;
}
```

We want: find employees belonging to a particular department.

We could use a derived query:

```text
List<Employee> findByDepartment(String department);
```

But with @Query, we can explicitly write:

```text
@Query("SELECT e FROM Employee e WHERE e.department = :department")
List<Employee> findEmployeesByDepartment(
        @Param("department") String department
);
```

So this is JPQL [query present inside @Query].

JPQL works with Java entity names and Java fields and not directly with database table and column names.

```text
JPQL
 ↓
Employee entity
 ↓
Hibernate
 ↓
SQL
 ↓
Database
```

Hibernate converts the JPQL into the appropriate SQL.

Derived query → Spring derives the query from the method name.
@Query → We explicitly write the query.

JPQL stands for Java Persistence Query Language. It is the query language used with JPA to query your Java entities.

Example: SELECT e FROM Employee e. What does e represent?
Ans: Alias for the Employee entity.

Example: if we call:

```text
getEmployeesByDepartment("IT");
```

the value flows:

```text
"IT"
  ↓
Service
  ↓
Repository method
  ↓
@Param("department")
  ↓
:department
  ↓
JPQL WHERE condition
  ↓
Hibernate
  ↓
SQL
  ↓
Database
```

@Param connects Java to JPQL.

Repository:

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    @Query("""
        SELECT e
        FROM Employee e
        WHERE e.department = :department
    """)
    List<Employee> findEmployeesByDepartment(
            @Param("department") String department
    );
}
```

Service:

```text
@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public List<Employee> getEmployeesByDepartment(
            String department) {

        return employeeRepository
                .findEmployeesByDepartment(department);
    }
}
```

Remember: WHERE e.department = :department → this means compare the entity's department field with the parameter named department.

### Why @Query is useful?

When the requirement becomes more complicated, instead of creating a huge method name like:

```text
findByDepartmentAndCityAndSalaryGreaterThanAndJoiningDateBetween(...)
```

you can write a readable JPQL query with @Query.

---

## 23. Date ranges to understand @Query

Example: find employees whose joining date falls within this range.

### 1. Repository

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    @Query("""
        SELECT e
        FROM Employee e
        WHERE e.joiningDate BETWEEN :fromDate AND :toDate
    """)
    List<Employee> findEmployeesByDateRange(
            @Param("fromDate") LocalDate fromDate,
            @Param("toDate") LocalDate toDate
    );
}
```

Let's break it down. This:

```text
WHERE e.joiningDate BETWEEN :fromDate AND :toDate
```

means: find employees where joiningDate is between the two dates supplied by the user.

The parameters connect like this:

```text
JPQL                 Java
--------------------------------
:fromDate     →      fromDate
:toDate       →      toDate
```

### 2. Service

```text
@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public List<Employee> getEmployeesByDateRange(
            LocalDate fromDate,
            LocalDate toDate) {

        return employeeRepository.findEmployeesByDateRange(
                fromDate,
                toDate
        );
    }
}
```

### 3. Controller

```text
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping("/search")
    public List<Employee> searchEmployees(
            @RequestParam LocalDate fromDate,
            @RequestParam LocalDate toDate) {

        return employeeService.getEmployeesByDateRange(
                fromDate,
                toDate
        );
    }
}
```

### 4. UI sends the request

The UI sends:

```text
GET /employees/search?fromDate=2026-01-01&toDate=2026-03-31
```

Spring converts those values into:

```text
LocalDate fromDate = 2026-01-01;
LocalDate toDate = 2026-03-31;
```

Then:

```text
UI
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
@Query
 ↓
Hibernate
 ↓
Database
```

---

## 24. LIKE in @Query

LIKE is used for pattern matching, commonly for search functionality such as finding records whose name contains, starts with, or ends with a particular value.

```text
%an%  → contains "an"

an%   → starts with "an"

%an   → ends with "an"
```

Example: the user expects employees whose names contain "an".

### 1. Repository

```text
public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {

    @Query("""
        SELECT e
        FROM Employee e
        WHERE e.name LIKE %:name%
    """)
    List<Employee> searchByName(
            @Param("name") String name
    );
}
```

The important part is:

```text
WHERE e.name LIKE %:name%
```

What does % mean?

% means: any number of characters can come before or after the search text.

So if:

```text
name = "an"
```

then:

```text
%an%
```

### 3. Controller

The UI can send:

```text
GET /employees/search?name=an
```

Controller:

```text
@GetMapping("/search")
public List<Employee> searchEmployees(
        @RequestParam String name) {

    return employeeService.searchByName(name);
}
```

Service:

```text
public List<Employee> searchByName(String name) {

    return employeeRepository.searchByName(name);
}
```

```text
UI
 ↓
GET /employees/search?name=an
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
@Query
 ↓
Hibernate
 ↓
Database
```

---

## 25. IN in @Query

It's especially useful when the UI allows multiple selections.

```text
Departments:
[ IT ✓ ] [ HR ✓ ] [ Finance ✗ ] [ Sales ✗ ]
```

The UI can send:

```text
GET /employees?department=IT&department=HR
```

Your backend can convert that into:

```text
List<String> departments
```

and pass it to:

```text
findEmployeesByDepartments(departments);
```

Difference between = and IN:

```text
=   → one value

IN  → multiple possible values
```

---

## 26. COUNT in @Query

```text
@Query("""
    SELECT COUNT(e)
    FROM Employee e
    WHERE e.department = :department
""")
```

If the database contains 10 IT employees, what will the repository method return?

A. A List<Employee> containing 10 employees
B. The number 10
C. The string "10"
D. An Employee object

Flow remains the same.

---

## 27. SUM in @Query

### 1. Repository

```text
@Query("""
    SELECT SUM(e.salary)
    FROM Employee e
    WHERE e.department = :department
""")
Double getTotalSalaryByDepartment(
        @Param("department") String department
);
```

The important part is:

```text
SELECT SUM(e.salary)
```

It means: add all the salary values of the matching employees.

### 2. Calling it

```text
Double totalSalary =
        employeeRepository.getTotalSalaryByDepartment("IT");
```

The matching employees are:

```text
Rahul   → ₹8L
Ananya  → ₹10L
Kiran   → ₹12L
```

So:

```text
8L + 10L + 12L = 30L
```

Result:

```text
totalSalary = ₹30L
```

> Note: SUM can return null if conditions do not match.

---

## 28. AVG in @Query

### 1. Repository

```text
@Query("""
    SELECT AVG(e.salary)
    FROM Employee e
    WHERE e.department = :department
""")
Double getAverageSalaryByDepartment(
        @Param("department") String department
);
```

The important part:

```text
AVG(e.salary)
```

means: calculate the average of the salary values.

### 2. Calling it

```text
Double averageSalary =
        employeeRepository.getAverageSalaryByDepartment("IT");
```

The matching salaries are:

```text
₹5L
₹7L
₹8L
```

Calculation:

```text
(5 + 7 + 8) / 3
= 20 / 3
= ₹6.67L
```

So:

```text
averageSalary = ₹6.67L
```

---

## 29. @Modifying

This @Query is going to modify data in the database.

It's mainly used with UPDATE and DELETE.

Without @Modifying, Spring Data JPA normally treats @Query as a read/query operation.

Syntax:

```text
@Modifying
@Query("UPDATE ...")
int methodName(...);
```

Or:

```text
@Modifying
@Query("DELETE ...")
int methodName(...);
```

### What does the return type mean?

Usually you'll see:

```text
int
```

Example:

```text
int updateDepartment(...);
```

The int represents: number of database rows affected.

So:

```text
0 → no rows affected
1 → one row affected
5 → five rows affected
```

One important distinction: @Modifying does NOT itself update the database.

The UPDATE/DELETE query does the actual modification.

@Modifying simply tells Spring how to treat that query.

### Example

```text
@Modifying
@Query("""
    UPDATE Employee e
    SET e.department = :department
    WHERE e.id = :id
""")
int updateDepartment(
        @Param("id") Long id,
        @Param("department") String department
);
```

Let's understand each part.

UPDATE Employee e

```text
UPDATE Employee e
```

We're telling JPQL: update the Employee entity.

Remember: JPQL uses the entity name, not the database table name.

---

## 30. @Transactional

@Transactional tells Spring to execute a method within a database transaction. A group of related DB operations is treated as one unit.

A transaction groups related database operations into one unit of work, so they are committed together if successful, or rolled back together if the transaction fails.

```text
@Transactional
      ↓
START TRANSACTION
      ↓
run method
      ↓
success → COMMIT
failure → ROLLBACK
```

### Why do we need both?

Suppose we have:

```text
@Transactional
@Modifying
@Query("""
    UPDATE Employee e
    SET e.department = :department
    WHERE e.id = :id
""")
int updateDepartment(
        @Param("id") Long id,
        @Param("department") String department
);
```

Each annotation has a different job:

```text
@Query
 ↓
What should the database do?
 → UPDATE department

@Modifying
 ↓
This query changes data
 → UPDATE / DELETE

@Transactional
 ↓
Manage the transaction
 → COMMIT / ROLLBACK
```

---

## 31. @Modifying + Delete

We use @Modifying with a JPQL DELETE query when we want to remove records from the database.

Syntax:

```text
@Modifying
@Query("""
    DELETE FROM Employee e
    WHERE e.id = :id
""")
int deleteEmployee(
        @Param("id") Long id
);
```

### Why does it return int?

Just like UPDATE, the int tells us: how many rows were affected.

For example:

```text
1 → one employee deleted
0 → no employee matched
5 → five employees deleted
```

```text
@Modifying
@Query("""
    DELETE FROM Employee e
    WHERE e.department = :department
""")
int deleteByDepartment(
        @Param("department") String department
);
```

and there are 5 HR employees, what will the method return when we call:

```text
deleteByDepartment("HR");
```

A. 0
B. 1
C. 5
D. A List<Employee>

Ans is 5.

---

## 32. Pagination

Pagination means retrieving a large set of records in smaller pages instead of loading everything at once.

### Pageable

Pageable is a Spring Data interface that tells the repository: which page do I want, and how many records should that page contain?

Basic idea:

```text
Pageable pageable
```

contains information such as:

```text
Page number
Page size
Sorting information
```

For example:

```text
page = 0
size = 20
```

means: give me the first page containing 20 records.

⚠️ Page numbers start from 0, not 1.

So:

```text
0 → first page
1 → second page
2 → third page
```

Instead of:

```text
List<Employee> findAll();
```

we can use:

```text
Page<Employee> findAll(Pageable pageable);
```

Notice two things:

```text
Page<Employee>
      ↑
Returns a page of employees

Pageable
      ↑
Tells Spring which page to retrieve
```

Syntax:

```text
Pageable pageable = PageRequest.of(pageNumber, pageSize);
```

```text
Pageable pageable = PageRequest.of(0, 20);
```

- 0 → first page
- 20 → 20 records per page

```text
Page<Employee> findAll(Pageable pageable);
```

Then:

```text
Pageable pageable = PageRequest.of(0, 20);

Page<Employee> employees =
        employeeRepository.findAll(pageable);
```

The flow is:

```text
PageRequest.of(0, 20)
        ↓
Pageable
        ↓
Repository
        ↓
Page<Employee>
```

One important distinction:

```text
Pageable → tells Spring what page to retrieve

Page<Employee> → contains the results and page information
```

### Page<Employee>

Page gives you the current records plus information about the pagination. So the Page object will provide us the result of the current employees + pagination information.

Syntax

Repository:

```text
Page<Employee> findAll(Pageable pageable);
```

Then:

```text
Page<Employee> page =
        employeeRepository.findAll(pageable);
```

Here:

```text
Page<Employee>
      ↑
A page containing Employee objects

Pageable
      ↑
Tells Spring which page and how many records to retrieve
```

Methods of Page:

```text
page.getContent();
page.getTotalElements();
page.getTotalPages();
page.getNumber();
page.getSize();
```

### Example

Suppose:

```text
Pageable pageable = PageRequest.of(1, 20);
```

and the database contains 55 employees.

Then:

```text
getContent().size()  → 20
getTotalElements()   → 55
getTotalPages()      → 3
getNumber()          → 1
getSize()            → 20
```

Notice:

```text
getSize()            → requested page size
getContent().size()  → actual records on this page
```

Suppose the database has:

```text
100 employees
```

We request:

```text
Pageable pageable = PageRequest.of(0, 10);
```

Meaning:

```text
Page 0
10 employees per page
```

Then:

```text
Page<Employee> page =
        employeeRepository.findAll(pageable);
```

Now:

```text
page.getContent()
```
→ Employees 1–10

```text
page.getTotalElements()
```
→ 100

```text
page.getTotalPages()
```
→ 10

```text
page.getNumber()
```
→ 0

```text
page.getSize()
```
→ 10

So the Page object is basically giving us:

```text
Current employees
        +
Pagination information
```

```text
getContent()       → records on THIS page
getTotalElements() → records across ALL pages
```

getTotalPages(): the total number of pages available for the query. getTotalPages() tells you how many complete/partial pages are needed to contain all records.

Syntax:

```text
int totalPages = page.getTotalPages();
```

The return type is int because the number of pages is an int.

### Example

Suppose:

```text
Total employees = 100
Page size       = 20
```

Then:

```text
Page 0 → 1–20
Page 1 → 21–40
Page 2 → 41–60
Page 3 → 61–80
Page 4 → 81–100
```

Therefore:

```text
page.getTotalPages()
```

returns:

```text
5
```

```text
getNumber()        → The page number of the current page
getSize()          → The number of records requested for each page

getContent()       → Employees on the current page
getTotalElements() → Total employees in database
getTotalPages()    → Total number of pages
getNumber()        → Current page index
getSize()          → Records requested per page
```

---

## 33. Sorting

Arranging database records in a particular order.

For example, your UI might have:

```text
Sort By:
○ Name — A → Z
○ Salary — High → Low
○ Joining Date — Newest → Oldest
```

For employees:

```text
Before sorting:

Rahul   ₹8L
Ananya  ₹12L
Kiran   ₹6L
Priya   ₹15L
```

Salary descending:

```text
Priya   ₹15L
Ananya  ₹12L
Rahul   ₹8L
Kiran   ₹6L
```

Syntax:

```text
Sort sort = Sort.by("salary").descending();
```

### Important detail

The field name should be the Java entity field, not necessarily the database column name.

For example:

```text
@Column(name = "employee_salary")
private Double salary;
```

You would use:

```text
Sort.by("salary")
```

not:

```text
Sort.by("employee_salary")
```

because Spring Data works with the entity property.

### Sorting with Pageable

```text
Pageable pageable =
        PageRequest.of(
            0,
            20,
            Sort.by("salary").descending()
        );
```

### Multiple field sorting

```text
Sort sort = Sort.by(
    Sort.Order.desc("salary"),
    Sort.Order.asc("name")
);
```

Note: the second rule is used when the first field has the same value.

### Quick quiz

Suppose we have:

| Name | Salary |
|---|---|
| Rahul | ₹10L |
| Ananya | ₹10L |
| Kiran | ₹8L |

With:

```text
Sort.by(
    Sort.Order.desc("salary"),
    Sort.Order.asc("name")
)
```

What will the order be?

A. Kiran, Ananya, Rahul
B. Ananya, Rahul, Kiran
C. Rahul, Ananya, Kiran
D. Ananya, Kiran, Rahul

Answer is B.

Now Rahul and Ananya have the same salary, so we use the second rule: Sort.Order.asc("name").

Name A → Z: Ananya, Rahul

---

## 34. Entity Relationship

How two entities are connected or associated with each other.

### Types of entity relationship

#### 1. OneToOne

One entity is associated with exactly one other entity.

Syntax in JPA:

```text
@OneToOne
private Address address;
```

That's the basic syntax.

So an Employee entity could have:

```text
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;

    @OneToOne
    private Address address;
}
```

This means Employee has a one-to-one relationship with Address.
