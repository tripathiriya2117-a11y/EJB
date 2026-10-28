# Enterprise Java Question Bank — Q.1 B
## JSP, JDBC and Web Forms — Complete Solutions (Questions 27–47)

**Environment:** NetBeans, JDK, Apache Tomcat 10.1.x, MySQL 8.x. Use `jakarta.servlet.*` for Java Servlets on Tomcat 10.1. JSP pages are placed under **Web Pages**. Java servlet/helper classes go under **Source Packages**.

> **How to use this guide:** Each numbered solution is independent. Create a separate NetBeans Web Application project for each question, or keep the files for only one question in a project at a time. Do not put every `index.jsp` from this guide into the same project because filenames overlap.

## Common setup

1. In NetBeans choose **File → New Project → Java with Ant → Web Application** (or the Web Application template available in your NetBeans version).
2. Give the project the name listed for that question and select **Apache Tomcat 10.1**.
3. Put `.jsp`, `.html`, and `WEB-INF` files under **Web Pages**. Put `.java` files under **Source Packages** in the package specified in the code.
4. Run the project. Open the indicated landing page, for example `http://localhost:8080/ProjectName/index.jsp`.
5. For JDBC questions, start MySQL and add MySQL Connector/J to the project's Libraries. Update the database username and password in the code to match your own machine. Never commit real credentials to GitHub.

### Shared MySQL database for JDBC examples

Run in MySQL Workbench:

```sql
CREATE DATABASE IF NOT EXISTS ejdb;
USE ejdb;

CREATE TABLE IF NOT EXISTS emp (
  eno INT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  age INT,
  designation VARCHAR(100),
  salary DECIMAL(10,2)
);

CREATE TABLE IF NOT EXISTS users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(60) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  email VARCHAR(150)
);

CREATE TABLE IF NOT EXISTS gym_members (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(150) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  plan VARCHAR(40) NOT NULL
);
```

Add the MySQL Connector/J `.jar` via **Project → Properties → Libraries → Add JAR/Folder** if your project doesn't already have the driver.

For database code below, use this connection pattern and replace the credentials:

```java
DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC",
    "root",
    "YOUR_MYSQL_PASSWORD"
);
```

---

# Q.1 B — JSP, JDBC and Web Forms

## Q27. JSP application to display values obtained from different implicit objects

**Project:** `JSPImplicitObjects`  
**Path:** `Web Pages/index.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html><head><title>JSP Implicit Objects</title></head><body>
<h2>JSP Implicit Objects</h2>
<p>Request method: <%= request.getMethod() %></p>
<p>Request URI: <%= request.getRequestURI() %></p>
<p>Response content type: <%= response.getContentType() %></p>
<p>Session ID: <%= session.getId() %></p>
<p>Application/server info: <%= application.getServerInfo() %></p>
<p>Servlet name: <%= config.getServletName() %></p>
<p>Page object class: <%= page.getClass().getName() %></p>
<p>PageContext request URI: <%= pageContext.getRequest().getRequestURI() %></p>
</body></html>
```

**Execution:** Run `JSPImplicitObjects` and open `/index.jsp`. Refresh the page to see the request URI and session ID. Some values depend on the server and URL.

## Q28. JSP application demonstrating at least three methods of four implicit objects

This example demonstrates methods of `request`, `response`, `session`, and `application`.

**Project:** `JSPImplicitMethods`  
**Path:** `Web Pages/index.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html><head><title>Implicit Object Methods</title></head><body>
<h2>Implicit Object Methods</h2>
<h3>Request</h3>
<p>Method: <%= request.getMethod() %></p>
<p>URI: <%= request.getRequestURI() %></p>
<p>Protocol: <%= request.getProtocol() %></p>
<h3>Response</h3>
<p>Content type before setting: <%= response.getContentType() %></p>
<% response.setContentType("text/html;charset=UTF-8"); %>
<p>Character encoding: <%= response.getCharacterEncoding() %></p>
<h3>Session</h3>
<p>ID: <%= session.getId() %></p>
<p>New session: <%= session.isNew() %></p>
<p>Creation time: <%= new java.util.Date(session.getCreationTime()) %></p>
<h3>Application</h3>
<p>Server info: <%= application.getServerInfo() %></p>
<p>Context path: <%= application.getContextPath() %></p>
<p>Major version: <%= application.getMajorVersion() %></p>
</body></html>
```

**Execution:** Run the project and visit `/index.jsp`. The response content type is most reliably set in the page directive; the response method calls are shown for learning purposes.

## Q29. JSP application demonstrating Expression Language (EL)

**Project:** `JSPExpressionLanguage`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><head><title>EL Demo</title></head><body>
<h2>Enter Student Details</h2>
<form action="display.jsp" method="post">
  Roll number: <input name="rollNo" required><br><br>
  Name: <input name="name" required><br><br>
  Course: <input name="course" required><br><br>
  Marks: <input type="number" name="marks" min="0" max="100" required><br><br>
  <button type="submit">Display</button>
