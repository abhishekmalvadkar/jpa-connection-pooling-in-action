# HikariCP Performance Benchmarks (vs. DBCP)

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

## 3. HikariCP Performance Benchmarks (vs. DBCP)
HikariCP is known for its **high performance and low latency**. Let's benchmark it against **Apache DBCP** and **plain JDBC**.

### 1. Setting Up the Benchmark
We will compare:
- **Plain JDBC** (No pooling)
- **Apache Commons DBCP**
- **HikariCP**

### 2. HikariCP Setup
#### Maven Dependency
```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

#### HikariCP Configuration
```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.sql.Connection;
import java.sql.SQLException;

public class HikariCPBenchmark {
    private static final HikariDataSource dataSource;

    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(2);
        config.setIdleTimeout(30000);
        config.setConnectionTimeout(2000);
        dataSource = new HikariDataSource(config);
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    public static void main(String[] args) throws SQLException {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            try (Connection conn = getConnection()) {
                System.out.println("Connection acquired using HikariCP: " + (i + 1));
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("Time taken with HikariCP: " + (end - start) + "ms");
        dataSource.close();
    }
}
```

---

### 3. Benchmarking Apache DBCP
```java
import org.apache.commons.dbcp2.BasicDataSource;
import java.sql.Connection;
import java.sql.SQLException;

public class DBCPBenchmark {
    private static final BasicDataSource dataSource = new BasicDataSource();

    static {
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setMinIdle(2);
        dataSource.setMaxIdle(5);
        dataSource.setMaxTotal(10);
        dataSource.setMaxWaitMillis(2000);
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    public static void main(String[] args) throws SQLException {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            try (Connection conn = getConnection()) {
                System.out.println("Connection acquired using DBCP: " + (i + 1));
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("Time taken with DBCP: " + (end - start) + "ms");
        dataSource.close();
    }
}
```

---

### 4. Benchmarking Plain JDBC
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class PlainJDBCBenchmark {
    private static final String URL = "jdbc:mysql://localhost:3306/mydb";
    private static final String USER = "root";
    private static final String PASSWORD = "password";

    public static void main(String[] args) throws SQLException {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD)) {
                System.out.println("Connection established without pooling: " + (i + 1));
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("Time taken without pooling: " + (end - start) + "ms");
    }
}
```

---

### 5. Expected Benchmark Results
| Connection Type | Avg. Time (10 Connections) |
|---------------|------------------|
| Plain JDBC (No Pooling) | ~350ms |
| Apache Commons DBCP | ~120ms |
| HikariCP | ~50ms |

**Why is HikariCP faster?**
- Uses **fewer locks** and **lower overhead**.
- **Connection validation** is optimized.
- **Efficient resource management** prevents unnecessary connection creation.

---

## 6. Summary of Benefits
| Feature | Without Pooling | With Pooling |
|---------|---------------|-------------|
| Connection Creation | New connection for each request | Reused from pool |
| Performance | Slower | Faster |
| Resource Usage | High | Optimized |
| Scalability | Poor | High |
| Fault Tolerance | Low | Improved |

---

## 7. Next Steps
- Try testing with a higher number of requests (e.g., 1000).
- Experiment with **different pool sizes** and measure performance.
- Consider **multi-threaded performance testing**.

---

## 8. Multi-Threaded Performance Tests
Would you like to proceed with **multi-threaded testing** to simulate a real-world high-load scenario? 🚀

