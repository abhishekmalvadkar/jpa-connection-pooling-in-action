## DriverManagerDataSource in Spring

### 🔹 What is `DriverManagerDataSource`?
`DriverManagerDataSource` is a simple implementation of `javax.sql.DataSource` provided by the Spring Framework. It is mainly used for testing or small applications but **not recommended for production** because it does not provide connection pooling.

### 🔹 How It Works
- It creates a **new database connection** every time a request is made, which can be inefficient.
- It directly uses JDBC's `DriverManager` to obtain connections.
- Unlike **HikariCP**, **Apache DBCP**, or **C3P0**, it does **not** maintain a connection pool.

---

### 🔹 Example Usage
#### **1️⃣ Java Configuration**
```java
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {
    
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        return dataSource;
    }
}
```
- This creates a new **database connection** each time a query is executed.

---

### 🔹 Why `DriverManagerDataSource` Is Not Recommended for Production
1. ❌ **No Connection Pooling** → Each query creates a new connection, leading to **performance issues**.
2. ❌ **Increased Database Load** → More overhead on the database due to frequent connection creation.
3. ❌ **Not Scalable** → Can't handle high concurrent requests efficiently.

---

### 🔹 What to Use Instead?
For **real-world applications**, use **a connection pool** like:
- ✅ **HikariCP** (default in Spring Boot)
- ✅ **Apache Commons DBCP2**
- ✅ **C3P0**

#### Example of using **HikariCP**:
```java
import com.zaxxer.hikari.HikariDataSource;
import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setMaximumPoolSize(10); // Connection Pooling
        return dataSource;
    }
}
```
This approach:
- **Improves performance** by reusing connections.
- **Optimizes database load** with connection pooling.
- **Scales well** for high-traffic applications.

---
