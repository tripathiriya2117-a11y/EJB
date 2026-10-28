# Enterprise Java Question Bank — Q.2 A
## EJB Applications — Complete Solutions (Questions 1–12)

**Question-bank scope:** Q.2 A, EJB Applications, questions 1–12 from `EJ QB.pdf`.

**Important environment note:** EJB applications do **not** run on plain Apache Tomcat. Use a Jakarta EE application server that supports Enterprise Beans, such as **Payara Server** or **WildFly**, configured in NetBeans. This file uses Jakarta Enterprise Beans (`jakarta.ejb.*`). Select server/library versions that are compatible with each other.

## Common NetBeans setup

1. Install a compatible JDK and Payara Server or WildFly.
2. In NetBeans, register the server under **Tools → Servers → Add Server**.
3. Create a **Jakarta EE Enterprise Application** or **EJB Module** (template names vary by NetBeans version).
4. For web examples, create an EAR containing an EJB module and a WAR module. Place EJB classes in the EJB module and HTML/JSP/Servlet classes in the WAR.
5. Deploy the EAR to the application server and open the WAR's context path in a browser.
6. For database questions, configure a JDBC data source/JNDI name on the server, or use the JDBC URL shown in the example after adding a compatible MySQL Connector/J driver.

> Each practical is independent. Create a separate project/EAR per question or keep its files in separate packages and ensure there are no duplicate class names or servlet mappings.

---

# Q.2 A — EJB Applications

## Q1. Currency converter using EJB to convert Indian Rupees into US Dollars, Euros and British Pounds

**Project:** `CurrencyConverterEJB`  
**Files:** EJB module `src/main/java/demo/CurrencyConverterBean.java`; WAR module `src/main/java/demo/CurrencyServlet.java`; `src/main/webapp/index.html`.

### File 1 — `CurrencyConverterBean.java` (EJB module)
```java
package demo;

import java.math.BigDecimal;
import java.math.RoundingMode;
import jakarta.ejb.Stateless;

@Stateless
public class CurrencyConverterBean {
    // Demonstration rates only; update these before any real-world use.
    private static final BigDecimal USD = new BigDecimal("0.0120");
    private static final BigDecimal EUR = new BigDecimal("0.0110");
    private static final BigDecimal GBP = new BigDecimal("0.0095");

    public BigDecimal convertINR(BigDecimal inr, String currency) {
        if (inr == null || inr.signum() < 0) {
            throw new IllegalArgumentException("Amount must be zero or greater.");
        }
        BigDecimal rate;
        switch (currency == null ? "" : currency.toUpperCase()) {
            case "USD": rate = USD; break;
            case "EUR": rate = EUR; break;
            case "GBP": rate = GBP; break;
            default: throw new IllegalArgumentException("Choose USD, EUR or GBP.");
        }
        return inr.multiply(rate).setScale(2, RoundingMode.HALF_UP);
    }
}
```

### File 2 — `index.html` (WAR module, `src/main/webapp/index.html`)
```html
<!DOCTYPE html>
<html><head><meta charset="UTF-8"><title>INR Currency Converter</title></head>
<body><h2>INR Currency Converter</h2>
<form action="convert" method="post">
  Amount in INR: <input type="number" name="amount" min="0" step="0.01" required><br><br>
  Convert to:
  <select name="currency">
    <option value="USD">US Dollars (USD)</option>
    <option value="EUR">Euros (EUR)</option>
    <option value="GBP">British Pounds (GBP)</option>
  </select><br><br>
  <button type="submit">Convert</button>
</form></body></html>
```

### File 3 — `CurrencyServlet.java` (WAR module)
```java
package demo;

import java.io.IOException;
import java.math.BigDecimal;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/convert")
public class CurrencyServlet extends HttpServlet {
    @EJB
    private CurrencyConverterBean converter;

    @Override protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            BigDecimal amount = new BigDecimal(req.getParameter("amount"));
            String currency = req.getParameter("currency");
            BigDecimal result = converter.convertINR(amount, currency);
            resp.getWriter().println("<!DOCTYPE html><html><body><h2>Conversion Result</h2>");
            resp.getWriter().println("<p>INR: ₹" + amount + "</p><p>" + currency + ": " + result + "</p>");
            resp.getWriter().println("<a href='index.html'>Back</a></body></html>");
        } catch (Exception ex) {
            resp.getWriter().println("<p>Could not convert. Check the amount and currency.</p><a href='index.html'>Back</a>");
        }
    }
}
```

