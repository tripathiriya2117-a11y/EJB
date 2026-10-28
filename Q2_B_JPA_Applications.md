# Q.2 B — JPA Applications (Questions 13–30)

This file covers **all 18 JPA practicals, Q13–Q30**, from the question bank. Each practical includes file paths, entity code, a servlet, and execution guidance.

## 1. Common setup (used by all 18 practicals)

The examples use **Jakarta Persistence (JPA)** with Hibernate, Jakarta Servlet, Maven, and MySQL. They are suitable for a **Tomcat 10.1+** web project. These are separate practicals: create a separate project for each question, or copy only the selected practical into one project. Do not place all the repeated servlet mappings with the same URL in one application.

### Project structure

```text
JpaPractical/
├── pom.xml
└── src/main/
    ├── java/com/example/jpa/
    │   ├── JpaUtil.java
    │   ├── Book.java                 (example: Q13)
    │   └── BookServlet.java          (example: Q13)
    └── resources/META-INF/persistence.xml
```

### `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>JpaPractical</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.hibernate.orm</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>6.6.5.Final</version>
    </dependency>
    <dependency>
      <groupId>jakarta.persistence</groupId>
      <artifactId>jakarta.persistence-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.0.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <version>8.4.0</version>
    </dependency>
  </dependencies>
  <build>
    <finalName>JpaPractical</finalName>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.13.0</version>
      </plugin>
      <plugin>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.4.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

### Database

Run this in MySQL Workbench or the MySQL command line:

```sql
CREATE DATABASE jpa_practical;
```

### `src/main/resources/META-INF/persistence.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd"
             version="3.0">
  <persistence-unit name="EJPU" transaction-type="RESOURCE_LOCAL">
    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
    <exclude-unlisted-classes>false</exclude-unlisted-classes>
    <properties>
      <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
      <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/jpa_practical"/>
      <property name="jakarta.persistence.jdbc.user" value="root"/>
      <property name="jakarta.persistence.jdbc.password" value="YOUR_MYSQL_PASSWORD"/>
      <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>
      <property name="hibernate.hbm2ddl.auto" value="update"/>
      <property name="hibernate.show_sql" value="true"/>
    </properties>
  </persistence-unit>
</persistence>
```

Replace `YOUR_MYSQL_PASSWORD` with your local MySQL password. For a college lab system with a blank password, use an empty value. `update` creates/updates tables; it does not reliably reset existing data.

### `src/main/java/com/example/jpa/JpaUtil.java`

```java
package com.example.jpa;

import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public final class JpaUtil {
    private static final EntityManagerFactory FACTORY =
            Persistence.createEntityManagerFactory("EJPU");

    private JpaUtil() {}

    public static EntityManager getEntityManager() {
        return FACTORY.createEntityManager();
    }

    public static void close() {
        if (FACTORY.isOpen()) FACTORY.close();
    }
}
```

**How to run:** use JDK 17+, MySQL, and Tomcat 10.1+. Build with `mvn clean package`, deploy `target/JpaPractical.war` to Tomcat, then open the URL shown for the selected question. The first run may take a few seconds while Hibernate initializes.

**Important:** Every servlet below uses `jakarta.servlet.*` and every entity uses `jakarta.persistence.*`. Do not mix these with old `javax.*` imports. For safety, the example pages display submitted values as plain text and use prepared JPA operations rather than concatenating SQL.

---

## Q13. JPA application to store and retrieve book details

**Files**
- `src/main/java/com/example/jpa/Book.java`
- `src/main/java/com/example/jpa/BookServlet.java`

### `Book.java`

```java
package com.example.jpa;

import jakarta.persistence.*;

@Entity
@Table(name = "books_q13")
public class Book {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String author;
    private double price;

