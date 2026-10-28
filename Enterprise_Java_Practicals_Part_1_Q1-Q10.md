# Enterprise Java Practical Question Bank — Solutions and Execution Guide
## Part 1: Questions 1–10

> **Environment:** NetBeans, JDK, Apache Tomcat 10.1, Jakarta Servlet API. These examples use `jakarta.servlet.*`, not `javax.servlet.*`.
>
> **Project setup:** Create a Java Web application in NetBeans and select Tomcat 10.1. Put HTML files under **Web Pages** and servlet classes under **Source Packages**. The source question bank provides titles for these tasks but not full question wording for every task, so some examples below are practical demonstrations based on the titles.
>
> **Important:** Each servlet below uses `@WebServlet`. Do not also register the same servlet URL in `WEB-INF/web.xml`; duplicate mappings prevent Tomcat from starting the application.

---

## Q1. Simple calculator using a Servlet

### Aim
Accept two numbers and an arithmetic operation in an HTML form, calculate the result in a servlet, and display it.

### File 1: `calculator.html`

```html
<!DOCTYPE html>
<html>
<head><title>Servlet Calculator</title></head>
<body>
  <h2>Simple Calculator</h2>
  <form action="CalculatorServlet" method="post">
    <label>First number:</label>
    <input type="number" name="num1" step="any" required><br><br>
    <label>Second number:</label>
    <input type="number" name="num2" step="any" required><br><br>
    <label>Operation:</label>
    <select name="operation">
      <option value="add">Addition (+)</option>
      <option value="subtract">Subtraction (-)</option>
      <option value="multiply">Multiplication (*)</option>
      <option value="divide">Division (/)</option>
    </select><br><br>
    <button type="submit">Calculate</button>
  </form>
</body>
</html>
```

### File 2: `CalculatorServlet.java`

```java
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/CalculatorServlet")
public class CalculatorServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        double a = Double.parseDouble(request.getParameter("num1"));
        double b = Double.parseDouble(request.getParameter("num2"));
        String operation = request.getParameter("operation");
        String result;

        switch (operation) {
            case "add": result = String.valueOf(a + b); break;
            case "subtract": result = String.valueOf(a - b); break;
            case "multiply": result = String.valueOf(a * b); break;
            case "divide":
                result = b == 0 ? "Cannot divide by zero" : String.valueOf(a / b);
                break;
            default: result = "Invalid operation";
        }

        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().println("<!DOCTYPE html><html><body>");
        response.getWriter().println("<h2>Result: " + result + "</h2>");
        response.getWriter().println("<a href='calculator.html'>Back</a>");
        response.getWriter().println("</body></html>");
    }
}
```

### Steps to execute
1. Create a Java Web project configured for Tomcat 10.1.
2. Add `calculator.html` under **Web Pages**.
3. Create `CalculatorServlet.java` under **Source Packages**. Add the correct package declaration at the top if your project uses one.
4. Check that the form action matches `@WebServlet("/CalculatorServlet")`.
5. Run the project and open `calculator.html`.
6. Enter values, select an operation, and click **Calculate**.

---

## Q2. Calculator using `ServletInputStream` and `ServletOutputStream` for arithmetic and modulus

### Aim
Read URL-encoded form data through `ServletInputStream` and return the arithmetic result through `ServletOutputStream`.

### File 1: `stream-calculator.html`

```html
<!DOCTYPE html>
<html>
<head><title>Stream Calculator</title></head>
<body>
  <h2>Stream Calculator</h2>
  <form action="StreamCalculatorServlet" method="post">
    <input type="number" name="num1" step="any" placeholder="First number" required>
    <input type="number" name="num2" step="any" placeholder="Second number" required>
    <select name="operation">
      <option value="add">Addition</option>
      <option value="subtract">Subtraction</option>
      <option value="multiply">Multiplication</option>
      <option value="divide">Division</option>
      <option value="modulus">Modulus (whole numbers)</option>
    </select>
    <button type="submit">Calculate</button>
  </form>
</body>
</html>
```

### File 2: `StreamCalculatorServlet.java`