</form>
</body></html>
```

**Path 2 — `Web Pages/display.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><head><title>EL Result</title></head><body>
<h2>Student Details</h2>
<p>Roll number: ${param.rollNo}</p>
<p>Name: ${param.name}</p>
<p>Course: ${param.course}</p>
<p>Marks: ${param.marks}</p>
<a href="index.jsp">Back</a>
</body></html>
```

**Execution:** Run the project, fill in the form, and submit. `${param.name}` reads the `name` request parameter using Expression Language.

## Q30. JSP application demonstrating JSTL

**Project:** `JSPJSTLDemo`

JSTL needs the Jakarta Tags library compatible with your JSP/container setup. For a Maven web project, add the JSTL API and implementation dependencies compatible with Jakarta Tags 3.x. If using a non-Maven NetBeans project, add the corresponding Jakarta Tags JARs to Libraries.

**Path — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<!DOCTYPE html><html><head><title>JSTL Demo</title></head><body>
<h2>JSTL Core Tags</h2>
<c:set var="studentName" value="Riya"/>
<c:set var="marks" value="82"/>
<p>Name: <c:out value="${studentName}"/></p>
<p>Marks: <c:out value="${marks}"/></p>
<c:if test="${marks >= 40}">
  <p>Status: Pass</p>
</c:if>
<c:choose>
  <c:when test="${marks >= 75}"><p>Grade: Distinction</p></c:when>
  <c:when test="${marks >= 60}"><p>Grade: First Class</p></c:when>
  <c:otherwise><p>Grade: Pass</p></c:otherwise>
</c:choose>
<c:forEach var="subject" items="${['Java','JSP','JDBC']}">
  <p>Subject: <c:out value="${subject}"/></p>
</c:forEach>
</body></html>
```

**Execution:** Add the compatible Jakarta Tags library, run the project and open `/index.jsp`. If the container does not support list literals in EL, replace the `forEach` sample with a list supplied by a servlet or use three `<c:out>` lines.

## Q31. JSP application demonstrating JSP Action Tags

**Project:** `JSPActionTags`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><head><title>JSP Action Tags</title></head><body>
<h2>JSP Action Tags</h2>
<jsp:include page="header.jsp"/>
<form action="index.jsp" method="get">
  Enter your name: <input name="name" required>
  <button type="submit">Submit</button>
</form>
<jsp:useBean id="student" class="demo.StudentBean" scope="request"/>
<jsp:setProperty name="student" property="name" param="name"/>
<p>Name from JavaBean: <jsp:getProperty name="student" property="name"/></p>
</body></html>
```

**Path 2 — `Web Pages/header.jsp`**
```jsp
<h3>Welcome to the JSP Action Tags demonstration</h3>
```

**Path 3 — `Source Packages/demo/StudentBean.java`**
```java
package demo;
import java.io.Serializable;

public class StudentBean implements Serializable {
    private String name = "";
    public StudentBean() {}
    public String getName() { return name; }
    public void setName(String name) { this.name = name == null ? "" : name; }
}
```

**Execution:** Create package `demo`, add `StudentBean.java`, and create the two JSP files. Run `/index.jsp`, enter a name, and submit. This demonstrates `<jsp:include>`, `<jsp:useBean>`, `<jsp:setProperty>`, and `<jsp:getProperty>`.

## Q32. JSP application passing values between pages and validating name, age, hobbies, email and gender

**Project:** `JSPFormValidation`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><head><title>Profile Form</title></head><body>
<h2>Profile Form</h2>
<form action="validate.jsp" method="post">
  Name: <input name="name" required minlength="2"><br><br>
  Age: <input type="number" name="age" min="1" max="120" required><br><br>
  Hobbies: <input name="hobbies" required><br><br>
  Email: <input type="email" name="email" required><br><br>
  Gender:
  <label><input type="radio" name="gender" value="Female" required> Female</label>
  <label><input type="radio" name="gender" value="Male"> Male</label>
  <label><input type="radio" name="gender" value="Other"> Other</label><br><br>
  <button type="submit">Submit</button>
</form>
</body></html>
```

**Path 2 — `Web Pages/validate.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%!
  private String esc(String s) {
    if (s == null) return "";
    return s.replace("&","&amp;").replace("<","&lt;")
            .replace(">","&gt;").replace("\"","&quot;").replace("'","&#x27;");
  }
%>
<%
String name = request.getParameter("name");
String ageText = request.getParameter("age");
String hobbies = request.getParameter("hobbies");
String email = request.getParameter("email");
String gender = request.getParameter("gender");
String error = null;
int age = -1;
try { age = Integer.parseInt(ageText); } catch (Exception ex) { error = "Enter a valid age."; }
if (name == null || name.trim().length() < 2) error = "Name must contain at least 2 characters.";
else if (age < 1 || age > 120) error = "Age must be between 1 and 120.";
else if (hobbies == null || hobbies.trim().isEmpty()) error = "Enter at least one hobby.";
else if (email == null || !email.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$")) error = "Enter a valid email.";
else if (gender == null || gender.isBlank()) error = "Select a gender.";
%>
<!DOCTYPE html><html><head><title>Validation Result</title></head><body>
<% if (error != null) { %>
  <h2>Validation failed</h2><p><%= esc(error) %></p><a href="index.jsp">Try again</a>
<% } else { %>
  <h2>Submitted Details</h2>
  <p>Name: <%= esc(name) %></p><p>Age: <%= age %></p>
  <p>Hobbies: <%= esc(hobbies) %></p><p>Email: <%= esc(email) %></p>
  <p>Gender: <%= esc(gender) %></p>
<% } %>
</body></html>
```

**Execution:** Run `/index.jsp`, submit valid values and then test invalid values. Browser validation is for convenience; the JSP also validates on the server.