    public Book() {}
    public Book(String title, String author, double price) {
        this.title = title; this.author = author; this.price = price;
    }
    public Long getId() { return id; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public double getPrice() { return price; }
}
```

### `BookServlet.java`

```java
package com.example.jpa;

import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityTransaction;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;
import java.io.*;
import java.util.List;

@WebServlet("/q13")
public class BookServlet extends HttpServlet {
    protected void service(HttpServletRequest req, HttpServletResponse res)
            throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        String action = req.getParameter("action");
        try (PrintWriter out = res.getWriter()) {
            EntityManager em = JpaUtil.getEntityManager();
            try {
                if ("save".equals(action)) {
                    EntityTransaction tx = em.getTransaction(); tx.begin();
                    em.persist(new Book(req.getParameter("title"),
                            req.getParameter("author"),
                            Double.parseDouble(req.getParameter("price"))));
                    tx.commit();
                }
                out.println("<h2>Book Details</h2><form>");
                out.println("<input type='hidden' name='action' value='save'>");
                out.println("Title <input name='title' required> Author <input name='author' required> Price <input name='price' type='number' step='0.01' required><button>Save</button></form>");
                List<Book> books = em.createQuery("select b from Book b order by b.id", Book.class).getResultList();
                for (Book b : books) out.printf("<p>%d | %s | %s | %.2f</p>",
                        b.getId(), esc(b.getTitle()), esc(b.getAuthor()), b.getPrice());
            } catch (Exception e) {
                res.setStatus(500); out.println("<p>Operation failed. Check server log and submitted values.</p>");
            } finally { em.close(); }
        }
    }
    private String esc(String s) {
        if (s == null) return "";
        return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;")
                .replace("\"","&quot;").replace("'","&#39;");
    }
}
```

Open `http://localhost:8080/JpaPractical/q13`. Add a book and reload to see the saved rows.

---

## Q14. JPA inventory application

**Files:** `InventoryItem.java`, `InventoryServlet.java` in `com.example.jpa`.

### `InventoryItem.java`

```java
package com.example.jpa;
import jakarta.persistence.*;

@Entity @Table(name="inventory_q14")
public class InventoryItem {
    @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
    private String itemName;
    private int quantity;
    public InventoryItem() {}
    public InventoryItem(String itemName, int quantity) { this.itemName=itemName; this.quantity=quantity; }
    public Long getId(){return id;} public String getItemName(){return itemName;} public int getQuantity(){return quantity;}
}
```

### `InventoryServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;

@WebServlet("/q14")
public class InventoryServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter(); EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();
    em.persist(new InventoryItem(req.getParameter("name"),Integer.parseInt(req.getParameter("quantity"))));tx.commit();}
   out.println("<h2>Inventory</h2><form><input type='hidden' name='action' value='save'>Item <input name='name' required> Quantity <input type='number' min='0' name='quantity' required><button>Save</button></form>");
   List<InventoryItem> rows=em.createQuery("select i from InventoryItem i order by i.id",InventoryItem.class).getResultList();
   for(InventoryItem i:rows)out.printf("<p>%d | %s | Quantity: %d</p>",i.getId(),esc(i.getItemName()),i.getQuantity());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not process inventory. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q14`. Enter an item name and quantity to save and display inventory.

---

## Q15. JPA guestbook to store and retrieve visitor details/comments

**Files:** `GuestEntry.java`, `GuestbookServlet.java`.

### `GuestEntry.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import java.time.LocalDateTime;

@Entity @Table(name="guestbook_q15")
public class GuestEntry {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String visitorName;
 @Column(length=1000) private String comment;
 private LocalDateTime createdAt;
 public GuestEntry(){}
 public GuestEntry(String visitorName,String comment){this.visitorName=visitorName;this.comment=comment;this.createdAt=LocalDateTime.now();}
 public Long getId(){return id;} public String getVisitorName(){return visitorName;} public String getComment(){return comment;} public LocalDateTime getCreatedAt(){return createdAt;}
}
```

### `GuestbookServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;

@WebServlet("/q15")
public class GuestbookServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new GuestEntry(req.getParameter("name"),req.getParameter("comment")));tx.commit();}
   out.println("<h2>Guestbook</h2><form><input type='hidden' name='action' value='save'>Name <input name='name' required><br>Comment <textarea name='comment' required></textarea><button>Submit</button></form>");
   List<GuestEntry> rows=em.createQuery("select g from GuestEntry g order by g.id desc",GuestEntry.class).getResultList();
   for(GuestEntry g:rows)out.printf("<article><b>%s</b> (%s)<p>%s</p></article>",esc(g.getVisitorName()),g.getCreatedAt(),esc(g.getComment()));
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save guestbook entry. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q15`. Each submitted visitor comment is stored and listed newest-first.

---

## Q16. JPA application demonstrating ORM associations

This example demonstrates a **one-to-many** association: one department has many employees.

**Files:** `Department.java`, `DepartmentEmployee.java`, `AssociationServlet.java`.

### `Department.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import java.util.ArrayList; import java.util.List;

