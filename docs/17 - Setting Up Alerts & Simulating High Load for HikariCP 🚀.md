# Setting Up Alerts & Simulating High Load for HikariCP 🚀

## **1. Setting Up Alerts When Pool Usage is Above 80%**

### **Step 1: Enable HikariCP Metrics in Spring Boot Actuator**
Ensure Actuator is enabled by adding this dependency in `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
Expose metrics in `application.properties`:
```properties
management.endpoints.web.exposure.include=metrics,health
management.metrics.export.prometheus.enabled=true
```

---

### **Step 2: Monitor Connection Pool Usage**
Check **HikariCP pool usage metric**:
```sh
http://localhost:8080/actuator/metrics/hikaricp.connections.usage
```
If the returned value **exceeds 0.8 (80%)**, the pool is **under pressure**.

---

### **Step 3: Configure Prometheus Alerting Rule**
📌 **Prometheus** can trigger an alert when **HikariCP pool usage exceeds 80%**.

Create a **Prometheus alerting rule (`hikari_alerts.yml`)**:
```yaml
groups:
  - name: hikari_alerts
    rules:
      - alert: HighHikariPoolUsage
        expr: hikaricp_connections_usage > 0.8
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "HikariCP pool usage is above 80%"
          description: "Database connection pool usage is at {{ $value }}. Consider increasing pool size."
```
Restart **Prometheus** with this rule:
```sh
docker run -p 9090:9090 -v $(pwd)/hikari_alerts.yml:/etc/prometheus/prometheus.yml prom/prometheus
```
✅ **Now, Prometheus will trigger an alert if pool usage exceeds 80% for over 1 minute!**

---

## **2. Simulating High Load to Test Connection Pool Settings**

### **Step 1: Write a High-Load Test Script**
Create a Java **multi-threaded test** to simulate high database requests:
```java
import java.sql.Connection;
import java.sql.SQLException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import javax.sql.DataSource;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class LoadTestRunner implements CommandLineRunner {

    private final DataSource dataSource;

    public LoadTestRunner(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public void run(String... args) {
        ExecutorService executor = Executors.newFixedThreadPool(50); // Simulating 50 concurrent users
        for (int i = 0; i < 100; i++) {
            executor.execute(() -> {
                try (Connection connection = dataSource.getConnection()) {
                    System.out.println("Connection acquired: " + connection);
                    Thread.sleep(500); // Simulate query execution
                } catch (SQLException | InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        executor.shutdown();
    }
}
```
📌 **This script:**  
- Creates **50 concurrent threads**.  
- Each thread **acquires a DB connection and holds it for 500ms**.  
- Helps simulate **high load and test pool performance**.

---

### **Step 2: Run Load Test and Observe Metrics**
Start the Spring Boot application and check:
```sh
http://localhost:8080/actuator/metrics/hikaricp.connections.active
http://localhost:8080/actuator/metrics/hikaricp.connections.pending
```
> If `connections.pending` increases, **requests are waiting** for available connections.

---

### **Step 3: Adjust Connection Pool Settings**
If **pool usage exceeds 80%**, optimize it:

#### 🔹 **Increase Pool Size** (if demand is high):
```properties
spring.datasource.hikari.maximum-pool-size=50
```

#### 🔹 **Reduce Connection Timeout** (prevent long waits):
```properties
spring.datasource.hikari.connection-timeout=3000  # 3 seconds
```

#### 🔹 **Enable Leak Detection** (identify unclosed connections):
```properties
spring.datasource.hikari.leak-detection-threshold=2000
```

---

## **3. Summary**
| **Action** | **What It Does** |
|------------|-----------------|
| **Set up Prometheus Alerts** | Warns if pool usage exceeds 80% |
| **Run Load Test** | Simulates high concurrent requests |
| **Monitor Metrics** | Checks if connections are exhausted |
| **Tune HikariCP Settings** | Adjusts pool size & timeouts |

---