**Execution**
1. Create an EJB module and add `CurrencyConverterBean.java`.
2. Create a WAR module in the same EAR and add `index.html` and `CurrencyServlet.java`.
3. Deploy the EAR to Payara/WildFly. Confirm the WAR module can inject the EJB.
4. Open the deployed WAR URL ending in `/index.html`, enter an INR amount, choose a currency and submit.

**Example:** With the demonstration rate `1 INR = 0.012 USD`, ₹1,000 produces USD 12.00. These are illustrative fixed rates, not live exchange rates.

## Q2. Visitor hit-count application using a Singleton Session Bean

**Project:** `EJBHitCounter`  
**Files:** EJB module `src/main/java/demo/HitCounterBean.java`; WAR module `src/main/java/demo/HitServlet.java`.

### File 1 — `HitCounterBean.java`
```java
package demo;

import jakarta.ejb.Singleton;
import jakarta.ejb.Lock;
import jakarta.ejb.LockType;

@Singleton
public class HitCounterBean {
    private long hits = 0;

    @Lock(LockType.WRITE)
    public long incrementAndGet() {
        return ++hits;
    }

    @Lock(LockType.READ)
    public long getHits() {
        return hits;
    }
}
```

### File 2 — `HitServlet.java`
```java
package demo;

import java.io.IOException;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/hits")
public class HitServlet extends HttpServlet {
    @EJB private HitCounterBean counter;

    @Override protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        long count = counter.incrementAndGet();
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Servlet Hit Counter</h2>");
        resp.getWriter().println("<p>Total visits since server start: " + count + "</p>");
        resp.getWriter().println("<a href='hits'>Refresh (counts as another hit)</a></body></html>");
    }
}
```

**Execution:** Deploy the EAR. Open `/hits` in the WAR context and refresh; the counter increments. The count is held in memory and resets when the application/server restarts.

## Q3. Visitor statistics using a Message-Driven Bean and Stateless Session Bean

**Project:** `EJBVisitorStatistics`  
**Purpose:** The servlet sends visit events to a JMS queue; a Message-Driven Bean consumes the messages and updates a stateless service's persistent counter. This example uses a container-configured JMS queue named `jms/VisitorQueue` and a database table.

**Database table**
```sql
CREATE DATABASE IF NOT EXISTS ejdb;
USE ejdb;
CREATE TABLE IF NOT EXISTS visitor_stats (
  id INT PRIMARY KEY,
  total BIGINT NOT NULL
);
INSERT INTO visitor_stats(id,total) VALUES (1,0)
ON DUPLICATE KEY UPDATE id=id;
```

Configure a JDBC data source on the application server for `jdbc/ejdb` pointing at the database above, and configure JMS queue `jms/VisitorQueue`. Names must match your server configuration.

### File 1 — `VisitorStatsService.java` (EJB module)
```java
package demo;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import javax.sql.DataSource;
import jakarta.annotation.Resource;
import jakarta.ejb.Stateless;

@Stateless
public class VisitorStatsService {
    @Resource(lookup = "jdbc/ejdb")
    private DataSource dataSource;

    public long increment() {
        try (Connection con = dataSource.getConnection()) {
            try (PreparedStatement update = con.prepareStatement(
                    "UPDATE visitor_stats SET total=total+1 WHERE id=1")) {
                if (update.executeUpdate() == 0) {
                    try (PreparedStatement insert = con.prepareStatement(
                            "INSERT INTO visitor_stats(id,total) VALUES(1,1)")) {
                        insert.executeUpdate();
                    }
                }
            }
            try (PreparedStatement query = con.prepareStatement(
                    "SELECT total FROM visitor_stats WHERE id=1");
                 ResultSet rs = query.executeQuery()) {
                if (rs.next()) return rs.getLong(1);
            }
            throw new IllegalStateException("Visitor counter row missing.");
        } catch (Exception ex) {
            throw new IllegalStateException("Unable to update visitor statistics.", ex);
        }
    }
}
```

