# Enterprise Java Practical 2 — Servlet Applications with Cookies and Sessions

**Based on:** `EJ PRACTICAL NO.2.pdf`  
**Practical topics:**  
- **2(a)** RequestDispatcher password validation and Welcome Servlet
- **2(b)** Cookie-based visit counter
- **2(c)** Session-based visit counter and session destruction

**Your setup:** NetBeans + JDK + Apache Tomcat 10.1.x.

> **Tomcat 10.1 note:** The scanned handout uses `javax.servlet.*`, which belongs to older servlet versions. For Tomcat 10.1, use `jakarta.servlet.*`. The code below uses Jakarta imports and annotation mappings. Do not also map these same servlet URLs in `web.xml`.

---

## Common NetBeans setup

For each practical, create a separate **Java Web Application** project and select **Apache Tomcat 10.1** as the server.

1. Open NetBeans.
2. Select **File → New Project → Java with Ant → Web Application** (the exact category may differ by NetBeans version).
3. Create the project with the name specified in each practical.
4. Put HTML/JSP pages under **Web Pages**.
5. Put servlet classes under **Source Packages**, inside package `com.practicals`.
6. Use `@WebServlet` to map each servlet. Do not create duplicate mappings for the same URL in `web.xml`.
7. Right-click the project and choose **Run**. Open the page shown in the browser, or right-click the page and choose **Run File**.

If NetBeans creates a default `index.html`, replace it with the `index.jsp` shown in Practical 2(a) if you want the error message to be displayed on the same page.

---

# Practical 2(a): RequestDispatcher password validation

## Aim

Create a servlet that checks the password entered by the user. If the password is `Servlet`, forward the request to `WelcomeServlet`. Otherwise, include the index page again and display an error message.

## Project name

`ServReqDisp`

## File structure

```text
ServReqDisp/
├── Web Pages/
│   └── index.jsp
└── Source Packages/
    └── com.practicals/
        ├── ReqDispLogin.java
        └── WelcomeServlet.java
```

## 1. `Web Pages/index.jsp`

Use JSP instead of plain HTML so the request can display the error message set by the servlet.

```jsp
<%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <title>Request Dispatcher Login</title>
</head>
<body>
    <h2>Login</h2>

    <%
        String error = (String) request.getAttribute("errorMessage");
        if (error != null) {
    %>
        <p style="color:red;"><%= error %></p>
    <%
        }
    %>

    <form action="ReqDispLogin" method="post">
        User Name:
        <input type="text" name="uname" required>
        <br><br>

        Password:
        <input type="password" name="upass" required>
        <br><br>

        <input type="submit" value="Login">
    </form>
</body>
</html>
```

## 2. `Source Packages/com.practicals/ReqDispLogin.java`

```java
package com.practicals;

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

        String username = request.getParameter("uname");
        String password = request.getParameter("upass");

        if ("Servlet".equals(password)) {
            RequestDispatcher dispatcher =
                    request.getRequestDispatcher("/WelcomeServlet");
            dispatcher.forward(request, response);
        } else {
            request.setAttribute("errorMessage", "Sorry, your password is wrong!");

            // Include the login page again, with the error message available to the JSP.
            response.setContentType("text/html;charset=UTF-8");
            RequestDispatcher dispatcher =
                    request.getRequestDispatcher("/index.jsp");
            dispatcher.include(request, response);
        }
    }
}
```

## 3. `Source Packages/com.practicals/WelcomeServlet.java`

```java
package com.practicals;

import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/WelcomeServlet")
public class WelcomeServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws IOException {

        String username = request.getParameter("uname");

        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().println(
                "<!DOCTYPE html><html><head><title>Welcome</title></head><body>"
                + "<h2>Welcome, " + escapeHtml(username) + "!</h2>"
                + "</body></html>"
        );
    }

    private String escapeHtml(String value) {
        if (value == null) return "";
        return value.replace("&", "&amp;")
                    .replace("<", "&lt;")
                    .replace(">", "&gt;")
                    .replace("\"", "&quot;")
                    .replace("'", "&#39;");
    }
}
```

## Execution

