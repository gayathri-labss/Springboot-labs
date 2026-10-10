# DAY-6-SB: Entity Relationships and Hibernate


## 1. @ManyToOne

Many entities are associated with one entity.

Syntax:

```text
@ManyToOne
private Department department;
```

Many Employees can belong to one Department.

### Example

```text
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;

    @ManyToOne
    private Department department;
}
```

And:

```text
@Entity
public class Department {

    @Id
    private Long id;

    private String name;
}
```

Example:

```text
Department
    IT
    ↑
    |
 ┌──┼──┐
 |  |  |
Rahul Ananya Kiran
```

For example:

```text
class Department {

    @OneToMany
    private List<Employee> employees;
}
```

means: from the Department's perspective, one Department has many Employees.

Whereas:

```text
class Employee {

    @ManyToOne
    private Department department;
}
```

means: from the Employee's perspective, many Employees belong to one Department.

---

## 2. Identify the relationship

```text
class Customer {

    @OneToMany
    private List<Order> orders;
}
```

Ans: One customer has many orders

```text
class Order {

    @ManyToOne
    private Customer customer;
}
```

Ans: Many orders have one customer

```text
class Employee {

    @OneToOne
    private Passport passport;
}
```

Ans: One employee has one passport

```text
class Student {

    @ManyToMany
    private List<Course> courses;
}
```

Ans: One Student has many Courses, and Courses can have many Students

```text
class Course {

    @OneToMany
    private List<Student> students;
}
```

Ans: One Course has many Students

---

## 3. mappedBy

This relationship is already mapped/managed by the field on the other entity. Don't create another relationship mapping from this side.

Syntax:

```text
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

Department:

```text
@Entity
public class Department {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}
```

Employee:

```text
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;

    @ManyToOne
    private Department department;
}
```

---

## 4. HIBERNATE

### 4.1 Entity States

An entity state describes the current status of a Java entity object in relation to Hibernate's Persistence Context.

Example:

```text
Employee employee = new Employee();
```

Hibernate needs to know:

- Is this a new object?
- Is Hibernate currently managing it?
- Has it been disconnected?
- Has it been deleted?

That's why we have 4 entity states:

```text
Transient
    ↓
Managed
    ↓
Detached

Managed
    ↓
Removed
```

### 4.2 Transient State

An entity is Transient when you create a Java object, but Hibernate is not managing it yet.

Example:

```text
Employee employee = new Employee();

employee.setName("Rahul");
employee.setSalary(80000.0);
```

At this point:

```text
Java Object
    ↓
Employee
    ↓
Hibernate is NOT managing it
    ↓
Transient
```

### 4.3 Managed / Persistent State

An entity is Managed when it is attached to Hibernate's Persistence Context. Once Hibernate manages the entity, Hibernate can track changes made to it.

Example:

```text
Employee employee = new Employee();

entityManager.persist(employee);
```

After persist():

```text
Employee object
    ↓
entityManager.persist()
    ↓
Persistence Context
    ↓
MANAGED
```

Now Hibernate is aware of this object.

### 4.4 Detached State

An entity is Detached when it was previously managed by Hibernate, but is no longer attached to the Persistence Context.

Example:

```text
Employee employee = new Employee();

entityManager.persist(employee);  // Managed

entityManager.detach(employee);   // Detached
```

```text
Transient
    ↓
persist()
    ↓
Managed
    ↓
detach()
    ↓
Detached
```

Important: the Java object still exists. But Hibernate is no longer tracking it.

### 4.5 Removed State

An entity is Removed when it is managed by Hibernate and has been marked for deletion from the database.

Example:

```text
Employee employee = entityManager.find(Employee.class, 101L);

entityManager.remove(employee);
```

Important distinction: remove() doesn't necessarily mean the SQL DELETE happens immediately.

It means: "Hibernate, this managed entity should be deleted."

---

## 5. First-Level Cache

First-Level Cache is a cache maintained by Hibernate inside the Persistence Context. It is enabled by default.

Example:

```text
Employee e1 = entityManager.find(Employee.class, 101L);
Employee e2 = entityManager.find(Employee.class, 101L);
```

Without caching:

```text
find employee 101
    ↓
Database

find employee 101
    ↓
Database again
```

Hibernate's first-level cache:

```text
First request
    ↓
Persistence Context
    ↓
Database
    ↓
Employee 101 stored in cache
```

```text
Second request
    ↓
Persistence Context
    ↓
Employee 101 already exists
    ↓
Return it from cache
```

⭐ Very important interview point

First-Level Cache:

- Enabled by default ✅
- Associated with Persistence Context / Session ✅
- Stores managed entities ✅
- Helps avoid repeated database access within the same persistence context ✅
- Not a global application-wide cache ❌

---

## 6. Hibernate — Lazy vs Eager Loading

LAZY loading means related data is loaded only when it is actually needed/accessed.

For example:

```text
@ManyToOne(fetch = FetchType.LAZY)
private Department department;
```

Suppose you do:

```text
Employee employee = employeeRepository.findById(101L);
```

You only need:

```text
employee.getName();
```

You don't need the department.

So Hibernate can avoid loading the Department at that point.

EAGER loading means related data is loaded immediately along with the main entity.

Example:

```text
@ManyToOne(fetch = FetchType.EAGER)
private Department department;
```

When you load:

```text
Employee employee = employeeRepository.findById(101L);
```

Hibernate loads the Employee and the Department relationship eagerly, according to its fetching strategy.

Conceptually:

```text
Get Employee
     ↓
Employee + Department
     ↓
Loaded
```

You don't have to explicitly access:

```text
employee.getDepartment();
```

for the relationship to be considered eagerly fetched.

| | LAZY | EAGER |
|---|---|---|
| Related data | Later | Immediately |
| Initial loading | Less data | More data |
| Can reduce unnecessary loading | ✅ | ❌ |
| Common concern | LazyInitialization issues | Unnecessary data / performance |

> Note:
> LAZY = don't fetch the relationship until needed.
> EAGER = fetch the relationship as part of loading the entity.

---

## 7. Native Queries

A Native Query is a query written directly in the database's SQL language.

```text
SELECT *
FROM employee
WHERE salary > 100000;
```

With JPQL, we write using Java entity and field names:

```text
@Query("SELECT e FROM Employee e WHERE e.salary > :salary")
```

So:

```text
JPQL
 ↓
Works with Entity + Java fields
 ↓
Hibernate
 ↓
SQL
 ↓
Database
```

Whereas:

```text
Native Query
 ↓
SQL directly
 ↓
Database
```

---

## 8. Persistence Context

Persistence Context is a temporary area managed by JPA/Hibernate where Hibernate keeps track of entity objects that are currently managed and enables features such as first-level caching and dirty checking.

## 9. Dirty Checking

Dirty checking is Hibernate's mechanism for detecting changes made to managed entities and synchronizing those changes with the database.

```text
Transient → ❌ not tracked
Managed   → ✅ tracked
Detached  → ❌ not tracked
```

Persistence Context manages/tracks the entity, and Dirty Checking detects changes made to that managed entity.

## 10. N+1 Problem

N+1 occurs when Hibernate executes one query to fetch the parent entities and additional queries for each related entity. It can be addressed using techniques such as JPQL JOIN FETCH or @EntityGraph.