### File 2 — `VisitorEventMDB.java` (EJB module)
```java
package demo;

import jakarta.ejb.ActivationConfigProperty;
import jakarta.ejb.MessageDriven;
import jakarta.jms.Message;
import jakarta.jms.MessageListener;
import jakarta.jms.JMSException;
import jakarta.ejb.EJB;

@MessageDriven(activationConfig = {
    @ActivationConfigProperty(propertyName = "destinationLookup", propertyValue = "jms/VisitorQueue"),
    @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "jakarta.jms.Queue")
})
public class VisitorEventMDB implements MessageListener {
    @EJB private VisitorStatsService stats;

    @Override public void onMessage(Message message) {
        stats.increment();
    }
}
```

### File 3 — `VisitorServlet.java` (WAR module)
```java
package demo;

import java.io.IOException;
import jakarta.annotation.Resource;
import jakarta.jms.ConnectionFactory;
import jakarta.jms.JMSContext;
import jakarta.jms.Queue;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/visitor")
public class VisitorServlet extends HttpServlet {
    @Resource(lookup = "jms/__defaultConnectionFactory")
    private ConnectionFactory factory;
    @Resource(lookup = "jms/VisitorQueue")
    private Queue queue;

    @Override protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        try (JMSContext context = factory.createContext()) {
            context.createProducer().send(queue, "visit");
        } catch (Exception ex) {
            resp.sendError(500, "Unable to record visit. Check JMS configuration.");
            return;
        }
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Visitor event queued</h2>");
        resp.getWriter().println("<p>A message was sent to the visitor statistics queue.</p></body></html>");
    }
}
```

**Execution**
1. Create the MySQL database/table.
2. Configure server JDBC resource `jdbc/ejdb`, JMS queue `jms/VisitorQueue`, and a JMS connection factory (change `jms/__defaultConnectionFactory` if your server uses a different JNDI name).
3. Deploy the EAR. Visit `/visitor` to enqueue a visit event.
4. Query `SELECT * FROM ejdb.visitor_stats;`. Because JMS is asynchronous, allow a short delay before checking the count.

**Note:** JMS resource setup differs between Payara and WildFly. If your server uses different annotation properties or JNDI names, adjust the configuration to match the server's admin console.

## Q4. Marks Entry EJB application: accept student details and marks, store them in a database, retrieve and display records

**Project:** `EJBMarksEntry`

**Database**
```sql
CREATE TABLE IF NOT EXISTS marks (
  id INT PRIMARY KEY AUTO_INCREMENT,
  roll_no VARCHAR(30) NOT NULL,
  student_name VARCHAR(100) NOT NULL,
  subject VARCHAR(100) NOT NULL,
  marks DECIMAL(5,2) NOT NULL
);
```
Configure `jdbc/ejdb` as a server data source.

### File 1 — `MarksServiceBean.java` (EJB module)
```java
package demo;

import java.sql.*;
import java.util.*;
import javax.sql.DataSource;
import jakarta.annotation.Resource;
import jakarta.ejb.Stateless;

@Stateless
public class MarksServiceBean {
    @Resource(lookup="jdbc/ejdb") private DataSource ds;

    public void add(String roll,String name,String subject,double marks) throws SQLException {
        if(roll==null||roll.isBlank()||name==null||name.isBlank()||subject==null||subject.isBlank()
            ||marks<0||marks>100) throw new IllegalArgumentException("Invalid details.");
        try(Connection c=ds.getConnection();
            PreparedStatement p=c.prepareStatement("INSERT INTO marks(roll_no,student_name,subject,marks) VALUES(?,?,?,?)")){
            p.setString(1,roll.trim());p.setString(2,name.trim());p.setString(3,subject.trim());p.setDouble(4,marks);p.executeUpdate();
        }
    }

    public List<MarksRecord> all() throws SQLException {
        List<MarksRecord> list=new ArrayList<>();
        try(Connection c=ds.getConnection();
            PreparedStatement p=c.prepareStatement("SELECT id,roll_no,student_name,subject,marks FROM marks ORDER BY id DESC");
            ResultSet r=p.executeQuery()){
            while(r.next()) list.add(new MarksRecord(r.getInt("id"),r.getString("roll_no"),
                r.getString("student_name"),r.getString("subject"),r.getDouble("marks")));
        }
        return list;
    }
}
```

Keep `MarksServiceBean.java` containing only the bean class. Put `MarksRecord.java` in its own file in the same `demo` package, and make the class available to the WAR module (for example, by sharing a common library or placing the DTO in a module accessible to both).

