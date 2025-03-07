# Case Study: Connection Pooling Optimization in a High-Traffic E-commerce Application

## **Background**
An **e-commerce platform** was experiencing:
🔴 **Slow response times during peak traffic** (Black Friday sales).  
🔴 **Database connection timeouts** leading to failed transactions.  
🔴 **Connection pool starvation** due to long-running queries.  

### **Architecture**
- **Tech Stack**: Spring Boot, MySQL, Hibernate, HikariCP  
- **Traffic**: 5,000+ concurrent users during sales events  

---

## **1. Initial Issues Identified**

📌 **High Latency & Connection Starvation**
- `hikaricp.connections.pending` **spiked to 50+** under high load.
- `hikaricp.connections.active` **reached maximum pool size (10 by default)**.
- API response times **jumped from 300ms to 5s** due to slow queries.

📌 **Connection Leaks**
- Queries were **holding connections too long**, causing **pool exhaustion**.
- Some transactions **weren’t committed properly**, leaving connections open.

📌 **Improper Connection Pool Settings**
- **Default max pool size (10)** was too low for 5,000 concurrent users.
- **Idle connections weren’t released efficiently**.
- **Timeout settings were too high**, causing blocked threads.

---

## **2. Optimization Strategy**

### ✅ **1. Increased Connection Pool Size**
**Before:**
```properties
spring.datasource.hikari.maximum-pool-size=10
```
**After:**
```properties
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
```
🔹 **Now, 50 users can be served simultaneously.**

---

### ✅ **2. Reduced Connection Timeout**
**Before:**
```properties
spring.datasource.hikari.connection-timeout=30000  # 30s
```
**After:**
```properties
spring.datasource.hikari.connection-timeout=5000  # 5s
```
🔹 **Now, if a connection isn’t available in 5s, an error is thrown instead of waiting indefinitely.**

---

### ✅ **3. Optimized Query Performance**
- **Indexed slow queries** in MySQL.  
- **Reduced unnecessary joins**.  
- **Refactored long-running transactions** to execute in smaller chunks.  

🔹 **Result: Query execution time reduced from 2s → 150ms!**

---

### ✅ **4. Enabled Connection Leak Detection**
```properties
spring.datasource.hikari.leak-detection-threshold=2000
```
🔹 **Any connection held longer than 2s is logged to detect leaks.**

---

### ✅ **5. Implemented Prometheus Alerts for Connection Starvation**
Configured **Prometheus & Alertmanager** to monitor:
```yaml
- alert: HighHikariPoolUsage
  expr: hikaricp_connections_usage > 0.8
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "🔥 High connection pool usage detected!"
    description: "HikariCP is at {{ $value }}% usage."
```
🔹 **Slack & Email alerts notify the team if pool usage exceeds 80%.**

---

## **3. Results After Optimization**

📈 **Performance Improvements**
| Metric | Before Optimization | After Optimization |
|--------|---------------------|--------------------|
| **Avg API Response Time** | **5000ms** | **250ms** |
| **Connection Timeout Errors** | **50+ per minute** | **<5 per hour** |
| **Pool Usage** | **100% (starvation)** | **70% (optimized)** |

✅ **Black Friday Sale handled 10,000 concurrent users with zero downtime!** 🚀

---

## **4. Key Takeaways**
| **Optimization** | **Benefit** |
|-----------------|------------|
| **Increased Pool Size** | Handled more concurrent users |
| **Reduced Connection Timeout** | Prevented long waiting requests |
| **Query Optimization** | Improved API response time |
| **Connection Leak Detection** | Ensured proper connection closing |
| **Prometheus Alerts** | Proactive issue detection |

---
