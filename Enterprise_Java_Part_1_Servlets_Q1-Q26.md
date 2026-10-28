# Enterprise Java Practicals — Part 1: Servlets

> **Scope:** This file covers the 26 Servlet tasks listed in the supplied question-bank Markdown, in their original order. The supplied file repeats a generic scaffold rather than providing complete solutions, so the implementations below are completed examples based on each listed task title.
>
> **Environment:** NetBeans + JDK + Apache Tomcat 10.1.x. Use `jakarta.servlet.*`, not `javax.servlet.*`. JDBC tasks also need MySQL and MySQL Connector/J.

## 1. Common setup in NetBeans

1. Open NetBeans → **File → New Project**.
2. Choose **Java with Ant → Web Application** (or the equivalent Web Application template available in your NetBeans version).
3. Project name: `ServletPracticals`; server: **Apache Tomcat 10.1**.
4. Under **Source Packages**, create package `com.practicals`.
5. Put Java servlet files in `Source Packages/com.practicals`.
6. Put HTML files in **Web Pages**.
7. For each practical, run the project and open its HTML page. If NetBeans asks for a main page, select the HTML file for the practical you are testing.
8. Keep only one servlet mapping per URL. If using `@WebServlet("/...")`, do not also map that same servlet URL in `web.xml`.

Example project structure:

```text
ServletPracticals/
├── Web Pages/
│   ├── q01.html
│   ├── q02.html
│   └── ...
└── Source Packages/
    └── com.practicals/
        ├── Q01Calculator.java
        └── ...
```

### Tomcat 10.1 compatibility

All imports must use `jakarta.servlet.*`. Tomcat 10.1 implements Jakarta Servlet 6.0. Do not paste `javax.servlet.*` imports from older Tomcat 9 tutorials.

### Database setup for JDBC questions

Install/start MySQL on a computer where it is available, then run:

```sql
CREATE DATABASE ejdb;
USE ejdb;

CREATE TABLE registrations (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(150) NOT NULL,
  password_hash VARCHAR(255) NOT NULL
);

CREATE TABLE books (
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(150) NOT NULL,
  author VARCHAR(100) NOT NULL,
  price DECIMAL(10,2) NOT NULL
);

CREATE TABLE questions (
  id INT PRIMARY KEY AUTO_INCREMENT,
  question_text VARCHAR(500) NOT NULL,
  answer_text VARCHAR(500) NOT NULL
);
```

Add MySQL Connector/J to the project: right-click **Libraries → Add JAR/Folder** and select the Connector/J JAR. Change the connection settings in JDBC code to match your local database. Do not publish your real password to GitHub.

---

## Q1. Simple calculator using a Servlet

**Files:** `Web Pages/q01.html`, `Source Packages/com.practicals/Q01Calculator.java`

### `q01.html`

```html
<!DOCTYPE html>
<html><head><meta charset="UTF-8"><title>Calculator</title></head>
<body>
<h2>Simple Calculator</h2>
<form action="q01" method="post">
  <input type="number" step="any" name="a" placeholder="First number" required>
  <input type="number" step="any" name="b" placeholder="Second number" required>
  <select name="op">
    <option value="+">Addition</option><option value="-">Subtraction</option>
    <option value="*">Multiplication</option><option value="/">Division</option>
  </select>
  <button type="submit">Calculate</button>
</form>
</body></html>
```