### File 2 — `MarksServlet.java` (WAR module)
```java
package demo;

import java.io.IOException;
import java.sql.SQLException;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/marks")
public class MarksServlet extends HttpServlet {
    @EJB private MarksServiceBean service;

    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            resp.getWriter().println("<!DOCTYPE html><html><body><h2>Marks Entry</h2>");
            resp.getWriter().println("<form method='post' action='marks'>");
            resp.getWriter().println("Roll no: <input name='roll' required><br><br>");
            resp.getWriter().println("Student name: <input name='name' required><br><br>");
            resp.getWriter().println("Subject: <input name='subject' required><br><br>");
            resp.getWriter().println("Marks: <input type='number' name='marks' min='0' max='100' step='0.01' required><br><br>");
            resp.getWriter().println("<button>Save</button></form><h3>Saved records</h3><table border='1'><tr><th>Roll</th><th>Name</th><th>Subject</th><th>Marks</th></tr>");
            for(MarksRecord m:service.all()) resp.getWriter().println("<tr><td>"+esc(m.rollNo)+"</td><td>"+esc(m.studentName)+"</td><td>"+esc(m.subject)+"</td><td>"+m.marks+"</td></tr>");
            resp.getWriter().println("</table></body></html>");
        } catch(Exception ex) { resp.sendError(500,"Could not load marks. Check EJB and data-source setup."); }
    }

    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        try {
            service.add(req.getParameter("roll"),req.getParameter("name"),req.getParameter("subject"),
                Double.parseDouble(req.getParameter("marks")));
            resp.sendRedirect("marks");
        } catch(Exception ex) { resp.sendError(400,"Could not save record. Check fields and database setup."); }
    }
    private String esc(String s){return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;");}
}
```

### Required separate file — `MarksRecord.java` (EJB module and visible to WAR)
```java
package demo;
public class MarksRecord {
    public final int id;
    public final String rollNo, studentName, subject;
    public final double marks;
    public MarksRecord(int id,String rollNo,String studentName,String subject,double marks){
        this.id=id;this.rollNo=rollNo;this.studentName=studentName;this.subject=subject;this.marks=marks;
    }
}
```

**Execution:** Create the `marks` table, configure `jdbc/ejdb`, package the EJB and WAR in the same EAR, deploy, then open `/marks`. Add a record and confirm it appears in the table. Ensure the WAR module has access to the EJB module's `MarksRecord` type.

## Q5. Room reservation system using EJB

**Project:** `EJRoomReservation`  
**Description:** A stateless EJB validates and stores room reservations in memory for demonstration. Data resets on restart; a production system should use a database.

### File 1 — `RoomReservationBean.java` (EJB module)
```java
package demo;
import java.util.*;
import jakarta.ejb.Stateless;

@Stateless
public class RoomReservationBean {
    private static final List<String> reservations = Collections.synchronizedList(new ArrayList<>());

    public String reserve(String guest, String roomType, int nights) {
        if(guest==null||guest.isBlank()||roomType==null||roomType.isBlank()||nights<1||nights>365)
            throw new IllegalArgumentException("Enter valid reservation details.");
        String id="RES-"+UUID.randomUUID().toString().substring(0,8).toUpperCase();
        reservations.add(id+" | "+guest.trim()+" | "+roomType+" | "+nights+" night(s)");
        return id;
    }

    public List<String> all() {
        synchronized(reservations) { return new ArrayList<>(reservations); }
    }
}
```

### File 2 — `RoomServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/rooms")
public class RoomServlet extends HttpServlet {
    @EJB private RoomReservationBean bean;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        var out=resp.getWriter();
        out.println("<!DOCTYPE html><html><body><h2>Room Reservation</h2>");
        out.println("<form method='post' action='rooms'>Guest: <input name='guest' required><br><br>");
        out.println("Room type: <select name='type'><option>Single</option><option>Double</option><option>Suite</option></select><br><br>");
        out.println("Nights: <input type='number' name='nights' min='1' max='365' required><br><br><button>Reserve</button></form>");
        out.println("<h3>Reservations in this server session</h3><ul>");
        for(String item:bean.all())out.println("<li>"+esc(item)+"</li>");
        out.println("</ul></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        try{bean.reserve(req.getParameter("guest"),req.getParameter("type"),Integer.parseInt(req.getParameter("nights")));resp.sendRedirect("rooms");}
        catch(Exception e){resp.sendError(400,"Invalid reservation details.");}
    }
    private String esc(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
}
```

**Execution:** Deploy to Payara/WildFly and open `/rooms`. Submit a reservation. Refresh the page to view the list. Note: an EJB stateless bean must not rely on per-instance mutable fields for durable shared storage; for reliable results use a database. For an exam demonstration, the in-memory list is illustrative only and should be replaced by persistence for a robust implementation.

## Q6. Shopping cart using a Stateful Session Bean

**Project:** `EJBShoppingCart`

### File 1 — `ShoppingCartBean.java` (EJB module)
```java
package demo;
import java.math.BigDecimal;
import java.util.*;
import jakarta.ejb.Stateful;

@Stateful
public class ShoppingCartBean {
    private final Map<String,BigDecimal> cart = new LinkedHashMap<>();