```java
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletInputStream;
import jakarta.servlet.ServletOutputStream;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/StreamCalculatorServlet")
public class StreamCalculatorServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        ServletInputStream input = request.getInputStream();
        byte[] bytes = new byte[1024];
        int count;
        while ((count = input.read(bytes)) != -1) {
            buffer.write(bytes, 0, count);
        }

        String body = buffer.toString(StandardCharsets.UTF_8);
        Map<String, String> values = new HashMap<>();
        for (String pair : body.split("&")) {
            String[] parts = pair.split("=", 2);
            if (parts.length == 2) {
                values.put(URLDecoder.decode(parts[0], StandardCharsets.UTF_8),
                           URLDecoder.decode(parts[1], StandardCharsets.UTF_8));
            }
        }

        double a = Double.parseDouble(values.get("num1"));
        double b = Double.parseDouble(values.get("num2"));
        String operation = values.get("operation");
        String result;

        switch (operation) {
            case "add": result = String.valueOf(a + b); break;
            case "subtract": result = String.valueOf(a - b); break;
            case "multiply": result = String.valueOf(a * b); break;
            case "divide": result = b == 0 ? "Cannot divide by zero" : String.valueOf(a / b); break;
            case "modulus":
                if (a % 1 != 0 || b % 1 != 0) {
                    result = "Modulus requires whole numbers";
                } else if (b == 0) {
                    result = "Cannot calculate modulus by zero";
                } else {
                    result = String.valueOf((long) a % (long) b);
                }
                break;
            default: result = "Invalid operation";
        }

        response.setContentType("text/plain;charset=UTF-8");
        try (ServletOutputStream output = response.getOutputStream()) {
            output.write(("Result: " + result).getBytes(StandardCharsets.UTF_8));
        }
    }
}
```

### Steps to execute
1. Add `stream-calculator.html` and `StreamCalculatorServlet.java`.
2. Run the project and open `stream-calculator.html`.
3. Test addition, division, and modulus. Modulus is for whole-number inputs.
4. The servlet reads the request body through `ServletInputStream` and sends a plain-text response through `ServletOutputStream`.

---

## Q3. Servlet login that displays “Hello” or “Login Failed”

### Aim
Accept a username and password, compare them with sample credentials, and display a success or failure message.

### File 1: `login.html`

```html
<!DOCTYPE html>
<html>
<head><title>Servlet Login</title></head>
<body>
  <h2>Login</h2>
  <form action="LoginServlet" method="post">
    <label>Username:</label>
    <input type="text" name="username" required><br><br>
    <label>Password:</label>
    <input type="password" name="password" required><br><br>
    <button type="submit">Login</button>
  </form>
</body>
</html>
```

### File 2: `LoginServlet.java`

```java
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/LoginServlet")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        response.setContentType("text/html;charset=UTF-8");

        if ("admin".equals(username) && "Servlet".equals(password)) {
            response.getWriter().println("<h2>Hello, " + escapeHtml(username) + "!</h2>");
        } else {
            response.getWriter().println("<h2>Login Failed</h2>");
            response.getWriter().println("<a href='login.html'>Try again</a>");
        }
    }

    private String escapeHtml(String text) {
        return text.replace("&", "&amp;").replace("<", "&lt;")
                   .replace(">", "&gt;").replace("\"", "&quot;")
                   .replace("'", "&#x27;");
    }
}
```

### Steps to execute
1. Add `login.html` and `LoginServlet.java`.
2. Run the project and open `login.html`.
3. Test username `admin` with password `Servlet`.
4. Enter incorrect credentials to test the failure branch.

> These are demonstration credentials for a practical, not production authentication.

---

## Q4. Online examination portal login

### Aim
Create a simple online examination portal login screen. A successful login displays a welcome message; incorrect details display an error.

### File 1: `exam-login.html`

```html
<!DOCTYPE html>
<html>
<head><title>Online Examination Portal</title></head>
<body>
  <h2>Online Examination Portal</h2>
  <form action="ExamLoginServlet" method="post">
    <label>Student ID:</label>
    <input name="studentId" required><br><br>
    <label>Password:</label>
    <input type="password" name="password" required><br><br>
    <button type="submit">Login</button>
  </form>
</body>
</html>
```

### File 2: `ExamLoginServlet.java`

```java
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/ExamLoginServlet")
public class ExamLoginServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String studentId = request.getParameter("studentId");
        String password = request.getParameter("password");
        response.setContentType("text/html;charset=UTF-8");

        if ("STU101".equals(studentId) && "Exam@123".equals(password)) {
            response.getWriter().println("<!DOCTYPE html><html><body>");
            response.getWriter().println("<h2>Welcome to the Examination Portal</h2>");
            response.getWriter().println("<p>Student ID: " + escapeHtml(studentId) + "</p>");
            response.getWriter().println("<p>You may proceed to the examination.</p>");
            response.getWriter().println("</body></html>");
        } else {
            response.getWriter().println("<h2>Invalid Student ID or Password</h2>");
            response.getWriter().println("<a href='exam-login.html'>Try again</a>");
        }
    }

    private String escapeHtml(String text) {
        return text.replace("&", "&amp;").replace("<", "&lt;")
                   .replace(">", "&gt;").replace("\"", "&quot;")
                   .replace("'", "&#x27;");
    }
}
```