## Q33. JSP page accepting employee number, name and salary and inserting into an emp table

**Project:** `JSPInsertEmployee`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><head><title>Insert Employee</title></head><body>
<h2>Insert Employee</h2>
<form action="insertEmployee.jsp" method="post">
  Employee number: <input type="number" name="eno" min="1" required><br><br>
  Name: <input name="name" required><br><br>
  Salary: <input type="number" name="salary" min="0" step="0.01" required><br><br>
  <button type="submit">Save</button>
</form>
</body></html>
```

**Path 2 — `Web Pages/insertEmployee.jsp`**
```jsp
<%@ page import="java.sql.*" contentType="text/html;charset=UTF-8" language="java" %>
<%
String message;
try {
    int eno = Integer.parseInt(request.getParameter("eno"));
    String name = request.getParameter("name");
    java.math.BigDecimal salary = new java.math.BigDecimal(request.getParameter("salary"));
    if (name == null || name.trim().isEmpty() || salary.signum() < 0) throw new IllegalArgumentException();
    String url = "jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC";
    try (Connection con = DriverManager.getConnection(url, "root", "YOUR_MYSQL_PASSWORD");
         PreparedStatement ps = con.prepareStatement(
             "INSERT INTO emp(eno,name,salary) VALUES(?,?,?)")) {
        ps.setInt(1, eno); ps.setString(2, name.trim()); ps.setBigDecimal(3, salary);
        ps.executeUpdate();
        message = "Employee inserted successfully.";
    }
} catch (SQLIntegrityConstraintViolationException ex) {
    message = "Employee number already exists.";
} catch (Exception ex) {
    message = "Unable to insert employee. Check the input, database and connection settings.";
}
%>
<!DOCTYPE html><html><body><h2><%= message %></h2><a href="index.jsp">Back</a></body></html>
```

**Database:** Use the shared `ejdb.emp` table. The `age` and `designation` columns are nullable for this question.  
**Execution:** Add Connector/J, change the password, run `/index.jsp`, and insert an employee. Verify with `SELECT * FROM ejdb.emp;`.

## Q34. HTML form submits employee number, name, age, designation and salary to update matching employee record

**Project:** `JSPUpdateEmployee`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Update Employee</h2>
<form action="updateEmployee.jsp" method="post">
  Employee number: <input type="number" name="eno" min="1" required><br><br>
  Name: <input name="name" required><br><br>
  Age: <input type="number" name="age" min="18" max="100" required><br><br>
  Designation: <input name="designation" required><br><br>
  Salary: <input type="number" name="salary" min="0" step="0.01" required><br><br>
  <button type="submit">Update</button>
</form></body></html>
```

**Path 2 — `Web Pages/updateEmployee.jsp`**
```jsp
<%@ page import="java.sql.*" contentType="text/html;charset=UTF-8" language="java" %>
<%
String message;
try {
    int eno = Integer.parseInt(request.getParameter("eno"));
    String name = request.getParameter("name");
    int age = Integer.parseInt(request.getParameter("age"));
    String designation = request.getParameter("designation");
    java.math.BigDecimal salary = new java.math.BigDecimal(request.getParameter("salary"));
    if (name == null || name.isBlank() || designation == null || designation.isBlank()
            || age < 18 || age > 100 || salary.signum() < 0) throw new IllegalArgumentException();
    String url = "jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC";
    try (Connection con = DriverManager.getConnection(url, "root", "YOUR_MYSQL_PASSWORD");
         PreparedStatement ps = con.prepareStatement(
             "UPDATE emp SET name=?, age=?, designation=?, salary=? WHERE eno=?")) {
        ps.setString(1,name.trim()); ps.setInt(2,age); ps.setString(3,designation.trim());
        ps.setBigDecimal(4,salary); ps.setInt(5,eno);
        int rows = ps.executeUpdate();
        message = rows == 1 ? "Employee updated successfully." : "No employee found with that number.";
    }
} catch (Exception ex) {
    message = "Update failed. Check input, database table and connection settings.";
}
%>
<!DOCTYPE html><html><body><h2><%= message %></h2><a href="index.jsp">Back</a></body></html>
```

**Execution:** Ensure the employee number already exists in `ejdb.emp`; submit the matching number and updated values. Verify using `SELECT * FROM emp WHERE eno=...;`.

## Q35. JSP application authenticating a user's login against a database

**Project:** `JSPDatabaseLogin`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Database Login</h2>
<form action="login.jsp" method="post">
  Username: <input name="username" required><br><br>
  Password: <input type="password" name="password" required><br><br>
  <button type="submit">Login</button>
