# Logging for HikariCP Behavior

## **1. Why Enable Logging for HikariCP?**
HikariCP logs help:
- Debug **connection leaks**.
- Monitor **pool size adjustments**.
- Detect **slow queries or timeouts**.

---

## **2. Enabling HikariCP Logging in Spring Boot**

### **Step 1: Enable Debug Logs for HikariCP**
Add this to `application.properties`:
```properties
logging.level.com.zaxxer.hikari=DEBUG
logging.level.com.zaxxer.hikari.HikariConfig=TRACE
logging.level.com.zaxxer.hikari.pool=DEBUG
```
> This logs **HikariCP configuration** and **runtime pool behavior**.

### **Step 2: Restart Your Spring Boot App**
Check the logs:
```
DEBUG com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Starting...
DEBUG com.zaxxer.hikari.pool.HikariPool - Pool stats (total=10, active=2, idle=8, waiting=0)
```
> **Meaning:**
> - **10 total connections in the pool**
> - **2 active, 8 idle connections**
> - **No waiting requests**

---

## **3. Detecting Connection Leaks**

### **Enable Leak Detection**
```properties
spring.datasource.hikari.leak-detection-threshold=2000
```
> If a connection is held **longer than 2 seconds**, this appears in logs:
```
[HikariCP connection leak detection] Connection leak detected
```
> **Solution:** Ensure all connections are **closed** after use.

---

## **4. Identifying Connection Timeouts**
If the database is slow, you might see:
```
WARN com.zaxxer.hikari.pool.PoolBase - Failed to validate connection
```
### **Fix: Increase Connection Timeout**
```properties
spring.datasource.hikari.connection-timeout=5000
```
> **This allows up to 5 seconds before a connection times out.**

---

## **5. Full Logging Configuration for HikariCP**
```properties
logging.level.com.zaxxer.hikari=DEBUG
logging.level.com.zaxxer.hikari.HikariConfig=TRACE
logging.level.com.zaxxer.hikari.pool=DEBUG

spring.datasource.hikari.leak-detection-threshold=2000
spring.datasource.hikari.connection-timeout=5000
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
```

---

## **6. Summary**
| **Log Type** | **Purpose** |
|-------------|------------|
| **Pool Stats Logs** | Track active & idle connections |
| **Leak Detection Logs** | Detect unclosed connections |
| **Timeout Logs** | Identify slow queries or database slowness |

---
