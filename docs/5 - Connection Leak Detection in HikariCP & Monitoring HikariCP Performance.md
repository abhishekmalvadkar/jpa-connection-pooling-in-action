# Connection Leak Detection in HikariCP & Monitoring HikariCP Performance

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

## 3. Connection Leak Detection in HikariCP

A **connection leak** occurs when a connection is **acquired but not closed**, leading to resource exhaustion.

### How HikariCP Detects Leaks
HikariCP has a built-in **leak detection threshold**, which logs a warning when a connection is held longer than the configured time.

### Enable Leak Detection
Add the following setting in **HikariCP configuration**:
```java
config.setLeakDetectionThreshold(2000); // 2 seconds
```
OR in **Spring Boot's `application.properties`**:
```properties
spring.datasource.hikari.leak-detection-threshold=2000
```
> If a connection is held **longer than 2 seconds**, HikariCP logs a warning:
```
[HikariCP connection leak detection] Connection leak detected
```

### Example of a Leaking Connection
```java
Connection conn = dataSource.getConnection();
// Forgot to close: conn.close();
```
To **fix the leak**, always **close connections** in a `try-with-resources` block:
```java
try (Connection conn = dataSource.getConnection()) {
    // Use connection
} // Automatically closed
```

---

## 4. Monitoring HikariCP Performance

### Enable Metrics Using Micrometer and Prometheus
HikariCP supports **metrics collection** for monitoring.

#### Spring Boot Setup
Add **Micrometer and Prometheus dependencies**:
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
Enable **HikariCP monitoring** in `application.properties`:
```properties
management.metrics.export.prometheus.enabled=true
management.endpoints.web.exposure.include=metrics
```
Now, **HikariCP metrics** can be accessed at:
```
http://localhost:8080/actuator/metrics/hikaricp.connections.active
```

#### Useful HikariCP Metrics
| Metric | Description |
|--------|------------|
| `hikaricp.connections.active` | Active connections |
| `hikaricp.connections.idle` | Idle connections |
| `hikaricp.connections.pending` | Pending connection requests |
| `hikaricp.connections.usage` | Connection usage ratio |

---

## 5. Tuning HikariCP for Specific Workloads

HikariCP **tuning** depends on your **database and application workload**.

### Key Configurations and Best Practices
| Setting | Description | Recommended Value |
|---------|------------|------------------|
| `maximumPoolSize` | Max connections in the pool | `(CPU cores * 2) + effective DB connections` |
| `minimumIdle` | Minimum idle connections | `50% of maxPoolSize` |
| `idleTimeout` | Time before idle connections are removed | `30000` (30 sec) |
| `connectionTimeout` | Max wait time for a connection | `2000` (2 sec) |
| `maxLifetime` | Max connection lifespan | `1800000` (30 min) |

---

## 6. Optimized HikariCP Configurations for Different Workloads

### Low-Throughput Applications (Light Workload)
For **small applications** with **low traffic**:
```properties
spring.datasource.hikari.maximum-pool-size=5
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=60000
spring.datasource.hikari.connection-timeout=2000
spring.datasource.hikari.max-lifetime=1800000
```

### High-Throughput Applications (Heavy Workload)
For **high-performance web applications**:
```properties
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=1000
spring.datasource.hikari.max-lifetime=600000
```

### Batch Processing Applications
For **long-running queries** (e.g., ETL jobs):
```properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=60000
spring.datasource.hikari.connection-timeout=5000
spring.datasource.hikari.max-lifetime=1200000
```

---

## 7. Summary
| Feature | Benefit |
|---------|---------|
| **Leak Detection** | Identifies unclosed connections |
| **Metrics Monitoring** | Tracks pool usage & performance |
| **Tuning for Workloads** | Optimizes performance & scalability |

---