</form></body></html>
```

**Path 2 — `Web Pages/login.jsp`**
```jsp
<%@ page import="java.sql.*,java.security.MessageDigest,java.nio.charset.StandardCharsets"
 contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String hash(String value) throws Exception {
    byte[] bytes = MessageDigest.getInstance("SHA-256").digest(value.getBytes(StandardCharsets.UTF_8));
    StringBuilder out = new StringBuilder();
    for (byte b : bytes) out.append(String.format("%02x", b));
    return out.toString();
}
private String esc(String s) {
    if (s == null) return "";
    return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;");
}
%>
<%
String username = request.getParameter("username");
String password = request.getParameter("password");
boolean valid = false;
try {
  String url = "jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC";
  try (Connection con = DriverManager.getConnection(url,"root","YOUR_MYSQL_PASSWORD");
       PreparedStatement ps = con.prepareStatement(
         "SELECT id FROM users WHERE username=? AND password_hash=?")) {
    ps.setString(1, username);
    ps.setString(2, hash(password == null ? "" : password));
    try (ResultSet rs = ps.executeQuery()) { valid = rs.next(); }
  }
} catch (Exception ex) { application.log("Database login failed", ex); }
%>
<!DOCTYPE html><html><body>
<% if (valid) { %><h2>Login successful. Hello, <%= esc(username) %>!</h2>
<% } else { %><h2>Login failed. Check your username and password.</h2><% } %>
<a href="index.jsp">Back</a></body></html>
```

**Note:** This compact example uses SHA-256 to avoid storing plaintext passwords, but production systems should use a password-specific algorithm such as BCrypt/Argon2 and a proper registration flow.  
**Execution:** Insert a test user with `password_hash` equal to the SHA-256 hash of the password (or create a matching registration flow); then test valid and invalid logins.

## Q36. JSP and JDBC gym membership portal with registration and login

**Project:** `JSPGymPortal`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Gym Membership</h2>
<h3>Register</h3><form action="register.jsp" method="post">
  Name: <input name="name" required><br><br>
  Email: <input type="email" name="email" required><br><br>
  Password: <input type="password" name="password" minlength="8" required><br><br>
  Plan: <select name="plan"><option>Monthly</option><option>Quarterly</option><option>Annual</option></select><br><br>
  <button>Register</button>
</form>
<h3>Login</h3><form action="login.jsp" method="post">
  Email: <input type="email" name="email" required><br><br>
  Password: <input type="password" name="password" required><br><br>
  <button>Login</button>
</form></body></html>
```

**Path 2 — `Web Pages/register.jsp`**
```jsp
<%@ page import="java.sql.*,java.security.MessageDigest,java.nio.charset.StandardCharsets"
 contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String hash(String s) throws Exception {
 byte[] a=MessageDigest.getInstance("SHA-256").digest(s.getBytes(StandardCharsets.UTF_8));
 StringBuilder b=new StringBuilder(); for(byte x:a)b.append(String.format("%02x",x)); return b.toString();
}
%>
<%
String msg="Registration failed.";
try {
 String name=request.getParameter("name"), email=request.getParameter("email");
 String password=request.getParameter("password"), plan=request.getParameter("plan");
 if(name==null||name.isBlank()||email==null||!email.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$")
    ||password==null||password.length()<8) throw new IllegalArgumentException();
 String url="jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC";
 try(Connection c=DriverManager.getConnection(url,"root","YOUR_MYSQL_PASSWORD");
     PreparedStatement p=c.prepareStatement(
       "INSERT INTO gym_members(name,email,password_hash,plan) VALUES(?,?,?,?)")){
   p.setString(1,name.trim());p.setString(2,email.trim());p.setString(3,hash(password));p.setString(4,plan);
   p.executeUpdate(); msg="Registration successful. You can now log in.";
 }
} catch(SQLIntegrityConstraintViolationException e){msg="This email is already registered.";}
  catch(Exception e){msg="Registration failed. Check the details and database connection.";}
%>
<!DOCTYPE html><html><body><h2><%=msg%></h2><a href="index.jsp">Back</a></body></html>
```

**Path 3 — `Web Pages/login.jsp`**
```jsp
<%@ page import="java.sql.*,java.security.MessageDigest,java.nio.charset.StandardCharsets"
 contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String hash(String s) throws Exception {
 byte[] a=MessageDigest.getInstance("SHA-256").digest(s.getBytes(StandardCharsets.UTF_8));
 StringBuilder b=new StringBuilder(); for(byte x:a)b.append(String.format("%02x",x)); return b.toString();
}
private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
%>
<%
String email=request.getParameter("email"), password=request.getParameter("password");
String memberName=null, plan=null;
try(Connection c=DriverManager.getConnection("jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC",
 "root","YOUR_MYSQL_PASSWORD");
 PreparedStatement p=c.prepareStatement("SELECT name,plan FROM gym_members WHERE email=? AND password_hash=?")){
 p.setString(1,email);p.setString(2,hash(password==null?"":password));
 try(ResultSet r=p.executeQuery()){if(r.next()){memberName=r.getString("name");plan=r.getString("plan");}}
}catch(Exception e){application.log("Gym login failed",e);}
%>
<!DOCTYPE html><html><body>
<%if(memberName!=null){session.setAttribute("gymMember",memberName);%>
<h2>Welcome, <%=esc(memberName)%>!</h2><p>Membership plan: <%=esc(plan)%></p>
<%}else{%><h2>Login failed. Check your email and password.</h2><%}%>
<a href="index.jsp">Back</a></body></html>
```

**Execution:** Create the `gym_members` table from the shared SQL. Add Connector/J and change the database password. Register a member, then log in with the same credentials.

## Q37. JSP event registration form with validation and submitted details

**Project:** `JSPEventRegistration`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Event Registration</h2>
<form action="result.jsp" method="post">
  Name: <input name="name" required minlength="2"><br><br>
  Email: <input type="email" name="email" required><br><br>
  Age: <input type="number" name="age" min="16" max="100" required><br><br>
  Contact number: <input name="contact" pattern="[0-9]{10}" title="Enter 10 digits" required><br><br>
  <button>Register</button>
