# Database Failover Strategies with HikariCP & Spring Boot Example Integrating All Optimizations

## 1. What is Connection Pooling?
Connection pooling is a mechanism that maintains a cache (pool) of database connections so they can be reused, reducing the overhead of creating and closing connections frequently.

### Why Use Connection Pooling?
- **Performance Improvement**: Avoids the cost of repeatedly creating and destroying connections.
- **Efficient Resource Management**: Prevents exhaustion of database connections.
- **Better Scalability**: Handles multiple requests efficiently by reusing idle connections.

---

## 2. Basics of JDBC Connection Without Pooling

A basic JDBC connection looks like this:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class JDBCConnectionExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mydb";
        String user = "root";
        String password = "password";

        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            System.out.println("Connected to database!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### Problems Without Connection Pooling
- Every request creates a new connection, leading to latency.
- Connections are not reused, leading to high resource consumption.
- High load can exhaust the database connections.

---

## 3. Database Failover Strategies with HikariCP

Failover is **critical** for high-availability applications to ensure the database remains accessible even if the primary instance goes down.

### How HikariCP Handles Failover
HikariCP does not handle failover **directly**, but it can be configured to:
1. **Use multiple database hosts for automatic failover.**
2. **Retry connections after failures.**
3. **Detect and remove failed connections from the pool.**

---

## 4. Configuring HikariCP for Failover

### Option 1: Multi-Host JDBC URLs (PostgreSQL & MySQL)
For **PostgreSQL**:
```properties
spring.datasource.url=jdbc:postgresql://primary-db,secondary-db/mydb?targetServerType=primary
```
For **MySQL with AWS RDS Cluster**:
```properties
spring.datasource.url=jdbc:mysql:loadbalance://db-node-1,db-node-2,db-node-3/mydb
spring.datasource.hikari.fail-fast=true
```
> If `db-node-1` fails, the connection is automatically routed to another node.

---

### Option 2: Setting Connection Retry Policy
HikariCP does **not** retry failed connections automatically. However, you can configure **reconnect attempts**:
```properties
spring.datasource.hikari.initialization-fail-timeout=2000
spring.datasource.hikari.connection-timeout=3000
```
- **`initialization-fail-timeout=2000`** → If the database is down at startup, HikariCP waits **2 seconds** before failing.
- **`connection-timeout=3000`** → If a query request **waits longer than 3 seconds**, it fails.

---

### Option 3: Custom Health Check for Database Failover
Implement a **custom health check** to detect database failures:
```java
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

public class DatabaseHealthCheck {
    private final DataSource dataSource;

    public DatabaseHealthCheck(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public boolean isDatabaseAlive() {
        try (Connection conn = dataSource.getConnection()) {
            return conn.isValid(2);
        } catch (SQLException e) {
            return false;
        }
    }
}
```
> If `isDatabaseAlive()` returns `false`, switch to a **secondary database**.

---

## 5. Spring Boot Example Integrating All Optimizations

### Spring Boot `application.properties`
```properties
spring.datasource.url=jdbc:mysql://primary-db,secondary-db/mydb?failOverReadOnly=false
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# HikariCP Configurations
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=1000
spring.datasource.hikari.max-lifetime=600000
spring.datasource.hikari.leak-detection-threshold=2000
spring.datasource.hikari.keepalive-time=30000
```

---

### Spring Boot Failover Handling with `@Primary` DataSource
```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://primary-db,secondary-db/mydb");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(50);
        config.setMinimumIdle(10);
        config.setIdleTimeout(30000);
        return new HikariDataSource(config);
    }
}
```
> **Spring Boot will automatically switch databases** if `primary-db` is unreachable.

---

## 6. Summary
| Optimization | Benefit |
|-------------|---------|
| **Failover JDBC URLs** | Ensures automatic database switch |
| **Connection Retry Policy** | Avoids immediate failures |
| **Custom Health Checks** | Detects database failures dynamically |
| **Spring Boot Failover Config** | Enables seamless failover handling |

---