### `Q01Calculator.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q01")
public class Q01Calculator extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        try {
            double a = Double.parseDouble(req.getParameter("a"));
            double b = Double.parseDouble(req.getParameter("b"));
            String op = req.getParameter("op");
            double result;
            switch (op) {
                case "+": result = a + b; break;
                case "-": result = a - b; break;
                case "*": result = a * b; break;
                case "/": if (b == 0) throw new ArithmeticException("Cannot divide by zero"); result = a / b; break;
                default: throw new IllegalArgumentException("Unknown operator");
            }
            res.getWriter().printf("<h2>Result: %.4f</h2><a href='q01.html'>Back</a>", result);
        } catch (RuntimeException e) {
            res.getWriter().println("<p>Invalid input: " + e.getMessage() + "</p><a href='q01.html'>Back</a>");
        }
    }
}
```

**Run:** Right-click `q01.html` → Run File. Enter two values and select an operation.

## Q2. Calculator using ServletInputStream and ServletOutputStream (including modulus)

This version reads a URL-encoded POST body from the request input stream and writes a plain-text response using the output stream.

### `q02.html`

```html
<!DOCTYPE html>
<html><head><meta charset="UTF-8"><title>Stream Calculator</title></head><body>
<h2>Stream Calculator</h2>
<form action="q02" method="post">
<input type="number" name="a" step="1" required placeholder="Integer A">
<input type="number" name="b" step="1" required placeholder="Integer B">
<select name="op"><option>+</option><option>-</option><option>*</option><option>/</option><option>%</option></select>
<button>Calculate</button>
</form></body></html>
```

### `Q02StreamCalculator.java`

```java
package com.practicals;
import java.io.*;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q02")
public class Q02StreamCalculator extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        ByteArrayOutputStream bytes = new ByteArrayOutputStream();
        req.getInputStream().transferTo(bytes);
        String body = bytes.toString(StandardCharsets.UTF_8);
        java.util.Map<String,String> p = new java.util.HashMap<>();
        for (String pair : body.split("&")) {
            String[] kv = pair.split("=", 2);
            if (kv.length == 2) p.put(URLDecoder.decode(kv[0], StandardCharsets.UTF_8),
                                      URLDecoder.decode(kv[1], StandardCharsets.UTF_8));
        }
        String out;
        try {
            int a = Integer.parseInt(p.get("a")), b = Integer.parseInt(p.get("b"));
            String op = p.get("op");
            long value;
            switch (op) {
                case "+": value = (long)a + b; break;
                case "-": value = (long)a - b; break;
                case "*": value = (long)a * b; break;
                case "/": if (b == 0) throw new ArithmeticException("division by zero"); value = a / b; break;
                case "%": if (b == 0) throw new ArithmeticException("modulus by zero"); value = a % b; break;
                default: throw new IllegalArgumentException("invalid operator");
            }
            out = "Result = " + value;
        } catch (RuntimeException e) { out = "Error: " + e.getMessage(); }
        res.setContentType("text/plain;charset=UTF-8");
        res.getOutputStream().write(out.getBytes(StandardCharsets.UTF_8));
    }
}
```

**Run:** Run `q02.html`, enter integers, and test `%` as well as the other operators.

## Q3. Servlet login that displays Hello or login failed

### `q03.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Login</title></head><body>
<h2>Login</h2><form action="q03" method="post">
<input name="username" required placeholder="Username">
<input name="password" type="password" required placeholder="Password">
<button>Login</button></form>
<p>Demo credentials: student / java123</p></body></html>
```

### `Q03Login.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q03")
public class Q03Login extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        String u = req.getParameter("username"), p = req.getParameter("password");
        if ("student".equals(u) && "java123".equals(p))
            res.getWriter().println("<h2>Hello, student!</h2>");
        else res.getWriter().println("<h2>Login failed</h2>");
        res.getWriter().println("<a href='q03.html'>Try again</a>");
    }
}
```

**Run:** Open `q03.html`; test both the stated demo credentials and incorrect credentials. This is a classroom demo, not production authentication.

## Q4. Online examination portal login

### `q04.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Exam Login</title></head><body>
<h2>Online Examination Portal</h2><form action="q04" method="post">
<label>Student ID <input name="studentId" required></label>
<label>Password <input type="password" name="password" required></label>
<button>Sign in</button></form></body></html>
```

### `Q04ExamLogin.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q04")
public class Q04ExamLogin extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        String id = req.getParameter("studentId"), pass = req.getParameter("password");
        if ("STU101".equals(id) && "exam123".equals(pass)) {
            HttpSession session = req.getSession();
            session.setAttribute("studentId", id);
            res.getWriter().println("<h2>Login successful</h2><p>Welcome " + id + ". You may start the exam.</p>");
        } else res.getWriter().println("<h2>Invalid student ID or password.</h2>");
        res.getWriter().println("<a href='q04.html'>Back</a>");
    }
}
```

**Run:** Use demo ID `STU101` and password `exam123`. This demonstrates login flow only; it is not a full exam platform.

## Q5. JDBC registration Servlet

**Prerequisites:** MySQL running, `ejdb` database and `registrations` table created as shown above; Connector/J added to project.

### `q05.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Registration</title></head><body>
<h2>Register</h2><form action="q05" method="post">
<input name="name" required placeholder="Full name">
<input name="email" type="email" required placeholder="Email">
<input name="password" type="password" minlength="6" required placeholder="Password">
<button>Register</button></form></body></html>
```

### `Q05Registration.java`

```java
package com.practicals;
import java.io.IOException;
import java.sql.*;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q05")
public class Q05Registration extends HttpServlet {
    private static final String URL = "jdbc:mysql://localhost:3306/ejdb";
    private static final String USER = "root";
    private static final String PASSWORD = "CHANGE_ME";

    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        String name = req.getParameter("name"), email = req.getParameter("email"), pass = req.getParameter("password");
        if (name == null || email == null || pass == null || pass.length() < 6) {
            res.getWriter().println("Please provide valid registration details."); return;
        }
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            // Demo hashing only. For real applications, use BCrypt/Argon2 with a maintained library.
            String hash = java.util.HexFormat.of().formatHex(
                MessageDigest.getInstance("SHA-256").digest(pass.getBytes(StandardCharsets.UTF_8)));
            try (Connection c = DriverManager.getConnection(URL, USER, PASSWORD);
                 PreparedStatement ps = c.prepareStatement(
                    "INSERT INTO registrations(name,email,password_hash) VALUES(?,?,?)")) {
                ps.setString(1, name); ps.setString(2, email); ps.setString(3, hash);
                ps.executeUpdate();
            }
            res.getWriter().println("<h2>Registration successful</h2>");
        } catch (Exception e) {
            getServletContext().log("Registration failed", e);
            res.getWriter().println("<h2>Registration failed. Check database setup and email uniqueness.</h2>");
        }
    }
}
```

**Run:** Replace `CHANGE_ME` locally with your MySQL password, start MySQL, then run `q05.html`. Never commit database credentials. The table above does not enforce unique email; add a unique constraint if required.

## Q6. RequestDispatcher password validation

This uses two servlets and forwards the same request to a success or failure servlet.

### `q06.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Password Check</title></head><body>
<h2>Password validation</h2><form action="q06validate" method="post">
<input name="username" required placeholder="Username">
<input type="password" name="password" required placeholder="Password">
<button>Validate</button></form></body></html>
```

### `Q06Validate.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q06validate")
public class Q06Validate extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
        if ("student".equals(req.getParameter("username")) && "java123".equals(req.getParameter("password"))) {
            req.setAttribute("message", "Password accepted. Welcome!");
            RequestDispatcher rd = req.getRequestDispatcher("/q06success");
            rd.forward(req, res);
        } else {
            req.setAttribute("message", "Invalid username or password.");
            req.getRequestDispatcher("/q06failure").forward(req, res);
        }
    }
}
```

### `Q06Success.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q06success")
public class Q06Success extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>" + req.getAttribute("message") + "</h2>");
    }
}
```

### `Q06Failure.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q06failure")
public class Q06Failure extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>" + req.getAttribute("message") + "</h2><a href='q06.html'>Try again</a>");
    }
}
```

**Run:** Open `q06.html`. Demo login: `student` / `java123`. Do not add these same mappings to `web.xml` because annotations already map them.

## Q7. Cookie visit counter

### `q07.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Cookie Counter</title></head><body>
<h2>Cookie Visit Counter</h2><p>Each click submits a request and increments a browser cookie.</p>
<form action="q07" method="get"><button>Count this visit</button></form></body></html>
```

### `Q07CookieCounter.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q07")
public class Q07CookieCounter extends HttpServlet {
    @Override protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        int count = 0;
        Cookie[] cookies = req.getCookies();
        if (cookies != null) for (Cookie c : cookies) {
            if ("visitCount".equals(c.getName())) {
                try { count = Integer.parseInt(c.getValue()); } catch (NumberFormatException ignored) {}
            }
        }
        count++;
        Cookie cookie = new Cookie("visitCount", Integer.toString(count));
        cookie.setMaxAge(7 * 24 * 60 * 60);
        cookie.setHttpOnly(true);
        cookie.setPath(req.getContextPath().isEmpty() ? "/" : req.getContextPath());
        res.addCookie(cookie);
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>This browser has visited " + count + " time(s).</h2><a href='q07.html'>Back</a>");
    }
}
```

**Run:** Open `q07.html` and click repeatedly. Clear site cookies to reset the count. The counter is browser-specific and can be edited by the client, so it is for demonstration only.

## Q8. Session creation and destruction

### `q08.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Session Demo</title></head><body>
<h2>Session Demo</h2><form action="q08" method="post">
<input name="username" required placeholder="Enter your name">
<button>Create / View Session</button></form>
<form action="q08logout" method="post"><button>Invalidate Session</button></form></body></html>
```

### `Q08Session.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q08")
public class Q08Session extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        HttpSession session = req.getSession();
        String name = req.getParameter("username");
        if (name != null && !name.isBlank()) session.setAttribute("username", name);
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Session created / retrieved</h2><p>Name: " +
            session.getAttribute("username") + "</p><p>Session ID: " + session.getId() + "</p>" +
            "<a href='q08.html'>Back</a>");
    }
}
```

### `Q08Logout.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q08logout")
public class Q08Logout extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        HttpSession session = req.getSession(false);
        if (session != null) session.invalidate();
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Session invalidated.</h2><a href='q08.html'>Back</a>");
    }
}
```

**Run:** Create/view a session, then click Invalidate Session. The logout form uses POST.

## Q9. File upload and download

Tomcat 10.1 supports `@MultipartConfig`. This example saves uploads under the servlet container's temporary directory, not inside the deployed application.

### `q09.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Upload</title></head><body>
<h2>Upload a text file</h2><form action="q09" method="post" enctype="multipart/form-data">
<input type="file" name="file" accept=".txt,text/plain" required><button>Upload</button>
</form></body></html>
```

### `Q09Upload.java`

```java
package com.practicals;
import java.io.*;
import java.nio.file.*;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.*;
import jakarta.servlet.http.*;

