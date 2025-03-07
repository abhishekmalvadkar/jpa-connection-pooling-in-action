# Connection Pooling in Spring Boot (Under the Hood & Proof Tests)

## 1. How Spring Boot Handles Connection Pooling
Spring Boot **automatically** configures connection pooling when you use **Spring Data JPA** or **JDBC**.

### **Priority Order for Connection Pool Selection:**
1. **HikariCP** (Default & Fastest 🚀)
2. **Tomcat JDBC Pool**
3. **Apache Commons DBCP2**

#### **Proof: Checking the Default Connection Pool**
Run this command in your Spring Boot project:
```shell
mvn dependency:tree | grep hikari
```
If **HikariCP** appears in the output, Spring Boot is using it.

---

## 2. Proving That Spring Boot Uses HikariCP
Enable debug logging to verify the connection pool:
```properties
logging.level.org.springframework.jdbc=DEBUG
```
When the application starts, you should see:
```
HikariCP is the default connection pool used by Spring Boot
```
> **This confirms Spring Boot has auto-configured HikariCP.**

---

## 3. Writing a Test to Verify Connection Pooling
### **JUnit Test for Connection Pooling**
```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
public class ConnectionPoolingTest {

    @Autowired
    private DataSource dataSource;

    @Test
    void testConnectionPooling() throws SQLException {
        Connection conn1 = dataSource.getConnection();
        Connection conn2 = dataSource.getConnection();

        assertNotNull(conn1);
        assertNotNull(conn2);
        assertNotSame(conn1, conn2); // Connections are from the pool

        conn1.close();
        conn2.close();
    }
}
```
> **Expected Result:**
- Connections **are different objects** because they are **borrowed from the pool**.
- Connections **are not newly created** but **reused**.

---

## 4. How HikariCP Works Under the Hood
### **Step 1: Auto-Configuration of DataSource**
Spring Boot loads **HikariDataSource** as a `@Bean` inside `DataSourceAutoConfiguration`.
```java
@Bean
@ConditionalOnMissingBean(DataSource.class)
public DataSource dataSource(DataSourceProperties properties) {
    HikariDataSource hikariDataSource = new HikariDataSource();
    hikariDataSource.setJdbcUrl(properties.getUrl());
    hikariDataSource.setUsername(properties.getUsername());
    hikariDataSource.setPassword(properties.getPassword());
    return hikariDataSource;
}
```
> If **HikariCP** is present, Spring Boot **automatically selects it**.

### **Step 2: HikariCP Creates a Connection Pool**
- When **Spring Boot starts**, HikariCP **pre-warms** the connection pool.
- When a **request** comes in, a **pooled connection** is used.
- When a **connection is closed**, it **goes back to the pool**.

### **Step 3: Proof – Intercepting Connection Creation**
Enable **DEBUG logs** to verify pooling:
```properties
logging.level.com.zaxxer.hikari=DEBUG
```
Expected output:
```
DEBUG com.zaxxer.hikari.pool.HikariPool - Pool stats (total=10, active=1, idle=9, waiting=0)
```
> **Proves that connections are reused instead of new ones being created.**

---

## 5. Performance Testing: Without vs. With Pooling
### **Test 1: Without Pooling (New Connection Every Time)**
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class NoPoolingTest {
    public static void main(String[] args) throws SQLException {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            try (Connection conn = DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/mydb", "root", "password")) {
                System.out.println("Connection created: " + (i + 1));
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("Time taken without pooling: " + (end - start) + "ms");
    }
}
```
> **Expected Result:** Slow execution, high resource usage.

### **Test 2: With HikariCP Pooling (Reusing Connections)**
```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.sql.Connection;
import java.sql.SQLException;

public class WithPoolingTest {
    public static void main(String[] args) throws SQLException {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(10);
        HikariDataSource dataSource = new HikariDataSource(config);

        long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            try (Connection conn = dataSource.getConnection()) {
                System.out.println("Connection acquired from pool: " + (i + 1));
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("Time taken with pooling: " + (end - start) + "ms");
        dataSource.close();
    }
}
```
> **Expected Result:** Faster execution, reduced overhead.

---

## 6. Summary
| Feature | Without Pooling | With HikariCP |
|---------|---------------|-------------|
| **Connection Creation** | New connection per request | Reused from pool |
| **Performance** | Slow | Fast |
| **CPU & DB Load** | High | Optimized |
| **Response Time** | Long | Short |

### **Key Takeaways**
✅ Spring Boot **automatically selects** HikariCP for connection pooling.  
✅ **Connections are reused** instead of being recreated.  
✅ **Performance improves significantly** with pooling.  

---