</form></body></html>
```

**Path 2 — `Web Pages/result.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String esc(String s){if(s==null)return "";return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;");}
%>
<%
String name=request.getParameter("name"), email=request.getParameter("email");
String ageText=request.getParameter("age"), contact=request.getParameter("contact"), error=null;
int age=0;
try{age=Integer.parseInt(ageText);}catch(Exception e){error="Enter a valid age.";}
if(name==null||name.trim().length()<2)error="Enter a name of at least two characters.";
else if(email==null||!email.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$"))error="Enter a valid email.";
else if(age<16||age>100)error="Age must be between 16 and 100.";
else if(contact==null||!contact.matches("[0-9]{10}"))error="Contact must contain exactly 10 digits.";
%>
<!DOCTYPE html><html><body>
<%if(error!=null){%><h2>Registration error</h2><p><%=esc(error)%></p><a href="index.jsp">Try again</a>
<%}else{%><h2>Registration successful</h2><p>Name: <%=esc(name)%></p>
<p>Email: <%=esc(email)%></p><p>Age: <%=age%></p><p>Contact: <%=esc(contact)%></p><%}%>
</body></html>
```

**Execution:** Run the form, submit valid values, and try an invalid email or contact number to verify server-side validation.

## Q38. JSP registration and login application

**Project:** `JSPRegistrationLogin`

This solution reuses the `users` table and the same database connection pattern as Q35.

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Register</h2>
<form action="register.jsp" method="post">
  Username: <input name="username" required minlength="3"><br><br>
  Email: <input type="email" name="email" required><br><br>
  Password: <input type="password" name="password" minlength="8" required><br><br>
  <button>Register</button>
</form>
<h2>Login</h2><form action="login.jsp" method="post">
  Username: <input name="username" required><br><br>
  Password: <input type="password" name="password" required><br><br>
  <button>Login</button>
</form></body></html>
```

**Path 2 — `Web Pages/register.jsp`**
```jsp
<%@ page import="java.sql.*,java.security.MessageDigest,java.nio.charset.StandardCharsets"
 contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String hash(String s)throws Exception{
 byte[] a=MessageDigest.getInstance("SHA-256").digest(s.getBytes(StandardCharsets.UTF_8));
 StringBuilder b=new StringBuilder();for(byte x:a)b.append(String.format("%02x",x));return b.toString();
}
%>
<%
String msg="Registration failed.";
try{
 String u=request.getParameter("username"),e=request.getParameter("email"),p=request.getParameter("password");
 if(u==null||u.trim().length()<3||e==null||!e.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$")
    ||p==null||p.length()<8)throw new IllegalArgumentException();
 try(Connection c=DriverManager.getConnection("jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC",
 "root","YOUR_MYSQL_PASSWORD");
 PreparedStatement ps=c.prepareStatement("INSERT INTO users(username,password_hash,email) VALUES(?,?,?)")){
 ps.setString(1,u.trim());ps.setString(2,hash(p));ps.setString(3,e.trim());ps.executeUpdate();
 msg="Registration successful. Please log in.";
 }
}catch(SQLIntegrityConstraintViolationException e){msg="Username already exists.";}
 catch(Exception e){msg="Registration failed. Check your inputs and database settings.";}
%>
<!DOCTYPE html><html><body><h2><%=msg%></h2><a href="index.jsp">Back</a></body></html>
```

**Path 3 — `Web Pages/login.jsp`**
```jsp
<%@ page import="java.sql.*,java.security.MessageDigest,java.nio.charset.StandardCharsets"
 contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String hash(String s)throws Exception{
 byte[] a=MessageDigest.getInstance("SHA-256").digest(s.getBytes(StandardCharsets.UTF_8));
 StringBuilder b=new StringBuilder();for(byte x:a)b.append(String.format("%02x",x));return b.toString();
}
private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
%>
<%
String u=request.getParameter("username"),p=request.getParameter("password");
boolean valid=false;
try(Connection c=DriverManager.getConnection("jdbc:mysql://localhost:3306/ejdb?useSSL=false&serverTimezone=UTC",
 "root","YOUR_MYSQL_PASSWORD");
 PreparedStatement ps=c.prepareStatement("SELECT id FROM users WHERE username=? AND password_hash=?")){
 ps.setString(1,u);ps.setString(2,hash(p==null?"":p));try(ResultSet rs=ps.executeQuery()){valid=rs.next();}
}catch(Exception e){application.log("Login failed",e);}
if(valid){session.setAttribute("username",u);response.sendRedirect("welcome.jsp");return;}
%>
<!DOCTYPE html><html><body><h2>Login failed</h2><a href="index.jsp">Try again</a></body></html>
```

**Path 4 — `Web Pages/welcome.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
%>
<%
String username=(String)session.getAttribute("username");
if(username==null){response.sendRedirect("index.jsp");return;}
%>
<!DOCTYPE html><html><body><h2>Welcome, <%=esc(username)%>!</h2>
<a href="logout.jsp">Logout</a></body></html>
```

**Path 5 — `Web Pages/logout.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<% session.invalidate(); response.sendRedirect("index.jsp"); %>
```

**Execution:** Create the `users` table, add Connector/J, change the database password, register a new user, log in, then test logout.

## Q39. Five-page web application using any two or three Java EE technologies

**Project:** `FivePageStudentPortal`  
**Technologies:** HTML/JSP, Servlet, Session. This simple five-page application stores a student profile in a session.

**Path 1 — `Web Pages/index.html`**
```html
<!DOCTYPE html><html><body><h2>Student Portal</h2>
<a href="register.html">Register student</a></body></html>
```

**Path 2 — `Web Pages/register.html`**
```html
<!DOCTYPE html><html><body><h2>Student Registration</h2>
<form action="saveStudent" method="post">
Roll number: <input name="roll" required><br><br>
Name: <input name="name" required><br><br>
Course: <input name="course" required><br><br>
<button>Save</button></form></body></html>
```

**Path 3 — `Source Packages/demo/SaveStudentServlet.java`**
```java
package demo;
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/saveStudent")
public class SaveStudentServlet extends HttpServlet {
  @Override protected void doPost(HttpServletRequest req, HttpServletResponse resp)
      throws ServletException, IOException {
    req.setCharacterEncoding("UTF-8");
    String roll=req.getParameter("roll"), name=req.getParameter("name"), course=req.getParameter("course");
    if(roll==null||roll.isBlank()||name==null||name.isBlank()||course==null||course.isBlank()){
      resp.sendError(400,"All fields are required."); return;
    }
    HttpSession session=req.getSession();
    session.setAttribute("roll",roll.trim());
    session.setAttribute("studentName",name.trim());
    session.setAttribute("course",course.trim());
    resp.sendRedirect("student.jsp");
  }
}
```

**Path 4 — `Web Pages/student.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
%>
<%
String roll=(String)session.getAttribute("roll");
if(roll==null){response.sendRedirect("register.html");return;}
%>
<!DOCTYPE html><html><body><h2>Student Details</h2>
<p>Roll: <%=esc(roll)%></p><p>Name: <%=esc((String)session.getAttribute("studentName"))%></p>
<p>Course: <%=esc((String)session.getAttribute("course"))%></p>
<a href="edit.html">Edit profile</a> | <a href="logout.jsp">Logout</a></body></html>
```

**Path 5 — `Web Pages/edit.html`**
```html
<!DOCTYPE html><html><body><h2>Update Profile</h2>
<p>For this demonstration, submit a new profile from the registration page.</p>
<a href="register.html">Open registration form</a> | <a href="student.jsp">View saved profile</a></body></html>
```

**Additional logout page — `Web Pages/logout.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<% session.invalidate(); response.sendRedirect("index.html"); %>
```

**Execution:** Create the `demo` package and servlet. In NetBeans, confirm the servlet is mapped to `/saveStudent`. Open `/index.html`, register a student, and view the session-backed details at `student.jsp`. The five primary pages are `index.html`, `register.html`, `student.jsp`, `edit.html`, and the registration result displayed by `student.jsp`; `logout.jsp` is an additional utility page.

## Q40. Servlet bakery application calculating total bill from cake quantity and price

**Project:** `ServletBakeryBill`

**Path 1 — `Web Pages/index.html`**
```html
<!DOCTYPE html><html><body><h2>Bakery Bill</h2>
<form action="bakeryBill" method="post">
Cake name: <input name="cake" required><br><br>
Quantity: <input type="number" name="quantity" min="1" required><br><br>
Price per cake: <input type="number" name="price" min="0" step="0.01" required><br><br>
<button>Calculate Bill</button></form></body></html>
```

**Path 2 — `Source Packages/demo/BakeryBillServlet.java`**
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/bakeryBill")
public class BakeryBillServlet extends HttpServlet {
  @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
    resp.setContentType("text/html;charset=UTF-8");
    try {
      String cake=req.getParameter("cake");
      int quantity=Integer.parseInt(req.getParameter("quantity"));
      BigDecimal price=new BigDecimal(req.getParameter("price"));
      if(cake==null||cake.isBlank()||quantity<1||price.signum()<0)throw new IllegalArgumentException();
      BigDecimal total=price.multiply(BigDecimal.valueOf(quantity));
      resp.getWriter().println("<!DOCTYPE html><html><body><h2>Bakery Bill</h2>");
      resp.getWriter().println("<p>Cake: "+esc(cake)+"</p><p>Quantity: "+quantity+"</p>");
      resp.getWriter().println("<p>Total: ₹"+total+"</p><a href='index.html'>Back</a></body></html>");
    } catch(Exception e){resp.getWriter().println("<p>Invalid input. Return and enter valid quantity and price.</p><a href='index.html'>Back</a>");}
  }
  private String esc(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;");}
}
```

**Execution:** Run the project and open `/index.html`. Enter quantity and price; the servlet displays `quantity × price`.

## Q41. Servlet mobile shop calculating final price after discount

**Project:** `ServletMobileDiscount`

**Path 1 — `Web Pages/index.html`**
```html
<!DOCTYPE html><html><body><h2>Mobile Shop Discount</h2>
<form action="mobileDiscount" method="post">
Mobile model: <input name="model" required><br><br>
Price: <input type="number" name="price" min="0" step="0.01" required><br><br>
Discount (%): <input type="number" name="discount" min="0" max="100" step="0.01" required><br><br>
<button>Calculate Final Price</button></form></body></html>
```

**Path 2 — `Source Packages/demo/MobileDiscountServlet.java`**
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import java.math.RoundingMode;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/mobileDiscount")
public class MobileDiscountServlet extends HttpServlet {
  @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
    resp.setContentType("text/html;charset=UTF-8");
    try {
      String model=req.getParameter("model");
      BigDecimal price=new BigDecimal(req.getParameter("price"));
      BigDecimal discount=new BigDecimal(req.getParameter("discount"));
      if(model==null||model.isBlank()||price.signum()<0||discount.signum()<0||discount.compareTo(BigDecimal.valueOf(100))>0)
        throw new IllegalArgumentException();
      BigDecimal amount=price.multiply(discount).divide(BigDecimal.valueOf(100),2,RoundingMode.HALF_UP);
      BigDecimal finalPrice=price.subtract(amount);
      resp.getWriter().println("<!DOCTYPE html><html><body><h2>Mobile Bill</h2><p>Model: "+esc(model)+"</p>");
      resp.getWriter().println("<p>Original price: ₹"+price+"</p><p>Discount: ₹"+amount+"</p>");
      resp.getWriter().println("<p>Final price: ₹"+finalPrice+"</p><a href='index.html'>Back</a></body></html>");
    } catch(Exception e){resp.getWriter().println("<p>Invalid input. Discount must be between 0 and 100.</p><a href='index.html'>Back</a>");}
  }
  private String esc(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;");}
}
```

**Execution:** Run `/index.html`, enter the price and discount, and submit. Formula: `final price = price − (price × discount / 100)`.

## Q42. Servlet hostel registration storing student details in a session and displaying them on a confirmation page

**Project:** `ServletHostelRegistration`

**Path 1 — `Web Pages/index.html`**
```html
<!DOCTYPE html><html><body><h2>Hostel Registration</h2>
<form action="hostelRegister" method="post">
Student name: <input name="name" required><br><br>
Roll number: <input name="roll" required><br><br>
Course: <input name="course" required><br><br>
Room preference: <select name="room"><option>Single</option><option>Shared</option></select><br><br>
<button>Register</button></form></body></html>
```

**Path 2 — `Source Packages/demo/HostelRegisterServlet.java`**
```java
package demo;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/hostelRegister")
public class HostelRegisterServlet extends HttpServlet {
  @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
    req.setCharacterEncoding("UTF-8");
    String name=req.getParameter("name"), roll=req.getParameter("roll"), course=req.getParameter("course"), room=req.getParameter("room");
    if(name==null||name.isBlank()||roll==null||roll.isBlank()||course==null||course.isBlank()){
      resp.sendError(400,"All required fields must be filled.");return;
    }
    HttpSession s=req.getSession();
    s.setAttribute("hostelName",name.trim());s.setAttribute("hostelRoll",roll.trim());
    s.setAttribute("hostelCourse",course.trim());s.setAttribute("hostelRoom",room);
    resp.sendRedirect("confirmation.jsp");
  }
}
```

**Path 3 — `Web Pages/confirmation.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%!
private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
%>
<%
String name=(String)session.getAttribute("hostelName");
if(name==null){response.sendRedirect("index.html");return;}
%>
<!DOCTYPE html><html><body><h2>Hostel Registration Confirmation</h2>
<p>Name: <%=esc(name)%></p><p>Roll number: <%=esc((String)session.getAttribute("hostelRoll"))%></p>
<p>Course: <%=esc((String)session.getAttribute("hostelCourse"))%></p>
<p>Room preference: <%=esc((String)session.getAttribute("hostelRoom"))%></p>
<a href="index.html">Back</a></body></html>
```

**Execution:** Run `/index.html`, submit the student details, and confirm that the next page reads values from the HTTP session.

## Q43. JSP using EL to display flight booking details received from another page

**Project:** `JSPFlightBookingEL`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Flight Booking</h2>
<form action="flight.jsp" method="post">
Passenger name: <input name="passenger" required><br><br>
Flight number: <input name="flightNo" required><br><br>
Destination: <input name="destination" required><br><br>
Ticket price: <input type="number" name="price" min="0" step="0.01" required><br><br>
<button>Show Booking</button></form></body></html>
```

**Path 2 — `Web Pages/flight.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<!DOCTYPE html><html><body><h2>Flight Booking Details</h2>
<p>Passenger: <c:out value="${param.passenger}"/></p>
<p>Flight: ${param.flightNo}</p><p>Destination: ${param.destination}</p>
<p>Ticket price: ₹${param.price}</p><a href="index.jsp">Back</a>
</body></html>
```

**Execution:** Submit the form. EL reads the request parameters. Use `<c:out>` for output that should be HTML-escaped.

## Q44. JSP using EL to display student examination details

**Project:** `JSPStudentExamEL`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Student Examination Details</h2>
<form action="result.jsp" method="post">
Roll number: <input name="roll" required><br><br>
Name: <input name="name" required><br><br>
Subject: <input name="subject" required><br><br>
Marks: <input type="number" name="marks" min="0" max="100" required><br><br>
<button>Display</button></form></body></html>
```

**Path 2 — `Web Pages/result.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Examination Result</h2>
<p>Roll number: ${param.roll}</p><p>Name: ${param.name}</p>
<p>Subject: ${param.subject}</p><p>Marks: ${param.marks}</p>
<a href="index.jsp">Back</a></body></html>
```

**Execution:** Run `/index.jsp`, enter examination details, and submit. EL expressions such as `${param.roll}` read values from the request.

## Q45. JSP using EL to display employee details received from another page

**Project:** `JSPEmployeeDetailsEL`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Employee Details</h2>
<form action="employee.jsp" method="post">
Employee ID: <input name="id" required><br><br>
Name: <input name="name" required><br><br>
Department: <input name="department" required><br><br>
Designation: <input name="designation" required><br><br>
<button>Display</button></form></body></html>
```

**Path 2 — `Web Pages/employee.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Employee Details</h2>
<p>ID: ${param.id}</p><p>Name: ${param.name}</p>
<p>Department: ${param.department}</p><p>Designation: ${param.designation}</p>
<a href="index.jsp">Back</a></body></html>
```

**Execution:** Run `/index.jsp`, submit the employee details and confirm the result page shows the values via EL.

## Q46. JSP using EL to display product details received from another page

**Project:** `JSPProductDetailsEL`

**Path 1 — `Web Pages/index.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Product Details</h2>
<form action="product.jsp" method="post">
Product ID: <input name="id" required><br><br>
Product name: <input name="name" required><br><br>
Quantity: <input type="number" name="quantity" min="0" required><br><br>
Price: <input type="number" name="price" min="0" step="0.01" required><br><br>
<button>Display</button></form></body></html>
```

**Path 2 — `Web Pages/product.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html><html><body><h2>Product Details</h2>
<p>ID: ${param.id}</p><p>Name: ${param.name}</p>
<p>Quantity: ${param.quantity}</p><p>Price: ₹${param.price}</p>
<a href="index.jsp">Back</a></body></html>
```

**Execution:** Submit product values. The display page uses EL to read the request parameters.

## Q47. JSP using JSTL to display a list of available books with titles, authors and prices

**Project:** `JSPBookListJSTL`

**Path 1 — `Source Packages/demo/Book.java`**
```java
package demo;
import java.math.BigDecimal;
public class Book {
  private final String title, author;
  private final BigDecimal price;
  public Book(String title,String author,BigDecimal price){this.title=title;this.author=author;this.price=price;}
  public String getTitle(){return title;}
  public String getAuthor(){return author;}
  public BigDecimal getPrice(){return price;}
}
```

**Path 2 — `Source Packages/demo/BookListServlet.java`**
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import java.util.Arrays;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/books")
public class BookListServlet extends HttpServlet {
  @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)
      throws ServletException,IOException {
    req.setAttribute("books", Arrays.asList(
      new Book("Effective Java","Joshua Bloch",new BigDecimal("650.00")),
      new Book("Head First Java","Kathy Sierra",new BigDecimal("550.00")),
      new Book("Clean Code","Robert C. Martin",new BigDecimal("700.00"))
    ));
    req.getRequestDispatcher("/books.jsp").forward(req,resp);
  }
}
```

**Path 3 — `Web Pages/books.jsp`**
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<!DOCTYPE html><html><body><h2>Available Books</h2>
<table border="1" cellpadding="8">
<tr><th>Title</th><th>Author</th><th>Price</th></tr>
<c:forEach var="book" items="${books}">
  <tr><td><c:out value="${book.title}"/></td>
      <td><c:out value="${book.author}"/></td>
      <td>₹<c:out value="${book.price}"/></td></tr>
</c:forEach>
</table></body></html>
```

**Execution:** Create package `demo`, add `Book.java` and `BookListServlet.java`, and configure the Jakarta Tags library. Run the project and open `/books`; the servlet forwards the list to `books.jsp`, where JSTL displays each book.

---

## Common troubleshooting

- **HTTP 404:** Check project context path, page name and servlet `@WebServlet` mapping.
- **`jakarta.servlet` import not found:** Confirm the project targets Tomcat 10.1 and has the server libraries configured.
- **JSTL tag prefix error:** Add a Jakarta Tags library compatible with the container and use `uri="jakarta.tags.core"` for Jakarta Tags 3.x.
- **JDBC driver not found:** Add MySQL Connector/J to project Libraries.
- **Access denied / unknown database:** Verify MySQL is running, the database exists, and your username/password are correct.
- **Duplicate employee number or username:** Primary-key and unique constraints reject duplicate rows; choose a new value or update the existing row.
- **JSP compilation error:** Check that each directive appears before HTML, file names match exactly, and Java imports are valid.

## Quick viva questions

1. **What is JSP?** JavaServer Pages, a server-side technology for generating dynamic web content.
2. **What are JSP implicit objects?** Objects provided by the JSP container, such as `request`, `response`, `session`, `application`, `out`, `config`, `pageContext`, and `page`.
3. **What is EL?** Expression Language, used to access scoped attributes and request data using expressions such as `${param.name}`.
4. **What is JSTL?** A standard tag library providing tags for conditionals, iteration, output and other common JSP tasks.
5. **What are JSP action tags?** Standard actions such as `<jsp:include>`, `<jsp:forward>`, `<jsp:useBean>`, `<jsp:setProperty>` and `<jsp:getProperty>`.
6. **What is JDBC?** Java Database Connectivity, the API for connecting Java applications to relational databases.
7. **Why use `PreparedStatement`?** It binds values safely and helps prevent SQL injection.
8. **What is a session?** Server-side state associated with a client across multiple requests.
9. **Why validate on the server?** Client-side validation can be bypassed; server-side validation is necessary for correctness and security.
10. **Why should passwords not be stored as plain text?** A database leak would expose all passwords directly; use a password hashing algorithm designed for credentials in production.