### Steps to execute
1. Add `exam-login.html` and `ExamLoginServlet.java`.
2. Run the project and open `exam-login.html`.
3. Sample login: student ID `STU101`, password `Exam@123`.
4. Try incorrect details to test the error branch.

> This is a demonstration login only. It does not implement an exam, database-backed accounts, or production authentication.

---

## Q5. JDBC registration Servlet

### Aim
Collect a name, email, and course and insert them into a MySQL table using JDBC.

### Step 1: Create the database and table

```sql
CREATE DATABASE ejdb;
USE ejdb;

CREATE TABLE registrations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL,
    course VARCHAR(100) NOT NULL
);
```

Add MySQL Connector/J to the project. For Maven, add the following inside `<dependencies>` in `pom.xml` (use a current version compatible with your Java version):

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

### File 1: `register.html`

```html
<!DOCTYPE html>
<html>
<head><title>Student Registration</title></head>
<body>
  <h2>Student Registration</h2>
  <form action="RegisterServlet" method="post">
    <label>Name:</label><input name="name" required><br><br>
    <label>Email:</label><input type="email" name="email" required><br><br>
    <label>Course:</label><input name="course" required><br><br>
    <button type="submit">Register</button>
  </form>
</body>
</html>
```

### File 2: `RegisterServlet.java`

```java
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/RegisterServlet")
public class RegisterServlet extends HttpServlet {
    private static final String URL = "jdbc:mysql://localhost:3306/ejdb";
    private static final String DB_USER = "root";

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        String name = request.getParameter("name");
        String email = request.getParameter("email");
        String course = request.getParameter("course");
        response.setContentType("text/html;charset=UTF-8");

        String sql = "INSERT INTO registrations (name, email, course) VALUES (?, ?, ?)";
        // Replace YOUR_MYSQL_PASSWORD with your local MySQL password.
        try (Connection con = DriverManager.getConnection(
                    URL, DB_USER, "YOUR_MYSQL_PASSWORD");
             PreparedStatement ps = con.prepareStatement(sql)) {
            ps.setString(1, name);
            ps.setString(2, email);
            ps.setString(3, course);
            int rows = ps.executeUpdate();
            response.getWriter().println(rows == 1
                    ? "<h2>Registration successful!</h2>"
                    : "<h2>Registration failed.</h2>");
        } catch (SQLException e) {
            log("Registration database error", e);
            response.getWriter().println(
                    "<h2>Could not complete registration. Check the server log.</h2>");
        }
    }
}
```

### Steps to execute
1. Start MySQL and execute the SQL statements.
2. Add MySQL Connector/J.
3. Replace `YOUR_MYSQL_PASSWORD` with your local MySQL password. Do not commit real credentials to GitHub.
4. Add `register.html` and `RegisterServlet.java`.
5. Run the project and submit the form.
6. Verify the row with `SELECT * FROM registrations;`.

---

## Q6. RequestDispatcher password validation

### Aim
Forward the request to `WelcomeServlet` when the password is `Servlet`. Otherwise, redirect back to the input page with an error message.

### File 1: `index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>Password Validation</title>
  <script>
    window.addEventListener("DOMContentLoaded", function () {
      const params = new URLSearchParams(window.location.search);
      if (params.get("error") === "1") {
        document.getElementById("error").textContent =
          "Incorrect password. Try again.";
      }
    });
  </script>
</head>
<body>
  <h2>Enter Password</h2>
  <p id="error" style="color:red"></p>
  <form action="ReqDispLogin" method="post">
    <input type="password" name="password" required>
    <button type="submit">Submit</button>
  </form>
