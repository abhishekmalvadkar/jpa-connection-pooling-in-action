# Multi-Threaded Performance Testing for Connection Pooling

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

## 3. Multi-Threaded Performance Testing for Connection Pooling

### Why Multi-Threaded Testing?
- Simulates **real-world concurrent requests**.
- Measures **performance under high load**.
- Compares connection pooling efficiency.

### Key Metrics:
- **Total Execution Time** for 100 concurrent requests.
- **Average Response Time** per connection.
- **Throughput** (requests per second).

---

## 4. Multi-Threaded Test for Plain JDBC (No Pooling)
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class PlainJDBCMultiThreadedTest {
    private static final String URL = "jdbc:mysql://localhost:3306/mydb";
    private static final String USER = "root";
    private static final String PASSWORD = "password";
    private static final int THREAD_COUNT = 100;

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        long start = System.currentTimeMillis();

        for (int i = 0; i < THREAD_COUNT; i++) {
            executor.execute(() -> {
                try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD)) {
                    System.out.println(Thread.currentThread().getName() + " - Connected using Plain JDBC");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.MINUTES);
        long end = System.currentTimeMillis();
        System.out.println("Time taken without pooling: " + (end - start) + "ms");
    }
}
```

---

## 5. Multi-Threaded Test for Apache DBCP
```java
import org.apache.commons.dbcp2.BasicDataSource;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class DBCPMultiThreadedTest {
    private static final BasicDataSource dataSource = new BasicDataSource();
    private static final int THREAD_COUNT = 100;

    static {
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setMinIdle(5);
        dataSource.setMaxIdle(10);
        dataSource.setMaxTotal(20);
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        long start = System.currentTimeMillis();

        for (int i = 0; i < THREAD_COUNT; i++) {
            executor.execute(() -> {
                try (Connection conn = dataSource.getConnection()) {
                    System.out.println(Thread.currentThread().getName() + " - Connected using DBCP");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.MINUTES);
        long end = System.currentTimeMillis();
        System.out.println("Time taken with DBCP: " + (end - start) + "ms");
    }
}
```

---

## 6. Multi-Threaded Test for HikariCP
```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class HikariCPMultiThreadedTest {
    private static final HikariDataSource dataSource;
    private static final int THREAD_COUNT = 100;

    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setIdleTimeout(30000);
        config.setConnectionTimeout(2000);
        dataSource = new HikariDataSource(config);
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        long start = System.currentTimeMillis();

        for (int i = 0; i < THREAD_COUNT; i++) {
            executor.execute(() -> {
                try (Connection conn = dataSource.getConnection()) {
                    System.out.println(Thread.currentThread().getName() + " - Connected using HikariCP");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.MINUTES);
        long end = System.currentTimeMillis();
        System.out.println("Time taken with HikariCP: " + (end - start) + "ms");
        dataSource.close();
    }
}
```

---

## 7. Summary of Multi-Threaded Performance
| Feature | Plain JDBC | Apache DBCP | HikariCP |
|---------|-----------|-------------|----------|
| Connection Creation Overhead | High | Medium | Low |
| Performance Under Load | Poor | Good | Excellent |
| Scalability | Poor | Moderate | High |
| Response Time | Slow | Medium | Fast |
| Best for Production | ❌ No | ✅ Maybe | ✅ Yes |

---

