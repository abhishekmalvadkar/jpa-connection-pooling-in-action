# Spring Boot Connection Pooling Journey Across Versions 🚀

Spring Boot has evolved its **default connection pool** over various versions, improving **performance, stability, and flexibility**.

---

## **1. Spring Boot 1.x (2014 - 2018) → Default: Tomcat JDBC Pool**
🔹 **Spring Boot 1.x** used **Tomcat JDBC Connection Pool** by default.  
🔹 Tomcat JDBC was **better than raw JDBC but slower than HikariCP**.

### 🔍 **How It Worked in Spring Boot 1.x?**
- If you added **`spring-boot-starter-jdbc`** or **`spring-boot-starter-data-jpa`**, Spring Boot auto-configured **Tomcat JDBC Pool**:
```properties
spring.datasource.tomcat.max-active=50
spring.datasource.tomcat.max-idle=10
```
- You could manually **switch to HikariCP** by adding:
```properties
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
```

🔴 **Issue:** Tomcat JDBC Pool had **higher memory usage and slower performance** than modern pools.

---

## **2. Spring Boot 2.x (2018 - 2023) → Default: HikariCP**
✅ **Spring Boot 2.0+ switched to HikariCP as the default connection pool!**  

### 🔍 **Why Did Spring Boot Choose HikariCP?**
- **Faster** 🚀 (Low-latency, high-performance pooling).
- **Lower memory footprint** (Optimized for modern cloud environments).
- **Better connection leak detection**.

### 🔍 **How It Worked in Spring Boot 2.x?**
By default, **HikariCP was auto-configured** when using:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```
**HikariCP properties:**
```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=5000
```
🔹 **If HikariCP was missing**, Spring Boot **fell back to Tomcat JDBC Pool**.

---

## **3. Spring Boot 3.x (2023 - Present) → HikariCP with Better Observability**
📌 **Spring Boot 3.x continues to use HikariCP** but introduces **enhanced observability & monitoring**.  
📌 **Micrometer & Prometheus integration** for tracking **connection pool performance**.  

### 🔍 **What’s New in Spring Boot 3.x?**
✅ **Automatic connection pooling metrics** with `spring-boot-actuator`.  
✅ **Improved failover handling** for database connections.  
✅ **Native support for `DataSourceAutoConfiguration` with Spring AOT (Ahead-Of-Time Compilation)**.

🔹 **HikariCP Metrics in Actuator:**
```properties
management.endpoints.web.exposure.include=metrics
management.metrics.export.prometheus.enabled=true
```
🔹 **Monitor connection pool usage at:**
```
http://localhost:8080/actuator/metrics/hikaricp.connections.active
```

---

## **4. Spring Boot Connection Pool Evolution Summary**
| **Spring Boot Version** | **Default Connection Pool** | **Reason for Change** |
|----------------|-------------------|----------------|
| **1.x (2014-2018)** | Tomcat JDBC Pool | Default, but slow & high memory usage |
| **2.x (2018-2023)** | HikariCP | Faster, low-latency, optimized for modern apps |
| **3.x (2023-Present)** | HikariCP + Observability | Improved monitoring, better failover support |

---