</body>
</html>
```

### File 2: `ReqDispLogin.java`

```java
import java.io.IOException;
import jakarta.servlet.RequestDispatcher;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/ReqDispLogin")
public class ReqDispLogin extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String password = request.getParameter("password");

        if ("Servlet".equals(password)) {
            RequestDispatcher dispatcher =
                    request.getRequestDispatcher("/WelcomeServlet");
            dispatcher.forward(request, response);
        } else {
            response.sendRedirect("index.html?error=1");
        }
    }
}
```

### File 3: `WelcomeServlet.java`

```java
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/WelcomeServlet")
public class WelcomeServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().println(
                "<h2>Welcome! Password validated successfully.</h2>");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
}
```

### Steps to execute
1. Add `index.html` under **Web Pages**.
2. Create `ReqDispLogin.java` and `WelcomeServlet.java` under **Source Packages**.
3. Use the annotations shown above OR equivalent mappings in `web.xml`, never both for the same URL.
4. Run the project and open `index.html`.
5. Enter `Servlet` to reach `WelcomeServlet`.
6. Enter another password to return to `index.html` with an error message.

> `forward()` transfers the same request to a server-side resource. `sendRedirect()` tells the browser to make a new request.

---

## Q7. Cookie visit counter

### Aim
Use a cookie to remember how many times the browser has requested the servlet.

### File: `VisitCounterServlet.java`

```java
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/VisitCounterServlet")
public class VisitCounterServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        int visits = 0;
        Cookie[] cookies = request.getCookies();

        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if ("visitCount".equals(cookie.getName())) {
                    try {
                        visits = Integer.parseInt(cookie.getValue());
                    } catch (NumberFormatException ignored) {
                        visits = 0;
                    }
                    break;
                }
            }
        }

        visits++;
        Cookie counter = new Cookie("visitCount", String.valueOf(visits));
        counter.setMaxAge(60 * 60 * 24 * 30);
        counter.setHttpOnly(true);
        counter.setPath(request.getContextPath().isEmpty()
                ? "/" : request.getContextPath());
        response.addCookie(counter);

        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().println("<h2>You have visited this servlet "
                + visits + " time(s).</h2>");
        response.getWriter().println(
                "<a href='VisitCounterServlet'>Refresh to count another visit</a>");
    }
}
```

### Steps to execute
1. Create `VisitCounterServlet.java`.
2. Run the project and open `/VisitCounterServlet` under the project context.
3. Refresh the page; the count should increase.
4. Clear the browser cookies or use a private window to reset the count.

---

## Q8. Session creation and destruction

### Aim
Create an HTTP session, store a value, and invalidate the session when requested.

### File: `SessionServlet.java`

```java
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

@WebServlet("/SessionServlet")
public class SessionServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String action = request.getParameter("action");
        response.setContentType("text/html;charset=UTF-8");

        if ("destroy".equals(action)) {
            HttpSession oldSession = request.getSession(false);
            if (oldSession != null) {
                oldSession.invalidate();
            }
            response.getWriter().println("<h2>Session destroyed.</h2>");
            response.getWriter().println(
                    "<a href='SessionServlet'>Create a new session</a>");
            return;
        }

        HttpSession session = request.getSession(true);
        session.setAttribute("message", "Hello from the session!");
        session.setMaxInactiveInterval(15 * 60);
        response.getWriter().println("<h2>Session created or retrieved.</h2>");
        response.getWriter().println("<p>Session ID: " + session.getId() + "</p>");
        response.getWriter().println("<p>" + session.getAttribute("message") + "</p>");
        response.getWriter().println(
                "<a href='SessionServlet?action=destroy'>Destroy session</a>");
    }
}
```

### Steps to execute
1. Create `SessionServlet.java`.
2. Run the project and open `/SessionServlet`.
3. Observe the session ID and stored message.
4. Click **Destroy session** to invalidate the session.
5. Click **Create a new session** to start a new session.

### Viva notes
- `request.getSession()` creates a session if none exists.
- `request.getSession(false)` returns an existing session or `null`.
- `session.invalidate()` invalidates the session.
- `setMaxInactiveInterval()` specifies the inactivity timeout in seconds.

---

## Q9. File upload and download

### Aim
Upload a file and download it later. This example stores uploaded files in a directory under the server's temporary directory.

### File 1: `file.html`

```html
<!DOCTYPE html>
<html>
<head><title>File Upload and Download</title></head>
<body>
  <h2>Upload a file</h2>
  <form action="FileServlet" method="post" enctype="multipart/form-data">
    <input type="file" name="file" required>
    <button type="submit">Upload</button>
  </form>

  <h2>Download a file</h2>
  <form action="FileServlet" method="get">
    <input name="filename" placeholder="Uploaded filename" required>
    <button type="submit">Download</button>
  </form>