    public void add(String item, BigDecimal price) {
        if(item==null||item.isBlank()||price==null||price.signum()<0)throw new IllegalArgumentException();
        cart.merge(item.trim(),price,BigDecimal::add);
    }
    public void remove(String item) { cart.remove(item); }
    public Map<String,BigDecimal> items() { return new LinkedHashMap<>(cart); }
    public BigDecimal total() {
        return cart.values().stream().reduce(BigDecimal.ZERO,BigDecimal::add);
    }
    public void clear() { cart.clear(); }
}
```

### File 2 — `CartServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import java.util.Map;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/cart")
public class CartServlet extends HttpServlet {
    @EJB private ShoppingCartBean cart;

    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        var out=resp.getWriter();
        out.println("<!DOCTYPE html><html><body><h2>Shopping Cart</h2>");
        out.println("<form method='post' action='cart'><input type='hidden' name='action' value='add'>");
        out.println("Item: <input name='item' required> Price: <input type='number' step='0.01' min='0' name='price' required><button>Add</button></form>");
        out.println("<ul>");
        for(Map.Entry<String,BigDecimal> e:cart.items().entrySet())out.println("<li>"+esc(e.getKey())+" — ₹"+e.getValue()
          +" <form style='display:inline' method='post' action='cart'><input type='hidden' name='action' value='remove'>"
          +"<input type='hidden' name='item' value='"+attr(e.getKey())+"'><button>Remove</button></form></li>");
        out.println("</ul><p>Total: ₹"+cart.total()+"</p>");
        out.println("<form method='post' action='cart'><input type='hidden' name='action' value='clear'><button>Clear cart</button></form>");
        out.println("</body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        try{
            String action=req.getParameter("action");
            if("add".equals(action))cart.add(req.getParameter("item"),new BigDecimal(req.getParameter("price")));
            else if("remove".equals(action))cart.remove(req.getParameter("item"));
            else if("clear".equals(action))cart.clear();
            resp.sendRedirect("cart");
        }catch(Exception e){resp.sendError(400,"Invalid cart operation.");}
    }
    private String esc(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
    private String attr(String s){return esc(s).replace("\"","&quot;").replace("'","&#39;");}
}
```

**Execution:** Deploy to an application server and open `/cart`. Add items, remove them and check the total.

> **Important stateful-bean note:** For a web app with multiple concurrent users, use an EJB lookup/producer pattern that associates a distinct stateful bean instance with each HTTP session, and remove the bean on logout. Direct `@EJB` injection into a servlet may share one stateful reference among requests depending on container handling and is not the right design for a multi-user production cart. For a practical demo with one test user it illustrates the stateful bean API, but the production design should keep the bean reference per user session.

## Q7. Electricity bill calculator using EJB

**Project:** `EJBElectricityBill`

### File 1 — `ElectricityBillBean.java` (EJB module)
```java
package demo;
import java.math.BigDecimal;
import java.math.RoundingMode;
import jakarta.ejb.Stateless;

@Stateless
public class ElectricityBillBean {
    public BigDecimal calculate(int units) {
        if(units<0)throw new IllegalArgumentException("Units cannot be negative.");
        BigDecimal bill=BigDecimal.ZERO;
        int remaining=units;
        int first=Math.min(remaining,100); bill=bill.add(BigDecimal.valueOf(first).multiply(new BigDecimal("3.00"))); remaining-=first;
        int second=Math.min(remaining,200); bill=bill.add(BigDecimal.valueOf(second).multiply(new BigDecimal("5.00"))); remaining-=second;
        bill=bill.add(BigDecimal.valueOf(remaining).multiply(new BigDecimal("7.00")));
        return bill.setScale(2,RoundingMode.HALF_UP);
    }
}
```

### File 2 — `ElectricityServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/electricity")
public class ElectricityServlet extends HttpServlet {
    @EJB private ElectricityBillBean bean;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Electricity Bill</h2><form method='post'>");
        resp.getWriter().println("Units consumed: <input type='number' name='units' min='0' required><button>Calculate</button></form></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            int units=Integer.parseInt(req.getParameter("units"));
            resp.getWriter().println("<h2>Bill: ₹"+bean.calculate(units)+"</h2><a href='electricity'>Back</a>");
        } catch(Exception e){resp.getWriter().println("<p>Enter a valid non-negative unit count.</p><a href='electricity'>Back</a>");}
    }
}
```

**Execution:** Deploy the EAR and open `/electricity`. The example slab rates (first 100 at ₹3, next 200 at ₹5, remaining at ₹7) are illustrative because the question bank does not specify tariff rates.

## Q8. Temperature converter using EJB to convert Celsius to Fahrenheit

**Project:** `EJBTemperatureConverter`

### File 1 — `TemperatureBean.java` (EJB module)
```java
package demo;
import jakarta.ejb.Stateless;