1. Create the project `ServReqDisp`.
2. Create `index.jsp` under **Web Pages**.
3. Create both servlet classes under package `com.practicals`.
4. Run the project and enter a username.
5. Enter password **`Servlet`**. The Welcome Servlet should display a welcome message.
6. Enter any other password. The login page should appear again with the error message.

## Expected output

- Correct password: `Welcome, <username>!`
- Incorrect password: `Sorry, your password is wrong!` above the login form.

## How it works

- `getParameter()` reads values submitted by the form.
- `RequestDispatcher.forward()` transfers control to the Welcome Servlet.
- `RequestDispatcher.include()` includes the login JSP in the response.
- `request.setAttribute()` passes the error message to the JSP for the current request.

---

# Practical 2(b): Cookie-based visit counter

## Aim

Create a servlet that uses a browser cookie to remember how many times the user has visited the servlet.

## Project name

`ServCookie`

## File structure

```text
ServCookie/
├── Web Pages/
│   └── index.html
└── Source Packages/
    └── com.practicals/
        └── CookieCounter.java
```

## 1. `Web Pages/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Cookie Visit Counter</title>
</head>
<body>
    <h2>Cookie Visit Counter</h2>
    <form action="CookieCounter" method="get">
        <input type="submit" value="Go to Servlet for Visit Counter">
    </form>
</body>
</html>
```

## 2. `Source Packages/com.practicals/CookieCounter.java`

```java
package com.practicals;

import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/CookieCounter")
public class CookieCounter extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException {

        int count = 0;
        Cookie[] cookies = request.getCookies();

        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if ("noOfVisit".equals(cookie.getName())) {
                    try {
                        count = Integer.parseInt(cookie.getValue());
                    } catch (NumberFormatException ignored) {
                        count = 0;
                    }
                    break;
                }
            }
        }

        count++;

        Cookie visitCookie = new Cookie("noOfVisit", Integer.toString(count));
        visitCookie.setMaxAge(7 * 24 * 60 * 60); // Keep it for seven days
        visitCookie.setHttpOnly(true);
        visitCookie.setPath(request.getContextPath().isEmpty()
                ? "/" : request.getContextPath());

        response.addCookie(visitCookie);
        response.setContentType("text/html;charset=UTF-8");

        if (count == 1) {
            response.getWriter().println("<h2>Welcome! This is your first recorded visit.</h2>");
        } else {
            response.getWriter().println(
                    "<h2>You have visited this servlet " + count + " times.</h2>");
        }

        response.getWriter().println("<p><a href='index.html'>Back</a></p>");
    }
}
```

## Execution

1. Create project `ServCookie`.
2. Create `index.html` and `CookieCounter.java`.
3. Run the project and click **Go to Servlet for Visit Counter** repeatedly.
4. The visit count should increase on each request.
5. To reset it, clear the site's cookies in your browser or wait for the cookie to expire.

## Expected output

- First visit: `Welcome! This is your first recorded visit.`
- Later visits: `You have visited this servlet 2 times.` and so on.

## Important note

The handout uses a static variable for the counter. This version uses the cookie's stored value instead, so the count belongs to the browser rather than being shared by all visitors on the server. Cookies are client-side data and can be cleared or changed by the user.

---

# Practical 2(c): Session creation, visit count, and session destruction

## Aim

Create a servlet that uses `HttpSession` to count visits during the current session, displays the session ID, and provides a link to invalidate the session.

## Project name

`ServSession`

## File structure

```text
ServSession/
├── Web Pages/
│   └── index.html
└── Source Packages/
    └── com.practicals/
        ├── SessionCounter.java
        └── SessionEnd.java
```

## 1. `Web Pages/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Session Demo</title>
</head>
<body>
    <h2>Session Creation and Destruction</h2>

    <form action="SessionCounter" method="get">
        <input type="submit" value="Go to Servlet for Session Demo">
    </form>
</body>
</html>
```

## 2. `Source Packages/com.practicals/SessionCounter.java`

```java
package com.practicals;

import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

@WebServlet("/SessionCounter")
public class SessionCounter extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException {

        HttpSession session = request.getSession(true);

        Integer visitCount = (Integer) session.getAttribute("sessionVisit");

        if (visitCount == null) {
            visitCount = 1;
        } else {
            visitCount++;
        }

        session.setAttribute("sessionVisit", visitCount);

        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().println(
                "<!DOCTYPE html><html><head><title>Visit Counter</title></head><body>"
                + "<h2>You have visited this page "
                + visitCount
                + " time(s) since the current session began.</h2>"
                + "<p>Your Session ID is: <b>"
                + session.getId()
                + "</b></p>"
                + "<p><a href='SessionEnd'>End the Session</a></p>"
                + "<p><a href='SessionCounter'>Refresh visit count</a></p>"
                + "</body></html>"
        );
    }
}
```

## 3. `Source Packages/com.practicals/SessionEnd.java`

```java
package com.practicals;

import java.io.IOException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

@WebServlet("/SessionEnd")
public class SessionEnd extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException {

        response.setContentType("text/html;charset=UTF-8");

        HttpSession session = request.getSession(false);

        if (session == null) {
            response.getWriter().println(
                    "<h2>There is no active session.</h2>");
        } else {
            Integer count = (Integer) session.getAttribute("sessionVisit");
            session.invalidate();

            response.getWriter().println(
                    "<h2>" + (count == null ? 0 : count)
                    + " visit(s) were recorded in the previous session.</h2>");
        }

        response.getWriter().println(
                "<p><a href='index.html'>Click here to start a new session</a></p>");
    }
}
```

## Execution

1. Create project `ServSession`.
2. Create `index.html`, `SessionCounter.java`, and `SessionEnd.java`.
3. Run the project and click **Go to Servlet for Session Demo**.
4. Refresh the servlet link several times. The count should increase.
5. Note the displayed session ID.
6. Click **End the Session**. The servlet invalidates the session.
7. Return to `index.html` and start again. A new session will be created when you visit `SessionCounter`.

## Expected output

The counter page displays:
- The number of visits during the current session.
- The current session ID.
- A link to end the session.

After ending the session, the response shows the previous visit count and a link to begin again.

## How it works

- `request.getSession(true)` retrieves an existing session or creates a new one.
- `session.getAttribute("sessionVisit")` reads the stored counter.
- `session.setAttribute(...)` saves the updated counter.
- `session.getId()` returns the current session identifier.
- `request.getSession(false)` checks for an existing session without creating one.
- `session.invalidate()` ends the session and removes its session attributes.

---

## Troubleshooting

### `javax.servlet` package not found

For Tomcat 10.1, use `jakarta.servlet.*` imports, as in the code above.

### HTTP 404

Check that the form's `action` matches the servlet's `@WebServlet` mapping exactly.

### Duplicate URL mapping error

If a servlet has `@WebServlet("/SessionCounter")`, remove any duplicate `/SessionCounter` mapping from `web.xml`, or choose XML mapping and remove the annotation. Use only one approach per servlet.

### Wrong password shows no error

Make sure the file is `index.jsp`, not a plain `index.html`, because this example reads the `errorMessage` request attribute in JSP. The servlet includes `/index.jsp`.

### Cookie count resets

The count depends on the browser cookie. Clearing cookies, using a different browser/profile, or using a private window resets the stored count.

---

## Quick viva questions

1. **What is a cookie?** A small name/value item stored by the browser and sent with later matching requests.
2. **What is `HttpSession`?** A server-side mechanism for storing user-specific data across multiple requests.
3. **Cookie vs session?** Cookie data is stored by the browser; session attributes are stored on the server.
4. **What is `RequestDispatcher`?** An interface used to forward a request to or include another server-side resource.
5. **Difference between `forward()` and `include()`?** `forward()` transfers control to another resource; `include()` inserts another resource's response into the current response.
6. **What does `session.invalidate()` do?** Ends the session and clears its attributes.
7. **Why is `getSession(false)` used?** It returns an existing session or `null` without creating a new session.
8. **Why should the visit counter not use a single static variable?** A static counter is shared by all users in the servlet instance and does not represent a per-user visit count.