</body>
</html>
```

### File 2: `FileServlet.java`

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.MultipartConfig;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.Part;

@WebServlet("/FileServlet")
@MultipartConfig(maxFileSize = 10 * 1024 * 1024)
public class FileServlet extends HttpServlet {
    private Path uploadDir;

    @Override
    public void init() throws ServletException {
        uploadDir = Paths.get(System.getProperty("java.io.tmpdir"), "ej-uploads")
                         .toAbsolutePath().normalize();
        try {
            Files.createDirectories(uploadDir);
        } catch (IOException e) {
            throw new ServletException("Could not create upload directory", e);
        }
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        Part part = request.getPart("file");
        String submittedName = part.getSubmittedFileName();
        String safeName = Paths.get(submittedName == null ? "" : submittedName)
                               .getFileName().toString()
                               .replaceAll("[^A-Za-z0-9._-]", "_");

        if (safeName.isBlank()) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Invalid filename");
            return;
        }

        Path destination = uploadDir.resolve(safeName).normalize();
        if (!destination.getParent().equals(uploadDir)) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Invalid filename");
            return;
        }

        try (InputStream in = part.getInputStream()) {
            Files.copy(in, destination, StandardCopyOption.REPLACE_EXISTING);
        }

        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().println("<h2>Uploaded successfully: "
                + safeName.replace("&", "&amp;").replace("<", "&lt;") + "</h2>");
        response.getWriter().println("<a href='file.html'>Back</a>");
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String filename = request.getParameter("filename");
        if (filename == null || filename.isBlank()) {
            response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Enter a filename");
            return;
        }

        String safeName = Paths.get(filename).getFileName().toString();
        Path file = uploadDir.resolve(safeName).normalize();
        if (!file.getParent().equals(uploadDir) || !Files.isRegularFile(file)) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND, "File not found");
            return;
        }

        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition",
                "attachment; filename=\"" + safeName.replace("\"", "") + "\"");
        response.setContentLengthLong(Files.size(file));

        try (InputStream in = Files.newInputStream(file);
             OutputStream out = response.getOutputStream()) {
            in.transferTo(out);
        }
    }
}
```

### Steps to execute
1. Create `file.html` and `FileServlet.java`.
2. Run the project and open `file.html`.
3. Select a small file and upload it.
4. Enter the uploaded filename exactly in the download form.
5. The sample stores files in the `ej-uploads` directory under the Java temporary directory.

> For production, validate file types, authenticate users, enforce quotas, and do not trust client-provided filenames.

---

## Q10. Non-blocking read using the Servlet API

### Aim
Demonstrate non-blocking request-body reading with `ServletInputStream` and `ReadListener`.

### File 1: `nonblocking.html`

```html
<!DOCTYPE html>
<html>
<head><title>Non-blocking Read</title></head>
<body>
  <h2>Submit text to the servlet</h2>
  <form action="NonBlockingReadServlet" method="post">
    <textarea name="message" rows="5" cols="40" required></textarea><br><br>
    <button type="submit">Send</button>
  </form>
</body>
</html>
```

### File 2: `NonBlockingReadServlet.java`

```java
import java.io.IOException;
import java.util.concurrent.atomic.AtomicInteger;
import jakarta.servlet.AsyncContext;
import jakarta.servlet.ReadListener;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletInputStream;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet(value = "/NonBlockingReadServlet", asyncSupported = true)
public class NonBlockingReadServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/plain;charset=UTF-8");
        AsyncContext async = request.startAsync();
        ServletInputStream input = request.getInputStream();
        AtomicInteger byteCount = new AtomicInteger();

        input.setReadListener(new ReadListener() {
            @Override
            public void onDataAvailable() throws IOException {
                byte[] buffer = new byte[1024];
                while (input.isReady() && !input.isFinished()) {
                    int read = input.read(buffer);
                    if (read > 0) {
                        byteCount.addAndGet(read);
                    }
                }
            }

            @Override
            public void onAllDataRead() throws IOException {
                try {
                    HttpServletResponse asyncResponse =
                            (HttpServletResponse) async.getResponse();
                    asyncResponse.getWriter().println(
                            "Non-blocking read completed. Bytes received: " + byteCount.get());
                } finally {
                    async.complete();
                }
            }

            @Override
            public void onError(Throwable throwable) {
                try {
                    async.complete();
                } catch (IllegalStateException ignored) {
                    // The asynchronous request may already have completed.
                }
            }
        });
    }
}
```

### Steps to execute
1. Create `nonblocking.html` and `NonBlockingReadServlet.java`.
2. Ensure the annotation includes `asyncSupported = true`.
3. Run the project and open `nonblocking.html`.
4. Submit some text. The servlet reads available request data asynchronously and displays the number of bytes received.
5. Tomcat 10.1 supports the Servlet non-blocking I/O API.

### Viva notes
- Blocking I/O waits for data to be available.
- Non-blocking I/O uses callbacks such as `ReadListener`.
- `isReady()` indicates whether reading can proceed without blocking.
- `isFinished()` indicates that all request data has been read.
- `AsyncContext.complete()` signals completion of the asynchronous request.

---

## End of Part 1 — Questions 1–10

**Next part:** Questions 11–20. Continue only when requested.