@Stateless
public class TemperatureBean {
    public double celsiusToFahrenheit(double celsius) {
        if(!Double.isFinite(celsius))throw new IllegalArgumentException("Temperature must be finite.");
        return (celsius * 9.0 / 5.0) + 32.0;
    }
}
```

### File 2 — `TemperatureServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/temperature")
public class TemperatureServlet extends HttpServlet {
    @EJB private TemperatureBean bean;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Celsius to Fahrenheit</h2><form method='post'>");
        resp.getWriter().println("Celsius: <input type='number' name='celsius' step='any' required><button>Convert</button></form></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            double c=Double.parseDouble(req.getParameter("celsius"));
            resp.getWriter().println("<h2>"+c+" °C = "+bean.celsiusToFahrenheit(c)+" °F</h2><a href='temperature'>Back</a>");
        } catch(Exception e){resp.getWriter().println("<p>Enter a valid temperature.</p><a href='temperature'>Back</a>");}
    }
}
```

**Execution:** Deploy to Payara/WildFly and open `/temperature`. Example: `0 °C` converts to `32 °F`.

## Q9. EJB hotel reservation application accepting guest, room type, check-in date and number of nights

**Project:** `EJBHotelReservation`

### File 1 — `HotelReservationBean.java` (EJB module)
```java
package demo;
import java.time.LocalDate;
import java.util.UUID;
import jakarta.ejb.Stateless;

@Stateless
public class HotelReservationBean {
    public String reserve(String guest,String roomType,LocalDate checkIn,int nights) {
        if(guest==null||guest.isBlank()||roomType==null||roomType.isBlank()||checkIn==null||nights<1||nights>365)
            throw new IllegalArgumentException("Invalid reservation.");
        if(checkIn.isBefore(LocalDate.now()))throw new IllegalArgumentException("Check-in cannot be in the past.");
        return "HOTEL-"+UUID.randomUUID().toString().substring(0,8).toUpperCase();
    }
}
```

### File 2 — `HotelServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import java.time.LocalDate;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/hotel")
public class HotelServlet extends HttpServlet {
    @EJB private HotelReservationBean bean;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Hotel Reservation</h2><form method='post'>");
        resp.getWriter().println("Guest: <input name='guest' required><br><br>Room type: <select name='type'><option>Single</option><option>Double</option><option>Suite</option></select><br><br>");
        resp.getWriter().println("Check-in: <input type='date' name='date' required><br><br>Nights: <input type='number' name='nights' min='1' max='365' required><br><br><button>Reserve</button></form></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            String id=bean.reserve(req.getParameter("guest"),req.getParameter("type"),
                LocalDate.parse(req.getParameter("date")),Integer.parseInt(req.getParameter("nights")));
            resp.getWriter().println("<h2>Reservation request accepted</h2><p>Reference: "+id+"</p>");
        } catch(Exception e){resp.getWriter().println("<p>Invalid reservation details. Check date and number of nights.</p><a href='hotel'>Back</a>");}
    }
}
```

**Execution:** Deploy the EAR and open `/hotel`. Submit guest details, room type, a non-past check-in date and nights. This educational example generates a reference but does not persist or verify room availability.

## Q10. EJB shopping bill calculator accepting product price and quantity

**Project:** `EJBShoppingBill`

### File 1 — `ShoppingBillBean.java` (EJB module)
```java
package demo;
import java.math.BigDecimal;
import jakarta.ejb.Stateless;