@WebServlet("/q09")
@MultipartConfig(maxFileSize = 2 * 1024 * 1024, maxRequestSize = 3 * 1024 * 1024)
public class Q09Upload extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException, ServletException {
        Part part = req.getPart("file");
        String submitted = Paths.get(part.getSubmittedFileName()).getFileName().toString();
        if (!submitted.toLowerCase().endsWith(".txt")) {
            res.sendError(400, "Only .txt files are accepted."); return;
        }
        Path dir = Paths.get(System.getProperty("java.io.tmpdir"), "servlet-practical-uploads");
        Files.createDirectories(dir);
        String safeName = java.util.UUID.randomUUID() + ".txt";
        Path target = dir.resolve(safeName);
        try (InputStream in = part.getInputStream()) { Files.copy(in, target, StandardCopyOption.REPLACE_EXISTING); }
        req.getSession().setAttribute("lastUpload", target.toString());
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Upload successful</h2><p>Saved as " + safeName +
            "</p><a href='q09download'>Download uploaded file</a>");
    }
}
```

### `Q09Download.java`

```java
package com.practicals;
import java.io.*;
import java.nio.file.*;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q09download")
public class Q09Download extends HttpServlet {
    @Override protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        Object value = req.getSession(false) == null ? null : req.getSession(false).getAttribute("lastUpload");
        if (!(value instanceof String)) { res.sendError(404, "Upload a file first."); return; }
        Path file = Paths.get((String)value).normalize();
        Path allowed = Paths.get(System.getProperty("java.io.tmpdir"), "servlet-practical-uploads").normalize();
        if (!file.startsWith(allowed) || !Files.isRegularFile(file)) { res.sendError(404); return; }
        res.setContentType("text/plain");
        res.setHeader("Content-Disposition", "attachment; filename=\"download.txt\"");
        try (InputStream in = Files.newInputStream(file); OutputStream out = res.getOutputStream()) {
            in.transferTo(out);
        }
    }
}
```

**Run:** Upload a small `.txt` file using `q09.html`; use the download link returned after upload.

## Q10. Non-blocking read

Servlet non-blocking I/O requires asynchronous processing and a non-blocking input stream. This minimal example demonstrates the listener lifecycle by reading request bytes asynchronously; it is a teaching example, not an application upload handler.

### `q10.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Non-blocking Read</title></head><body>
<h2>Non-blocking request-body read</h2><form action="q10" method="post">
<textarea name="data" rows="5" cols="40" required>Sample request content</textarea><button>Send</button>
</form></body></html>
```

### `Q10NonBlockingRead.java`

```java
package com.practicals;
import java.io.*;
import java.nio.charset.StandardCharsets;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet(value="/q10", asyncSupported=true)
public class Q10NonBlockingRead extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/plain;charset=UTF-8");
        AsyncContext async = req.startAsync();
        ServletInputStream input = req.getInputStream();
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        input.setReadListener(new ReadListener() {
            @Override public void onDataAvailable() throws IOException {
                byte[] bytes = new byte[1024];
                while (input.isReady() && !input.isFinished()) {
                    int n = input.read(bytes);
                    if (n > 0) buffer.write(bytes, 0, n);
                }
            }
            @Override public void onAllDataRead() throws IOException {
                try {
                    async.getResponse().getWriter().println("Read asynchronously:");
                    async.getResponse().getWriter().println(buffer.toString(StandardCharsets.UTF_8));
                } finally { async.complete(); }
            }
            @Override public void onError(Throwable t) { async.complete(); }
        });
    }
}
```

**Run:** Open `q10.html`, submit the text, and inspect the plain-text response. The Servlet container controls when listener callbacks run.

## Q11. Database question-and-answer application

Use the `questions` table from the database setup above. Insert a sample row:

```sql
INSERT INTO questions(question_text, answer_text)
VALUES ('What does JVM stand for?', 'Java Virtual Machine');
```

### `q11.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Database Q&A</title></head><body>
<h2>Question lookup</h2><form action="q11" method="get">
<input type="number" name="id" min="1" required placeholder="Question ID"><button>Show answer</button>
</form></body></html>
```

### `Q11DatabaseQA.java`

```java
package com.practicals;
import java.io.IOException;
import java.sql.*;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q11")
public class Q11DatabaseQA extends HttpServlet {
    private static final String URL="jdbc:mysql://localhost:3306/ejdb", USER="root", PASSWORD="CHANGE_ME";
    @Override protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        try {
            int id = Integer.parseInt(req.getParameter("id"));
            try (Connection c=DriverManager.getConnection(URL,USER,PASSWORD);
                 PreparedStatement ps=c.prepareStatement("SELECT question_text,answer_text FROM questions WHERE id=?")) {
                ps.setInt(1,id);
                try (ResultSet rs=ps.executeQuery()) {
                    if (rs.next()) res.getWriter().println("<h2>"+escape(rs.getString(1))+"</h2><p>"+escape(rs.getString(2))+"</p>");
                    else res.getWriter().println("<p>No question found for that ID.</p>");
                }
            }
        } catch (Exception e) { getServletContext().log("Q&A lookup failed",e); res.getWriter().println("Unable to retrieve question. Check DB configuration."); }
    }
    private String escape(String s) { return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;"); }
}
```

**Run:** Replace the local DB password, insert sample data, then query its ID from `q11.html`.

## Q12. Five-question multiple-choice quiz

### `q12.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>MCQ Quiz</title></head><body>
<h2>Java MCQ Quiz</h2><form action="q12" method="post">
<p>1. Java source files use which extension?</p><label><input type="radio" name="q1" value="a" required> .class</label>
<label><input type="radio" name="q1" value="b"> .java</label>
<p>2. Which keyword creates an object?</p><label><input type="radio" name="q2" value="a" required> new</label>
<label><input type="radio" name="q2" value="b"> class</label>
<p>3. Which is not a primitive type?</p><label><input type="radio" name="q3" value="a" required> int</label>
<label><input type="radio" name="q3" value="b"> String</label>
<p>4. Which method starts a thread's execution?</p><label><input type="radio" name="q4" value="a" required> start()</label>
<label><input type="radio" name="q4" value="b"> runNow()</label>
<p>5. Which interface represents a sequence of characters?</p><label><input type="radio" name="q5" value="a" required> Runnable</label>
<label><input type="radio" name="q5" value="b"> CharSequence</label>
<button>Submit Quiz</button></form></body></html>
```

### `Q12Quiz.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q12")
public class Q12Quiz extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req, HttpServletResponse res) throws IOException {
        String[] correct={"b","a","b","a","b"}; int score=0;
        res.setContentType("text/html;charset=UTF-8");
        for(int i=1;i<=5;i++) if(correct[i-1].equals(req.getParameter("q"+i))) score++;
        res.getWriter().println("<h2>Your score: "+score+" / 5</h2><a href='q12.html'>Try again</a>");
    }
}
```

**Run:** Answer all five questions and submit. Each correct answer earns one mark.

## Q13. Two-number arithmetic operation form

This is a second basic arithmetic form, using a separate practical URL.

### `q13.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Arithmetic Operation</title></head><body>
<h2>Arithmetic Operation</h2><form action="q13" method="post">
<input name="x" type="number" step="any" required placeholder="Number 1">
<input name="y" type="number" step="any" required placeholder="Number 2">
<select name="operation"><option value="add">Add</option><option value="subtract">Subtract</option>
<option value="multiply">Multiply</option><option value="divide">Divide</option></select><button>Calculate</button>
</form></body></html>
```

### `Q13Arithmetic.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q13")
public class Q13Arithmetic extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        try{
            double x=Double.parseDouble(req.getParameter("x")), y=Double.parseDouble(req.getParameter("y"));
            String op=req.getParameter("operation"); double ans;
            switch(op){case "add":ans=x+y;break;case "subtract":ans=x-y;break;case "multiply":ans=x*y;break;
                case "divide":if(y==0)throw new ArithmeticException("Cannot divide by zero");ans=x/y;break;
                default:throw new IllegalArgumentException("Invalid operation");}
            res.getWriter().println("<h2>Answer: "+ans+"</h2>");
        }catch(RuntimeException e){res.getWriter().println("<p>Input error: "+e.getMessage()+"</p>");}
    }
}
```

**Run:** Open `q13.html` and test each operation.

## Q14. Simple input validation

### `q14.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Input Validation</title></head><body>
<h2>Student Details</h2><form action="q14" method="post">
<input name="name" required minlength="2" placeholder="Name">
<input name="age" type="number" min="1" max="120" required placeholder="Age">
<input name="email" type="email" required placeholder="Email"><button>Validate</button>
</form></body></html>
```

### `Q14Validation.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q14")
public class Q14Validation extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        String name=req.getParameter("name"), email=req.getParameter("email"), ageText=req.getParameter("age");
        try{
            int age=Integer.parseInt(ageText);
            if(name==null||name.trim().length()<2||age<1||age>120||
               email==null||!email.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$")){
                res.getWriter().println("<h2>Validation failed. Check name, age and email.</h2>"); return;
            }
            res.getWriter().println("<h2>Valid details received for "+escape(name)+"</h2><p>Email: "+escape(email)+"</p><p>Age: "+age+"</p>");
        }catch(NumberFormatException e){res.getWriter().println("Age must be a number.");}
    }
    private String escape(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Submit valid and invalid data. HTML validation improves usability, but server-side validation is still necessary.

## Q15. Name form and welcome Servlet

### `q15.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Welcome</title></head><body>
<h2>Enter your name</h2><form action="q15" method="get">
<input name="name" required placeholder="Your name"><button>Submit</button>
</form></body></html>
```

### `Q15Welcome.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q15")
public class Q15Welcome extends HttpServlet {
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse res)throws IOException{
        String name=req.getParameter("name");
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Welcome, "+escape(name==null?"Guest":name)+"!</h2>");
    }
    private String escape(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Open `q15.html` and submit a name.

## Q16. Stationery shop arithmetic

### `q16.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Stationery Shop</title></head><body>
<h2>Stationery Bill</h2><form action="q16" method="post">
<label>Pen price <input type="number" step="0.01" min="0" name="pen" required></label>
<label>Pen quantity <input type="number" min="0" name="penQty" required></label>
<label>Notebook price <input type="number" step="0.01" min="0" name="book" required></label>
<label>Notebook quantity <input type="number" min="0" name="bookQty" required></label><button>Calculate Bill</button>
</form></body></html>
```

### `Q16Stationery.java`

```java
package com.practicals;
import java.io.IOException;
import java.math.BigDecimal;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q16")
public class Q16Stationery extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        try{
            BigDecimal p=new BigDecimal(req.getParameter("pen")), b=new BigDecimal(req.getParameter("book"));
            int pq=Integer.parseInt(req.getParameter("penQty")), bq=Integer.parseInt(req.getParameter("bookQty"));
            if(p.signum()<0||b.signum()<0||pq<0||bq<0)throw new IllegalArgumentException();
            BigDecimal pt=p.multiply(BigDecimal.valueOf(pq)), bt=b.multiply(BigDecimal.valueOf(bq));
            res.getWriter().println("<h2>Stationery Bill</h2><p>Pens: "+pt+"</p><p>Notebooks: "+bt+
                "</p><h3>Total: "+pt.add(bt)+"</h3>");
        }catch(RuntimeException e){res.getWriter().println("Enter valid non-negative prices and quantities.");}
    }
}
```

**Run:** Enter item prices and quantities; verify the subtotal and total.

## Q17. Canteen total cost

### `q17.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Canteen Bill</title></head><body>
<h2>Canteen Order</h2><form action="q17" method="post">
<label>Sandwiches (₹60 each) <input type="number" min="0" name="sandwich" value="0" required></label>
<label>Juices (₹30 each) <input type="number" min="0" name="juice" value="0" required></label>
<label>Meals (₹100 each) <input type="number" min="0" name="meal" value="0" required></label>
<button>Calculate Total</button></form></body></html>
```

### `Q17Canteen.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q17")
public class Q17Canteen extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        try{
            int s=Integer.parseInt(req.getParameter("sandwich")), j=Integer.parseInt(req.getParameter("juice")), m=Integer.parseInt(req.getParameter("meal"));
            if(s<0||j<0||m<0)throw new IllegalArgumentException();
            int total=s*60+j*30+m*100;
            res.getWriter().println("<h2>Canteen Bill</h2><p>Sandwiches: ₹"+(s*60)+"</p><p>Juices: ₹"+(j*30)+
                "</p><p>Meals: ₹"+(m*100)+"</p><h3>Total: ₹"+total+"</h3>");
        }catch(RuntimeException e){res.getWriter().println("Enter valid non-negative quantities.");}
    }
}
```

**Run:** Enter quantities and compare the displayed item costs and total.

## Q18. Admission validation with RequestDispatcher

### `q18.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Admission</title></head><body>
<h2>Admission Eligibility</h2><form action="q18" method="post">
<input name="name" required placeholder="Applicant name">
<input type="number" name="marks" min="0" max="100" required placeholder="Percentage">
<button>Check Eligibility</button></form></body></html>
```

### `Q18Admission.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q18")
public class Q18Admission extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws ServletException,IOException{
        try{
            double marks=Double.parseDouble(req.getParameter("marks"));
            if(marks<0||marks>100)throw new NumberFormatException();
            req.setAttribute("applicant",req.getParameter("name"));
            req.setAttribute("message",marks>=50?"Eligible for admission (demo cutoff: 50%).":"Not eligible (demo cutoff: 50%).");
            req.getRequestDispatcher("/q18result").forward(req,res);
        }catch(NumberFormatException e){res.sendError(400,"Marks must be between 0 and 100.");}
    }
}
```

### `Q18AdmissionResult.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q18result")
public class Q18AdmissionResult extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Admission result for "+escape(String.valueOf(req.getAttribute("applicant")))+"</h2><p>"+
            escape(String.valueOf(req.getAttribute("message")))+"</p>");
    }
    private String escape(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Submit marks. The 50% cutoff is an illustrative rule; use the exact cutoff from your teacher's question if different.

## Q19. Bookshop JDBC storage

Use the `books` table from the setup section. Add a sample book if desired:

```sql
INSERT INTO books(title, author, price) VALUES ('Effective Java', 'Joshua Bloch', 750.00);
```

### `q19.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Bookshop</title></head><body>
<h2>Add Book</h2><form action="q19" method="post">
<input name="title" required placeholder="Book title"><input name="author" required placeholder="Author">
<input type="number" name="price" step="0.01" min="0" required placeholder="Price"><button>Save Book</button>
</form></body></html>
```

### `Q19Bookshop.java`

```java
package com.practicals;
import java.io.IOException;
import java.math.BigDecimal;
import java.sql.*;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q19")
public class Q19Bookshop extends HttpServlet {
    private static final String URL="jdbc:mysql://localhost:3306/ejdb", USER="root", PASSWORD="CHANGE_ME";
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        try{
            String title=req.getParameter("title"), author=req.getParameter("author");
            BigDecimal price=new BigDecimal(req.getParameter("price"));
            if(title==null||title.isBlank()||author==null||author.isBlank()||price.signum()<0)throw new IllegalArgumentException();
            try(Connection c=DriverManager.getConnection(URL,USER,PASSWORD);
                PreparedStatement ps=c.prepareStatement("INSERT INTO books(title,author,price) VALUES(?,?,?)")){
                ps.setString(1,title);ps.setString(2,author);ps.setBigDecimal(3,price);ps.executeUpdate();
            }
            res.getWriter().println("<h2>Book saved successfully.</h2>");
        }catch(IllegalArgumentException e){res.getWriter().println("Enter valid book details.");}
        catch(Exception e){getServletContext().log("Book insert failed",e);res.getWriter().println("Unable to save book. Check MySQL and table setup.");}
    }
}
```

**Run:** Start MySQL, change the local password in the servlet, and submit book details from `q19.html`. Verify with `SELECT * FROM books;`.

## Q20. Parcel tracking using session

### `q20.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Parcel Tracking</title></head><body>
<h2>Track a Parcel</h2><form action="q20" method="post">
<input name="trackingId" required placeholder="Tracking ID"><button>Track</button></form></body></html>
```

### `Q20ParcelTracking.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q20")
public class Q20ParcelTracking extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        String id=req.getParameter("trackingId");
        HttpSession session=req.getSession(); session.setAttribute("trackingId",id);
        String status="IN123".equalsIgnoreCase(id)?"In transit":"Tracking ID recorded (demo status unavailable)";
        session.setAttribute("parcelStatus",status);
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Parcel "+escape(id)+"</h2><p>Status: "+escape(status)+"</p><p>Stored in session "+session.getId()+"</p>");
    }
    private String escape(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Try tracking ID `IN123` to see the demo status. This simulates session storage; it does not connect to a courier service.

## Q21. Online test portal student session

### `q21.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Student Test Portal</title></head><body>
<h2>Student Test Portal</h2><form action="q21" method="post">
<input name="studentId" required placeholder="Student ID"><input name="studentName" required placeholder="Name">
<button>Start Test</button></form></body></html>
```

### `Q21TestSession.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q21")
public class Q21TestSession extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        HttpSession session=req.getSession();
        session.setAttribute("studentId",req.getParameter("studentId"));
        session.setAttribute("studentName",req.getParameter("studentName"));
        session.setAttribute("testStarted",Boolean.TRUE);
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Test session started</h2><p>Student: "+escape(req.getParameter("studentName"))+
            "</p><p>ID: "+escape(req.getParameter("studentId"))+"</p><p>Session ID: "+session.getId()+"</p>");
    }
    private String escape(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Submit a student ID and name. The server-side session stores them for subsequent requests in the same browser session.

## Q22. Shopping website visitor cookie

### `q22.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Shop Visitor</title></head><body>
<h2>Welcome to the Demo Shop</h2><form action="q22" method="get"><button>Visit shop</button></form></body></html>
```

### `Q22ShopCookie.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q22")
public class Q22ShopCookie extends HttpServlet {
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse res)throws IOException{
        boolean returning=false; Cookie[] cookies=req.getCookies();
        if(cookies!=null)for(Cookie c:cookies)if("shopVisited".equals(c.getName()))returning=true;
        Cookie c=new Cookie("shopVisited","yes");c.setMaxAge(30*24*60*60);c.setHttpOnly(true);
        c.setPath(req.getContextPath().isEmpty()?"/":req.getContextPath());res.addCookie(c);
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println(returning?"<h2>Welcome back to the shop!</h2>":"<h2>Welcome to our shop for the first time!</h2>");
    }
}
```

**Run:** Visit `q22.html` twice. Clear cookies to reset the first-visit message.

## Q23. Water bill calculator

The source title does not specify the tariff, so this example uses an explicitly stated **demo tariff**: first 100 units at ₹2/unit, next 100 at ₹3/unit, and units above 200 at ₹5/unit.

### `q23.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Water Bill</title></head><body>
<h2>Water Bill Calculator</h2><form action="q23" method="post">
<label>Units consumed <input type="number" name="units" min="0" required></label><button>Calculate</button>
</form><p>Demo tariff: first 100 units ₹2, next 100 ₹3, above 200 ₹5 per unit.</p></body></html>
```

### `Q23WaterBill.java`

```java
package com.practicals;
import java.io.IOException;
import java.math.BigDecimal;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q23")
public class Q23WaterBill extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        try{
            int u=Integer.parseInt(req.getParameter("units"));if(u<0)throw new NumberFormatException();
            BigDecimal bill;
            if(u<=100)bill=BigDecimal.valueOf(u*2L);
            else if(u<=200)bill=BigDecimal.valueOf(200L+(u-100L)*3L);
            else bill=BigDecimal.valueOf(500L+(u-200L)*5L);
            res.getWriter().println("<h2>Units consumed: "+u+"</h2><h3>Water bill: ₹"+bill+"</h3>");
        }catch(NumberFormatException e){res.getWriter().println("Enter a non-negative whole number of units.");}
    }
}
```

**Run:** Test 80 units (₹160), 150 units (₹350), and 250 units (₹750). Replace the sample tariff if your practical question gives a different one.

## Q24. PDF notice upload/download

This example allows only PDF files and saves them outside the deployed application. It reuses the session to remember the latest uploaded file.

### `q24.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>PDF Notice</title></head><body>
<h2>Upload a PDF notice</h2><form action="q24" method="post" enctype="multipart/form-data">
<input type="file" name="notice" accept="application/pdf,.pdf" required><button>Upload PDF</button>
</form></body></html>
```

### `Q24PdfUpload.java`

```java
package com.practicals;
import java.io.*;
import java.nio.file.*;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.*;
import jakarta.servlet.http.*;

