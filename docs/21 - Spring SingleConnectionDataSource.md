## Spring SingleConnectionDataSource

### 🔹 What is `SingleConnectionDataSource`?
`SingleConnectionDataSource` is a Spring implementation of `javax.sql.DataSource` that **returns the same connection every time** instead of creating a new one.

🚨 **Not recommended for production** because:
1. It does **not** support connection pooling.
2. It reuses a **single connection**, which is not scalable.
3. If the connection closes unexpectedly, all future database queries will fail.

---

### 🔹 When to Use `SingleConnectionDataSource`?
✅ **Unit Testing** → When you need a simple in-memory database connection for lightweight tests.  
✅ **Simple Applications** → Where only one query runs at a time.  
❌ **Not for production** → Use `HikariCP` or another connection pool instead.  

---

### 🔹 Example Usage
#### **1️⃣ Java Configuration**
```java
import org.springframework.jdbc.datasource.SingleConnectionDataSource;
import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        SingleConnectionDataSource dataSource = new SingleConnectionDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setSuppressClose(true); // Keeps connection open
        return dataSource;
    }
}
```
- `setSuppressClose(true)`: Prevents closing the connection, allowing it to be reused.

---

### 🔹 Alternative for Production
Use **HikariCP** for a real-world application:
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
✅ **Better Performance**  
✅ **Scales with concurrent requests**  
✅ **More reliable**  