@Stateless
public class ShoppingBillBean {
    public BigDecimal total(BigDecimal price,int quantity) {
        if(price==null||price.signum()<0||quantity<0)throw new IllegalArgumentException("Invalid price or quantity.");
        return price.multiply(BigDecimal.valueOf(quantity));
    }
}
```

### File 2 — `ShoppingBillServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/shoppingBill")
public class ShoppingBillServlet extends HttpServlet {
    @EJB private ShoppingBillBean bean;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Shopping Bill</h2><form method='post'>");
        resp.getWriter().println("Product: <input name='product' required><br><br>Price: <input type='number' name='price' min='0' step='0.01' required><br><br>Quantity: <input type='number' name='quantity' min='0' required><br><br><button>Calculate</button></form></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            String product=req.getParameter("product");
            BigDecimal price=new BigDecimal(req.getParameter("price"));
            int quantity=Integer.parseInt(req.getParameter("quantity"));
            if(product==null||product.isBlank())throw new IllegalArgumentException();
            resp.getWriter().println("<h2>Total bill for "+esc(product)+": ₹"+bean.total(price,quantity)+"</h2><a href='shoppingBill'>Back</a>");
        } catch(Exception e){resp.getWriter().println("<p>Enter valid product, price and quantity.</p><a href='shoppingBill'>Back</a>");}
    }
    private String esc(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;").replace("\"","&quot;");}
}
```

**Execution:** Deploy to Payara/WildFly, open `/shoppingBill`, enter a product, unit price and quantity. The EJB calculates `price × quantity`.

## Q11. EJB salary calculator accepting basic salary and allowances and calculating gross salary

**Project:** `EJBSalaryCalculator`

### File 1 — `SalaryBean.java` (EJB module)
```java
package demo;
import java.math.BigDecimal;
import jakarta.ejb.Stateless;

@Stateless
public class SalaryBean {
    public BigDecimal grossSalary(BigDecimal basic,BigDecimal allowances) {
        if(basic==null||allowances==null||basic.signum()<0||allowances.signum()<0)
            throw new IllegalArgumentException("Salary values must be zero or greater.");
        return basic.add(allowances);
    }
}
```

### File 2 — `SalaryServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/salary")
public class SalaryServlet extends HttpServlet {
    @EJB private SalaryBean bean;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("<!DOCTYPE html><html><body><h2>Gross Salary Calculator</h2><form method='post'>");
        resp.getWriter().println("Basic salary: <input type='number' name='basic' min='0' step='0.01' required><br><br>");
        resp.getWriter().println("Allowances: <input type='number' name='allowances' min='0' step='0.01' required><br><br><button>Calculate</button></form></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try {
            BigDecimal basic=new BigDecimal(req.getParameter("basic"));
            BigDecimal allowances=new BigDecimal(req.getParameter("allowances"));
            resp.getWriter().println("<h2>Gross salary: ₹"+bean.grossSalary(basic,allowances)+"</h2><a href='salary'>Back</a>");
        } catch(Exception e){resp.getWriter().println("<p>Enter valid non-negative amounts.</p><a href='salary'>Back</a>");}
    }
}
```

**Execution:** Deploy the EAR and open `/salary`. Gross salary is calculated as `basic salary + allowances`.

## Q12. Stateful Session Bean for restaurant ordering: add/remove food items and calculate total bill

**Project:** `EJBRestaurantOrder`

### File 1 — `RestaurantOrderBean.java` (EJB module)
```java
package demo;
import java.math.BigDecimal;
import java.util.LinkedHashMap;
import java.util.Map;
import jakarta.ejb.Stateful;

@Stateful
public class RestaurantOrderBean {
    private final Map<String,BigDecimal> items = new LinkedHashMap<>();