@WebServlet("/q24")
@MultipartConfig(maxFileSize=5*1024*1024,maxRequestSize=6*1024*1024)
public class Q24PdfUpload extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException,ServletException{
        Part part=req.getPart("notice");
        String original=Paths.get(part.getSubmittedFileName()).getFileName().toString();
        if(!original.toLowerCase().endsWith(".pdf")){res.sendError(400,"Only PDF files are allowed.");return;}
        Path dir=Paths.get(System.getProperty("java.io.tmpdir"),"servlet-pdf-notices");Files.createDirectories(dir);
        Path target=dir.resolve(java.util.UUID.randomUUID()+".pdf");
        try(InputStream in=part.getInputStream()){Files.copy(in,target);}
        req.getSession().setAttribute("noticePath",target.toString());
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>PDF uploaded</h2><a href='q24download'>Download PDF notice</a>");
    }
}
```

### `Q24PdfDownload.java`

```java
package com.practicals;
import java.io.*;
import java.nio.file.*;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q24download")
public class Q24PdfDownload extends HttpServlet {
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse res)throws IOException{
        HttpSession s=req.getSession(false);Object o=s==null?null:s.getAttribute("noticePath");
        if(!(o instanceof String)){res.sendError(404,"Upload a notice first.");return;}
        Path p=Paths.get((String)o).normalize(), dir=Paths.get(System.getProperty("java.io.tmpdir"),"servlet-pdf-notices").normalize();
        if(!p.startsWith(dir)||!Files.isRegularFile(p)){res.sendError(404);return;}
        res.setContentType("application/pdf");res.setHeader("Content-Disposition","attachment; filename=\"notice.pdf\"");
        try(InputStream in=Files.newInputStream(p);OutputStream out=res.getOutputStream()){in.transferTo(out);}
    }
}
```

**Run:** Upload a PDF using `q24.html`, then click the download link. This example checks the extension and size but is not a complete production malware/content-validation system.

## Q25. Library book details on another page

### `q25.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Library Book</title></head><body>
<h2>Enter Book Details</h2><form action="q25" method="post">
<input name="title" required placeholder="Book title"><input name="author" required placeholder="Author">
<input name="isbn" required placeholder="ISBN"><button>Show Details</button>
</form></body></html>
```

### `Q25BookDetails.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q25")
public class Q25BookDetails extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws ServletException,IOException{
        req.setAttribute("title",req.getParameter("title"));
        req.setAttribute("author",req.getParameter("author"));
        req.setAttribute("isbn",req.getParameter("isbn"));
        req.getRequestDispatcher("/q25result").forward(req,res);
    }
}
```

### `Q25BookResult.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q25result")
public class Q25BookResult extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>Library Book Details</h2><p>Title: "+esc(req.getAttribute("title"))+
            "</p><p>Author: "+esc(req.getAttribute("author"))+"</p><p>ISBN: "+esc(req.getAttribute("isbn"))+"</p>");
    }
    private String esc(Object o){String s=String.valueOf(o);return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Enter book details. `RequestDispatcher.forward()` transfers the same request to the result servlet; request attributes are available there.

## Q26. Event registration with RequestDispatcher

### `q26.html`

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Event Registration</title></head><body>
<h2>Event Registration</h2><form action="q26" method="post">
<input name="name" required placeholder="Full name"><input type="email" name="email" required placeholder="Email">
<select name="event"><option value="Java Workshop">Java Workshop</option><option value="Web Development">Web Development</option>
<option value="Cybersecurity Talk">Cybersecurity Talk</option></select><button>Register</button>
</form></body></html>
```

### `Q26EventRegistration.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q26")
public class Q26EventRegistration extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws ServletException,IOException{
        String name=req.getParameter("name"), email=req.getParameter("email"), event=req.getParameter("event");
        if(name==null||name.isBlank()||email==null||!email.matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$")){
            req.setAttribute("message","Registration failed. Enter a name and valid email address.");
        }else{
            req.setAttribute("message","Registration successful!");
            req.setAttribute("name",name);req.setAttribute("email",email);req.setAttribute("event",event);
        }
        req.getRequestDispatcher("/q26result").forward(req,res);
    }
}
```

### `Q26EventResult.java`

```java
package com.practicals;
import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/q26result")
public class Q26EventResult extends HttpServlet {
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse res)throws IOException{
        res.setContentType("text/html;charset=UTF-8");
        res.getWriter().println("<h2>"+esc(req.getAttribute("message"))+"</h2>");
        if(req.getAttribute("event")!=null)res.getWriter().println("<p>Name: "+esc(req.getAttribute("name"))+
            "</p><p>Email: "+esc(req.getAttribute("email"))+"</p><p>Event: "+esc(req.getAttribute("event"))+"</p>");
    }
    private String esc(Object o){String s=String.valueOf(o);return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;").replace("'","&#39;");}
}
```

**Run:** Open `q26.html`, submit valid details, and confirm the result page displays the registration summary.

---

## Troubleshooting

### Error: duplicate servlet URL mapping

Example: `The servlets named [ReqDispLogin] and [com.servcalculator.ReqDispLogin] are both mapped to [/ReqDispLogin]`.

- Search the Java class for `@WebServlet("/ReqDispLogin")`.
- Search `web.xml` for the same URL mapping.
- Keep one mapping method only: annotation **or** `web.xml`, not both.
- If using XML, ensure the servlet class name is fully qualified, e.g. `com.servcalculator.ReqDispLogin`.

### Error: `package javax.servlet does not exist`

You are using old Servlet imports. For Tomcat 10.1, change `javax.servlet...` to `jakarta.servlet...`.

### HTTP 404

- Confirm the project deployed successfully.
- Confirm the HTML form action matches the servlet annotation, e.g. `action="q01"` with `@WebServlet("/q01")`.
- Check the project context path in the browser URL.

### HTTP 500 / JDBC failure

- Read the NetBeans Output and Tomcat logs.
- Confirm MySQL is running and the database/table exists.
- Confirm Connector/J is added to Libraries.
- Confirm the JDBC URL, username, and password.
- Keep real passwords out of source control.

## Quick viva questions

1. **What is a Servlet?** A server-side Java component that handles requests and produces responses.
2. **What is `doGet()` vs `doPost()`?** `doGet()` handles GET requests; `doPost()` handles POST requests.
3. **What does `@WebServlet` do?** Maps a servlet to a URL pattern.
4. **What is `RequestDispatcher.forward()`?** Forwards the same request and response to another server-side resource.
5. **Cookie vs session?** A cookie is stored by the browser; session data is stored server-side and associated with a session identifier.
6. **What is `PreparedStatement`?** A parameterized SQL statement that helps avoid SQL injection.
7. **Why use `try-with-resources` in JDBC?** It automatically closes connections, statements, and result sets.
8. **What is `@MultipartConfig`?** It enables servlet processing of multipart form data such as file uploads.
9. **What is asynchronous/non-blocking I/O?** It allows a servlet to process I/O without blocking a thread while waiting for data.
10. **Why use `jakarta.servlet` here?** Tomcat 10+ uses Jakarta EE package names; older Tomcat examples often use `javax.servlet`.

## Before submitting

- Run each practical separately in NetBeans.
- Make sure each servlet class name matches its filename.
- Confirm package declarations match the package folder.
- Do not duplicate URL mappings between annotations and `web.xml`.
- For JDBC tasks, test the SQL setup and use your own local credentials.
- Replace any demo rules (login credentials, admission cutoff, water tariff) with values explicitly provided by your practical question, if it specifies them.
