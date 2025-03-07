# How Spring Boot Skips `DataSourceAutoConfiguration` for Manual Configuration

Spring Boot **uses conditional annotations** to check whether a `DataSource` bean already exists. If a **manual DataSource is defined**, Spring Boot **skips auto-configuration**.

---

## **1. `@ConditionalOnMissingBean(DataSource.class)` Prevents Auto-Configuration**
Inside `DataSourceAutoConfiguration`, Spring Boot **only configures a `DataSource` if none exists**:
```java
@Bean
@ConditionalOnMissingBean(DataSource.class)
public DataSource dataSource(DataSourceProperties properties) {
    return properties.initializeDataSourceBuilder().build();
}
```
> **How does it work?**  
> - `@ConditionalOnMissingBean(DataSource.class)` → **Checks if a `DataSource` bean is already present**.  
> - If **no DataSource exists**, Spring Boot **auto-configures it**.  
> - If **a custom DataSource exists**, Spring Boot **skips auto-configuration**.  

---

## **2. Proof: How to Verify Spring Boot Skips Auto-Configuration**

### **Step 1: Enable Debug Logging**
```properties
logging.level.org.springframework.boot.autoconfigure=DEBUG
```
### **Step 2: Run the Application & Check Logs**
If you define a manual DataSource, you will see this in logs:
```
DEBUG o.s.b.a.d.DataSourceAutoConfiguration - Skipping DataSource auto-configuration as a bean of type DataSource exists
```
> **This proves that Spring Boot detected your custom DataSource and disabled auto-configuration.**

---

## **3. What If You Remove the Custom DataSource?**
- **With Manual DataSource** → Spring Boot **skips auto-config**.  
- **Without Manual DataSource** → Spring Boot **automatically enables auto-config**.  

---

## **4. Summary: How Spring Boot Decides to Skip Auto-Configuration**
| **Condition** | **Spring Boot Behavior** |
|--------------|-------------------------|
| No `DataSource` Bean | ✅ Auto-configures `HikariCP` |
| Custom `DataSource` Bean Exists | ❌ Skips `DataSourceAutoConfiguration` |

---
