# Monitoring HikariCP with Spring Boot Actuator

## **1. Why Monitor HikariCP with Actuator?**
Monitoring HikariCP helps to:
✅ Detect **connection pool exhaustion**.  
✅ Identify **slow database connections**.  
✅ Ensure **optimal pool size configuration**.  

---

## **2. Adding Spring Boot Actuator for HikariCP Monitoring**

### **Step 1: Add Actuator Dependency**
Add the following dependency in `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

### **Step 2: Enable Actuator Endpoints in `application.properties`**
Expose all actuator endpoints (or just metrics):
```properties
management.endpoints.web.exposure.include=metrics,health
management.endpoint.health.show-details=always
management.metrics.export.prometheus.enabled=true
```

---

### **Step 3: Check HikariCP Metrics**
Start your Spring Boot application and access:
```
http://localhost:8080/actuator/metrics
```
Look for **HikariCP-related metrics**:
```
hikaricp.connections.active
hikaricp.connections.idle
hikaricp.connections.usage
hikaricp.connections.pending
```

---

## **3. Understanding HikariCP Actuator Metrics**
| **Metric** | **Description** |
|------------|---------------|
| `hikaricp.connections.active` | Number of connections currently in use |
| `hikaricp.connections.idle` | Number of connections available but idle |
| `hikaricp.connections.pending` | Number of requests waiting for a connection |
| `hikaricp.connections.usage` | Percentage of pool usage |
| `hikaricp.connections.creation` | Total number of created connections |
| `hikaricp.connections.timeout` | Number of connection timeouts |

---

## **4. Monitoring Connection Pool Usage**

### **Step 1: Query a Specific HikariCP Metric**
```
http://localhost:8080/actuator/metrics/hikaricp.connections.active
```
**Example Response:**  
```json
{
  "name": "hikaricp.connections.active",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 5.0
    }
  ]
}
```
> **Meaning:** 5 database connections are currently in use.

---

### **Step 2: Analyze Pool Usage**
If `hikaricp.connections.usage` is **above 80%**, consider increasing:
```properties
spring.datasource.hikari.maximum-pool-size=20
```
> **Prevents connection starvation under high load.**

---

## **5. Integrating HikariCP Metrics with Prometheus/Grafana**
If you need **graphical visualization**, integrate with **Prometheus & Grafana**:  
- Add **Micrometer Prometheus** dependency:
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```
- Run Prometheus and configure `scrape_config` for Spring Boot metrics:
```yaml
scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```
> ✅ Enables **real-time monitoring of HikariCP** in Grafana.

---

## **6. Summary**
| **Feature** | **Benefit** |
|------------|------------|
| **Actuator HikariCP Metrics** | Provides real-time database connection insights |
| **Prometheus/Grafana Integration** | Enables graphical monitoring of connection pool |
| **Connection Pool Alerts** | Detects exhaustion & performance issues |

---