    public void addItem(String item,BigDecimal price) {
        if(item==null||item.isBlank()||price==null||price.signum()<0)throw new IllegalArgumentException();
        items.put(item.trim(),price);
    }
    public void removeItem(String item) { items.remove(item); }
    public Map<String,BigDecimal> getItems() { return new LinkedHashMap<>(items); }
    public BigDecimal total() {
        return items.values().stream().reduce(BigDecimal.ZERO,BigDecimal::add);
    }
    public void clear() { items.clear(); }
}
```

### File 2 — `RestaurantServlet.java` (WAR module)
```java
package demo;
import java.io.IOException;
import java.math.BigDecimal;
import java.util.Map;
import jakarta.ejb.EJB;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.*;

@WebServlet("/restaurant")
public class RestaurantServlet extends HttpServlet {
    @EJB private RestaurantOrderBean order;
    @Override protected void doGet(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        resp.setContentType("text/html;charset=UTF-8");
        var out=resp.getWriter();
        out.println("<!DOCTYPE html><html><body><h2>Restaurant Order</h2>");
        out.println("<form method='post' action='restaurant'><input type='hidden' name='action' value='add'>");
        out.println("Food item: <input name='item' required> Price: <input type='number' name='price' min='0' step='0.01' required><button>Add item</button></form>");
        out.println("<h3>Current order</h3><ul>");
        for(Map.Entry<String,BigDecimal> e:order.getItems().entrySet()) {
            out.println("<li>"+esc(e.getKey())+" — ₹"+e.getValue()
              +" <form style='display:inline' method='post' action='restaurant'><input type='hidden' name='action' value='remove'>"
              +"<input type='hidden' name='item' value='"+attr(e.getKey())+"'><button>Remove</button></form></li>");
        }
        out.println("</ul><p>Total: ₹"+order.total()+"</p>");
        out.println("<form method='post' action='restaurant'><input type='hidden' name='action' value='clear'><button>Clear order</button></form></body></html>");
    }
    @Override protected void doPost(HttpServletRequest req,HttpServletResponse resp)throws IOException {
        try {
            String action=req.getParameter("action"), item=req.getParameter("item");
            if("add".equals(action))order.addItem(item,new BigDecimal(req.getParameter("price")));
            else if("remove".equals(action))order.removeItem(item);
            else if("clear".equals(action))order.clear();
            resp.sendRedirect("restaurant");
        } catch(Exception e){resp.sendError(400,"Invalid order action.");}
    }
    private String esc(String s){return s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;");}
    private String attr(String s){return esc(s).replace("\"","&quot;").replace("'","&#39;");}
}
```

**Execution:** Deploy the EAR and open `/restaurant`. Add food items, remove them, and observe the total.

> **Stateful bean note:** As with Q6, use one stateful bean reference per HTTP session in a proper multi-user implementation. Direct servlet field injection is a simplified demonstration and may not maintain separate carts/orders per browser session.

---

## General troubleshooting

- **EJB injection is `null`:** The servlet and EJB must be deployed in a supported Java/Jakarta EE application server and packaged/configured correctly. Plain Tomcat cannot provide EJB container services.
- **`jakarta.ejb` not found:** Ensure the EJB module uses a compatible Jakarta EE API and target server.
- **JNDI resource lookup fails:** Check the exact resource name configured in the server console.
- **JMS event not processed:** Verify queue, connection factory, MDB activation configuration, and server logs.
- **Database connection fails:** Check JDBC resource URL, driver, credentials, and table schema.
- **Changes do not appear:** Clean and build the EAR, redeploy it, and check the application server logs.

## Viva questions

1. **What is EJB?** Enterprise JavaBeans is a server-side component model for enterprise business logic.
2. **What is a stateless session bean?** A bean that does not maintain conversational state for a particular client between calls.
3. **What is a stateful session bean?** A bean that maintains conversational state for a client.
4. **What is a singleton session bean?** One shared bean instance per application deployment.
5. **What is a Message-Driven Bean?** An asynchronous EJB component that consumes messages, commonly from JMS.
6. **What is dependency injection?** The container supplies required resources or components, such as an EJB reference, to a class.
7. **Why won't EJB run on plain Tomcat?** Tomcat is a Servlet/JSP container, not a full Jakarta EE EJB container.
8. **What is JNDI?** Java Naming and Directory Interface, used to look up configured resources and components.
9. **Why use a database for persistent visitor statistics?** An in-memory counter resets on restart and is not reliable across multiple server instances.
10. **What is an EAR?** An Enterprise Archive that can package EJB and web modules together for deployment to an application server.
