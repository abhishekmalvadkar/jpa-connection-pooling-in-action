## Real-World Debugging Techniques for Connection Pooling

### 🔹 1. **Detecting Connection Leaks**
**Issue:** Over time, the connection pool gets exhausted, leading to `ConnectionTimeoutException` errors.

**How to Debug:**
- Enable **leak detection** in HikariCP:
  ```properties
  spring.datasource.hikari.leak-detection-threshold=2000  # Logs connections held for >2s
  ```
- Use database tools (`SHOW PROCESSLIST` in MySQL) to check long-running queries.
- Ensure `try-with-resources` is used to close connections automatically:
  ```java
  try (Connection conn = dataSource.getConnection()) {
      // Use connection
  } // Auto-closed
  ```

---

### 🔹 2. **Handling Connection Timeout Errors**
**Issue:** Application logs show frequent `Connection is not available` exceptions.

**How to Debug:**
- Increase the connection pool size:
  ```properties
  spring.datasource.hikari.maximum-pool-size=30
  ```
- Check database load (CPU/memory/disk usage) to identify bottlenecks.
- Ensure queries are optimized to reduce long-held connections.

---

### 🔹 3. **Identifying Stale or Broken Connections**
**Issue:** Random `Connection is closed` or `Socket timeout` errors.

**How to Debug:**
- Enable **validation query** in the pool:
  ```properties
  spring.datasource.hikari.validation-timeout=3000
  spring.datasource.hikari.connection-test-query=SELECT 1
  ```
- Use HikariCP's **keepalive feature** to prevent premature disconnections:
  ```properties
  spring.datasource.hikari.keepalive-time=30000  # 30s
  ```
- Ensure the database allows long-lived connections (`wait_timeout` in MySQL).

---

### 🔹 4. **Reducing High Latency in Connection Acquisition**
**Issue:** Queries take too long due to connection wait times.

**How to Debug:**
- Check the pool usage with JMX (`ActiveConnections`, `IdleConnections`).
- Increase `minimumIdle` to keep more connections available:
  ```properties
  spring.datasource.hikari.minimum-idle=10
  ```
- Reduce `connectionTimeout` to detect issues faster:
  ```properties
  spring.datasource.hikari.connection-timeout=5000  # 5s
  ```

---

### 🔹 5. **Monitoring Connection Pool Performance**
**Best Practices:**
- Enable **JMX monitoring** to track connection metrics:
  ```properties
  spring.datasource.hikari.register-mbeans=true
  ```
- Use **Prometheus + Grafana** to visualize database connection statistics.
- Log pool statistics periodically:
  ```java
  HikariDataSource ds = (HikariDataSource) dataSource;
  System.out.println("Active connections: " + ds.getHikariPoolMXBean().getActiveConnections());
  ```