@Entity @Table(name="department_q16")
public class Department {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name;
 @OneToMany(mappedBy="department",cascade=CascadeType.ALL,orphanRemoval=true)
 private List<DepartmentEmployee> employees=new ArrayList<>();
 public Department(){}
 public Department(String name){this.name=name;}
 public void addEmployee(DepartmentEmployee e){employees.add(e);e.setDepartment(this);}
 public Long getId(){return id;} public String getName(){return name;} public List<DepartmentEmployee> getEmployees(){return employees;}
}
```

### `DepartmentEmployee.java`

```java
package com.example.jpa;
import jakarta.persistence.*;

@Entity @Table(name="department_employee_q16")
public class DepartmentEmployee {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name;
 @ManyToOne(optional=false) @JoinColumn(name="department_id") private Department department;
 public DepartmentEmployee(){}
 public DepartmentEmployee(String name){this.name=name;}
 public void setDepartment(Department d){department=d;}
 public Long getId(){return id;} public String getName(){return name;} public Department getDepartment(){return department;}
}
```

### `AssociationServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;

@WebServlet("/q16")
public class AssociationServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();
    Department d=new Department(req.getParameter("department"));d.addEmployee(new DepartmentEmployee(req.getParameter("employee")));em.persist(d);tx.commit();}
   out.println("<h2>Department and Employees (One-to-Many)</h2><form><input type='hidden' name='action' value='save'>Department <input name='department' required> Employee <input name='employee' required><button>Save</button></form>");
   List<Department> rows=em.createQuery("select distinct d from Department d left join fetch d.employees order by d.id",Department.class).getResultList();
   for(Department d:rows){out.printf("<p><b>%s</b> — ",esc(d.getName()));for(DepartmentEmployee e:d.getEmployees())out.print(esc(e.getName())+" ");out.println("</p>");}
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save association. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q16`. Saving a department and employee demonstrates both sides of the relationship.

---

## Q17. JPA application to store employee names and salaries

**Files:** `EmployeeSalary.java`, `EmployeeSalaryServlet.java`.

### `EmployeeSalary.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="employee_salary_q17")
public class EmployeeSalary {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private double salary;
 public EmployeeSalary(){}
 public EmployeeSalary(String name,double salary){this.name=name;this.salary=salary;}
 public Long getId(){return id;} public String getName(){return name;} public double getSalary(){return salary;}
}
```

### `EmployeeSalaryServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q17")
public class EmployeeSalaryServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new EmployeeSalary(req.getParameter("name"),Double.parseDouble(req.getParameter("salary"))));tx.commit();}
   out.println("<h2>Employee Salary</h2><form><input type='hidden' name='action' value='save'>Name <input name='name' required> Salary <input type='number' min='0' step='0.01' name='salary' required><button>Save</button></form>");
   List<EmployeeSalary> rows=em.createQuery("select e from EmployeeSalary e order by e.id",EmployeeSalary.class).getResultList();
   for(EmployeeSalary e:rows)out.printf("<p>%d | %s | %.2f</p>",e.getId(),esc(e.getName()),e.getSalary());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save employee. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q17`.

---

## Q18. JPA application to store course details and demonstrate ORM mapping

**Files:** `Course.java`, `CourseServlet.java`.

### `Course.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="course_q18")
public class Course {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String courseName; private int durationMonths; private double fee;
 public Course(){}
 public Course(String courseName,int durationMonths,double fee){this.courseName=courseName;this.durationMonths=durationMonths;this.fee=fee;}
 public Long getId(){return id;} public String getCourseName(){return courseName;} public int getDurationMonths(){return durationMonths;} public double getFee(){return fee;}
}
```

### `CourseServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q18")
public class CourseServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new Course(req.getParameter("name"),Integer.parseInt(req.getParameter("duration")),Double.parseDouble(req.getParameter("fee"))));tx.commit();}
   out.println("<h2>Course Details — ORM Mapping</h2><form><input type='hidden' name='action' value='save'>Course <input name='name' required> Duration (months) <input type='number' min='1' name='duration' required> Fee <input type='number' min='0' step='0.01' name='fee' required><button>Save</button></form>");
   List<Course> rows=em.createQuery("select c from Course c order by c.id",Course.class).getResultList();
   for(Course c:rows)out.printf("<p>%d | %s | %d months | %.2f</p>",c.getId(),esc(c.getCourseName()),c.getDurationMonths(),c.getFee());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save course. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q18`. Hibernate maps the Java fields to database columns.

---

## Q19. JPA application to store product names and quantities

**Files:** `StockProduct.java`, `StockProductServlet.java`.

### `StockProduct.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="stock_product_q19")
public class StockProduct {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private int quantity;
 public StockProduct(){}
 public StockProduct(String name,int quantity){this.name=name;this.quantity=quantity;}
 public Long getId(){return id;} public String getName(){return name;} public int getQuantity(){return quantity;}
}
```

### `StockProductServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q19")
public class StockProductServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new StockProduct(req.getParameter("name"),Integer.parseInt(req.getParameter("quantity"))));tx.commit();}
   out.println("<h2>Product Quantity</h2><form><input type='hidden' name='action' value='save'>Product <input name='name' required> Quantity <input type='number' min='0' name='quantity' required><button>Save</button></form>");
   List<StockProduct> rows=em.createQuery("select p from StockProduct p order by p.id",StockProduct.class).getResultList();
   for(StockProduct p:rows)out.printf("<p>%d | %s | %d</p>",p.getId(),esc(p.getName()),p.getQuantity());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save product. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q19`.

---

## Q20. JPA application demonstrating one-to-one mapping

This example maps one `Person` to one `Passport`.

**Files:** `Person.java`, `Passport.java`, `OneToOneServlet.java`.

### `Person.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="person_q20")
public class Person {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name;
 @OneToOne(cascade=CascadeType.ALL,optional=false) @JoinColumn(name="passport_id",unique=true)
 private Passport passport;
 public Person(){}
 public Person(String name,Passport passport){this.name=name;this.passport=passport;}
 public Long getId(){return id;} public String getName(){return name;} public Passport getPassport(){return passport;}
}
```

### `Passport.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="passport_q20")
public class Passport {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 @Column(unique=true,nullable=false) private String passportNumber;
 public Passport(){}
 public Passport(String passportNumber){this.passportNumber=passportNumber;}
 public Long getId(){return id;} public String getPassportNumber(){return passportNumber;}
}
```

### `OneToOneServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q20")
public class OneToOneServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new Person(req.getParameter("name"),new Passport(req.getParameter("passport"))));tx.commit();}
   out.println("<h2>One-to-One Person–Passport</h2><form><input type='hidden' name='action' value='save'>Person name <input name='name' required> Passport number <input name='passport' required><button>Save</button></form>");
   List<Person> rows=em.createQuery("select p from Person p order by p.id",Person.class).getResultList();
   for(Person p:rows)out.printf("<p>%d | %s | Passport: %s</p>",p.getId(),esc(p.getName()),esc(p.getPassport().getPassportNumber()));
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save person/passport. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q20`. The unique join column demonstrates one-to-one mapping.

---

## Q21. JPA application to store student names and roll numbers

**Files:** `StudentRoll.java`, `StudentRollServlet.java`.

### `StudentRoll.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="student_roll_q21")
public class StudentRoll {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String studentName; private String rollNumber;
 public StudentRoll(){}
 public StudentRoll(String studentName,String rollNumber){this.studentName=studentName;this.rollNumber=rollNumber;}
 public Long getId(){return id;} public String getStudentName(){return studentName;} public String getRollNumber(){return rollNumber;}
}
```

### `StudentRollServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q21")
public class StudentRollServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new StudentRoll(req.getParameter("name"),req.getParameter("roll")));tx.commit();}
   out.println("<h2>Student Roll Numbers</h2><form><input type='hidden' name='action' value='save'>Name <input name='name' required> Roll number <input name='roll' required><button>Save</button></form>");
   List<StudentRoll> rows=em.createQuery("select s from StudentRoll s order by s.id",StudentRoll.class).getResultList();
   for(StudentRoll s:rows)out.printf("<p>%d | %s | Roll: %s</p>",s.getId(),esc(s.getStudentName()),esc(s.getRollNumber()));
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save student. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q21`.

---

## Q22. JPA application to store student names and marks

**Files:** `StudentMark.java`, `StudentMarkServlet.java`.

### `StudentMark.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="student_mark_q22")
public class StudentMark {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private double marks;
 public StudentMark(){}
 public StudentMark(String name,double marks){this.name=name;this.marks=marks;}
 public Long getId(){return id;} public String getName(){return name;} public double getMarks(){return marks;}
}
```

### `StudentMarkServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q22")
public class StudentMarkServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new StudentMark(req.getParameter("name"),Double.parseDouble(req.getParameter("marks"))));tx.commit();}
   out.println("<h2>Student Marks</h2><form><input type='hidden' name='action' value='save'>Name <input name='name' required> Marks <input type='number' min='0' max='100' step='0.01' name='marks' required><button>Save</button></form>");
   List<StudentMark> rows=em.createQuery("select s from StudentMark s order by s.id",StudentMark.class).getResultList();
   for(StudentMark s:rows)out.printf("<p>%d | %s | %.2f marks</p>",s.getId(),esc(s.getName()),s.getMarks());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save marks. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q22`.

---

## Q23. JPA application to store book names and prices

**Files:** `PricedBook.java`, `PricedBookServlet.java`.

### `PricedBook.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="priced_book_q23")
public class PricedBook {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private double price;
 public PricedBook(){}
 public PricedBook(String name,double price){this.name=name;this.price=price;}
 public Long getId(){return id;} public String getName(){return name;} public double getPrice(){return price;}
}
```

### `PricedBookServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q23")
public class PricedBookServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new PricedBook(req.getParameter("name"),Double.parseDouble(req.getParameter("price"))));tx.commit();}
   out.println("<h2>Book Names and Prices</h2><form><input type='hidden' name='action' value='save'>Book <input name='name' required> Price <input type='number' min='0' step='0.01' name='price' required><button>Save</button></form>");
   List<PricedBook> rows=em.createQuery("select b from PricedBook b order by b.id",PricedBook.class).getResultList();
   for(PricedBook b:rows)out.printf("<p>%d | %s | ₹%.2f</p>",b.getId(),esc(b.getName()),b.getPrice());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save book. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q23`.

---

## Q24. JPA student management system: insert, display, and update

**Files:** `ManagedStudent.java`, `StudentManagementServlet.java`.

### `ManagedStudent.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="managed_student_q24")
public class ManagedStudent {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private String course;
 public ManagedStudent(){}
 public ManagedStudent(String name,String course){this.name=name;this.course=course;}
 public Long getId(){return id;} public String getName(){return name;} public String getCourse(){return course;}
 public void setName(String name){this.name=name;} public void setCourse(String course){this.course=course;}
}
```

### `StudentManagementServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q24")
public class StudentManagementServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new ManagedStudent(req.getParameter("name"),req.getParameter("course")));tx.commit();}
   if("update".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();ManagedStudent s=em.find(ManagedStudent.class,Long.parseLong(req.getParameter("id")));if(s!=null){s.setName(req.getParameter("name"));s.setCourse(req.getParameter("course"));}tx.commit();}
   out.println("<h2>Student Management</h2><h3>Insert</h3><form><input type='hidden' name='action' value='save'>Name <input name='name' required> Course <input name='course' required><button>Insert</button></form>");
   out.println("<h3>Update by ID</h3><form><input type='hidden' name='action' value='update'>ID <input type='number' min='1' name='id' required> New name <input name='name' required> Course <input name='course' required><button>Update</button></form><h3>Records</h3>");
   List<ManagedStudent> rows=em.createQuery("select s from ManagedStudent s order by s.id",ManagedStudent.class).getResultList();
   for(ManagedStudent s:rows)out.printf("<p>ID %d | %s | %s</p>",s.getId(),esc(s.getName()),esc(s.getCourse()));
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Operation failed; verify the ID and server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q24`. Insert new students, then update a record using its displayed ID.

---

## Q25. JPA employee management: insert, display, and update

**Files:** `ManagedEmployee.java`, `EmployeeManagementServlet.java`.

### `ManagedEmployee.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="managed_employee_q25")
public class ManagedEmployee {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private String department; private double salary;
 public ManagedEmployee(){}
 public ManagedEmployee(String name,String department,double salary){this.name=name;this.department=department;this.salary=salary;}
 public Long getId(){return id;} public String getName(){return name;} public String getDepartment(){return department;} public double getSalary(){return salary;}
 public void setName(String name){this.name=name;} public void setDepartment(String department){this.department=department;} public void setSalary(double salary){this.salary=salary;}
}
```

### `EmployeeManagementServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q25")
public class EmployeeManagementServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new ManagedEmployee(req.getParameter("name"),req.getParameter("department"),Double.parseDouble(req.getParameter("salary"))));tx.commit();}
   if("update".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();ManagedEmployee e=em.find(ManagedEmployee.class,Long.parseLong(req.getParameter("id")));if(e!=null){e.setName(req.getParameter("name"));e.setDepartment(req.getParameter("department"));e.setSalary(Double.parseDouble(req.getParameter("salary")));}tx.commit();}
   out.println("<h2>Employee Management</h2><form><input type='hidden' name='action' value='save'>Name <input name='name' required> Department <input name='department' required> Salary <input type='number' min='0' step='0.01' name='salary' required><button>Insert</button></form>");
   out.println("<h3>Update by ID</h3><form><input type='hidden' name='action' value='update'>ID <input type='number' min='1' name='id' required> Name <input name='name' required> Department <input name='department' required> Salary <input type='number' min='0' step='0.01' name='salary' required><button>Update</button></form><h3>Records</h3>");
   List<ManagedEmployee> rows=em.createQuery("select e from ManagedEmployee e order by e.id",ManagedEmployee.class).getResultList();
   for(ManagedEmployee e:rows)out.printf("<p>ID %d | %s | %s | ₹%.2f</p>",e.getId(),esc(e.getName()),esc(e.getDepartment()),e.getSalary());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Operation failed; verify values and server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q25`. Use the ID from the displayed records to update an employee.

---

## Q26. JPA grocery store: insert/display product ID, name, quantity, and price

**Files:** `GroceryProduct.java`, `GroceryServlet.java`.

### `GroceryProduct.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="grocery_product_q26")
public class GroceryProduct {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String name; private int quantity; private double price;
 public GroceryProduct(){}
 public GroceryProduct(String name,int quantity,double price){this.name=name;this.quantity=quantity;this.price=price;}
 public Long getId(){return id;} public String getName(){return name;} public int getQuantity(){return quantity;} public double getPrice(){return price;}
}
```

### `GroceryServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q26")
public class GroceryServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new GroceryProduct(req.getParameter("name"),Integer.parseInt(req.getParameter("quantity")),Double.parseDouble(req.getParameter("price"))));tx.commit();}
   out.println("<h2>Grocery Store</h2><form><input type='hidden' name='action' value='save'>Product <input name='name' required> Quantity <input type='number' min='0' name='quantity' required> Price <input type='number' min='0' step='0.01' name='price' required><button>Insert</button></form><h3>Products</h3>");
   List<GroceryProduct> rows=em.createQuery("select p from GroceryProduct p order by p.id",GroceryProduct.class).getResultList();
   out.println("<table border='1'><tr><th>ID</th><th>Name</th><th>Quantity</th><th>Price</th></tr>");
   for(GroceryProduct p:rows)out.printf("<tr><td>%d</td><td>%s</td><td>%d</td><td>₹%.2f</td></tr>",p.getId(),esc(p.getName()),p.getQuantity(),p.getPrice());out.println("</table>");
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save grocery product. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q26`.

---

## Q27. JPA student result system: roll number, name, subject, and marks

**Files:** `StudentResult.java`, `StudentResultServlet.java`.

### `StudentResult.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="student_result_q27")
public class StudentResult {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String rollNumber; private String studentName; private String subject; private double marks;
 public StudentResult(){}
 public StudentResult(String rollNumber,String studentName,String subject,double marks){this.rollNumber=rollNumber;this.studentName=studentName;this.subject=subject;this.marks=marks;}
 public Long getId(){return id;} public String getRollNumber(){return rollNumber;} public String getStudentName(){return studentName;} public String getSubject(){return subject;} public double getMarks(){return marks;}
}
```

### `StudentResultServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q27")
public class StudentResultServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new StudentResult(req.getParameter("roll"),req.getParameter("name"),req.getParameter("subject"),Double.parseDouble(req.getParameter("marks"))));tx.commit();}
   out.println("<h2>Student Result</h2><form><input type='hidden' name='action' value='save'>Roll number <input name='roll' required> Student name <input name='name' required> Subject <input name='subject' required> Marks <input type='number' min='0' max='100' step='0.01' name='marks' required><button>Save</button></form>");
   List<StudentResult> rows=em.createQuery("select s from StudentResult s order by s.id",StudentResult.class).getResultList();
   out.println("<table border='1'><tr><th>ID</th><th>Roll No.</th><th>Name</th><th>Subject</th><th>Marks</th></tr>");
   for(StudentResult s:rows)out.printf("<tr><td>%d</td><td>%s</td><td>%s</td><td>%s</td><td>%.2f</td></tr>",s.getId(),esc(s.getRollNumber()),esc(s.getStudentName()),esc(s.getSubject()),s.getMarks());out.println("</table>");
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save result. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q27`.

---

## Q28. JPA mobile shop: insert, display, and update mobile details

**Files:** `Mobile.java`, `MobileShopServlet.java`.

### `Mobile.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="mobile_q28")
public class Mobile {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String brand; private String model; private double price; private int quantity;
 public Mobile(){}
 public Mobile(String brand,String model,double price,int quantity){this.brand=brand;this.model=model;this.price=price;this.quantity=quantity;}
 public Long getId(){return id;} public String getBrand(){return brand;} public String getModel(){return model;} public double getPrice(){return price;} public int getQuantity(){return quantity;}
 public void setBrand(String brand){this.brand=brand;} public void setModel(String model){this.model=model;} public void setPrice(double price){this.price=price;} public void setQuantity(int quantity){this.quantity=quantity;}
}
```

### `MobileShopServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q28")
public class MobileShopServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new Mobile(req.getParameter("brand"),req.getParameter("model"),Double.parseDouble(req.getParameter("price")),Integer.parseInt(req.getParameter("quantity"))));tx.commit();}
   if("update".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();Mobile m=em.find(Mobile.class,Long.parseLong(req.getParameter("id")));if(m!=null){m.setBrand(req.getParameter("brand"));m.setModel(req.getParameter("model"));m.setPrice(Double.parseDouble(req.getParameter("price")));m.setQuantity(Integer.parseInt(req.getParameter("quantity")));}tx.commit();}
   out.println("<h2>Mobile Shop</h2><h3>Insert mobile</h3><form><input type='hidden' name='action' value='save'>Brand <input name='brand' required> Model <input name='model' required> Price <input type='number' min='0' step='0.01' name='price' required> Quantity <input type='number' min='0' name='quantity' required><button>Insert</button></form>");
   out.println("<h3>Update by ID</h3><form><input type='hidden' name='action' value='update'>ID <input type='number' min='1' name='id' required> Brand <input name='brand' required> Model <input name='model' required> Price <input type='number' min='0' step='0.01' name='price' required> Quantity <input type='number' min='0' name='quantity' required><button>Update</button></form><h3>Mobiles</h3>");
   List<Mobile> rows=em.createQuery("select m from Mobile m order by m.id",Mobile.class).getResultList();
   for(Mobile m:rows)out.printf("<p>ID %d | %s %s | ₹%.2f | Qty %d</p>",m.getId(),esc(m.getBrand()),esc(m.getModel()),m.getPrice(),m.getQuantity());
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Operation failed; check mobile ID, values, and server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q28`. Insert a mobile, note its ID, and use that ID to update it.

---

## Q29. JPA vehicle showroom: insert and display vehicle ID, model, and price

**Files:** `Vehicle.java`, `VehicleShowroomServlet.java`.

### `Vehicle.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="vehicle_q29")
public class Vehicle {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String model; private double price;
 public Vehicle(){}
 public Vehicle(String model,double price){this.model=model;this.price=price;}
 public Long getId(){return id;} public String getModel(){return model;} public double getPrice(){return price;}
}
```

### `VehicleShowroomServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q29")
public class VehicleShowroomServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new Vehicle(req.getParameter("model"),Double.parseDouble(req.getParameter("price"))));tx.commit();}
   out.println("<h2>Vehicle Showroom</h2><form><input type='hidden' name='action' value='save'>Model <input name='model' required> Price <input type='number' min='0' step='0.01' name='price' required><button>Insert</button></form><h3>Vehicles</h3>");
   List<Vehicle> rows=em.createQuery("select v from Vehicle v order by v.id",Vehicle.class).getResultList();
   out.println("<table border='1'><tr><th>Vehicle ID</th><th>Model</th><th>Price</th></tr>");
   for(Vehicle v:rows)out.printf("<tr><td>%d</td><td>%s</td><td>₹%.2f</td></tr>",v.getId(),esc(v.getModel()),v.getPrice());out.println("</table>");
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save vehicle. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q29`.

---

## Q30. JPA bookshop: insert and display book details

**Files:** `ShopBook.java`, `BookshopServlet.java`.

### `ShopBook.java`

```java
package com.example.jpa;
import jakarta.persistence.*;
@Entity @Table(name="shop_book_q30")
public class ShopBook {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY) private Long id;
 private String title; private String author; private double price; private int quantity;
 public ShopBook(){}
 public ShopBook(String title,String author,double price,int quantity){this.title=title;this.author=author;this.price=price;this.quantity=quantity;}
 public Long getId(){return id;} public String getTitle(){return title;} public String getAuthor(){return author;} public double getPrice(){return price;} public int getQuantity(){return quantity;}
}
```

### `BookshopServlet.java`

```java
package com.example.jpa;
import jakarta.persistence.*; import jakarta.servlet.annotation.WebServlet; import jakarta.servlet.http.*;
import java.io.*; import java.util.List;
@WebServlet("/q30")
public class BookshopServlet extends HttpServlet {
 protected void service(HttpServletRequest req,HttpServletResponse res)throws IOException{
  res.setContentType("text/html;charset=UTF-8");
  try(PrintWriter out=res.getWriter();EntityManager em=JpaUtil.getEntityManager()){
   if("save".equals(req.getParameter("action"))){EntityTransaction tx=em.getTransaction();tx.begin();em.persist(new ShopBook(req.getParameter("title"),req.getParameter("author"),Double.parseDouble(req.getParameter("price")),Integer.parseInt(req.getParameter("quantity"))));tx.commit();}
   out.println("<h2>Bookshop</h2><form><input type='hidden' name='action' value='save'>Title <input name='title' required> Author <input name='author' required> Price <input type='number' min='0' step='0.01' name='price' required> Quantity <input type='number' min='0' name='quantity' required><button>Insert</button></form><h3>Books</h3>");
   List<ShopBook> rows=em.createQuery("select b from ShopBook b order by b.id",ShopBook.class).getResultList();
   out.println("<table border='1'><tr><th>ID</th><th>Title</th><th>Author</th><th>Price</th><th>Quantity</th></tr>");
   for(ShopBook b:rows)out.printf("<tr><td>%d</td><td>%s</td><td>%s</td><td>₹%.2f</td><td>%d</td></tr>",b.getId(),esc(b.getTitle()),esc(b.getAuthor()),b.getPrice(),b.getQuantity());out.println("</table>");
  }catch(Exception e){res.setStatus(500);res.getWriter().println("Could not save book. Check server log.");}
 }
 private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

Open `/JpaPractical/q30`.

---

## Common troubleshooting

1. **`No Persistence provider for EntityManager named EJPU`** — verify the path is exactly `src/main/resources/META-INF/persistence.xml`, the unit name is `EJPU`, and Hibernate is included in Maven dependencies.
2. **`Access denied for user`** — update the MySQL username/password in `persistence.xml`.
3. **`Unknown database 'jpa_practical'`** — create the database using the SQL shown above.
4. **`javax.servlet` or `javax.persistence` errors** — use Jakarta imports and Tomcat 10.1+; older Tomcat versions require a different legacy stack.
5. **HTTP 404** — check the WAR context path and use the servlet URL, for example `/JpaPractical/q24`.
6. **HTTP 500 / number format error** — enter valid numeric values. Check the Tomcat console for the full exception.
7. **Tables not created** — confirm MySQL is running and `hibernate.hbm2ddl.auto` is set to `update`.

## Viva quick revision

- **JPA:** Java specification for object-relational mapping.
- **Entity:** Java class mapped to a database table using `@Entity`.
- **`@Id`:** Marks the primary key.
- **`@GeneratedValue`:** Configures automatic ID generation.
- **`EntityManager`:** Performs persistence operations such as `persist`, `find`, and queries.
- **JPQL:** Query language that works with entity classes and fields, not directly with table names.
- **`persist()`:** Makes a new entity managed so it can be inserted.
- **`find()`:** Retrieves an entity by primary key.
- **Transaction:** Groups database changes; in these examples, `begin()` and `commit()` surround writes.
- **ORM association:** Relationship between entity objects; Q16 demonstrates one-to-many and Q20 demonstrates one-to-one.
- **`persistence.xml`:** Configures the persistence unit, database connection, provider, and Hibernate options.

