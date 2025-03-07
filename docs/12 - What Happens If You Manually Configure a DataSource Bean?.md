# What Happens If You Manually Configure a DataSource Bean?

If you **define a custom `DataSource` bean**, Spring Boot **disables its automatic DataSource configuration** (`DataSourceAutoConfiguration`).

---

## **1. Spring Boot Skips Auto-Configuration**  
Spring Boot’s **auto-configuration is conditional**. If a `DataSource` bean exists, `DataSourceAutoConfiguration` **does not execute**.

### **Example: Manually Configuring a DataSource**
```java
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;

@Configuration
public class CustomDataSourceConfig {

    @Bean
    public DataSource customDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setMaximumPoolSize(20);
        return dataSource;
    }
}
```
> **Now, Spring Boot will NOT configure a default DataSource** because it detects a manually defined bean.

---

## **2. Proof: How to Verify Spring Boot Skips Auto-Configuration**

### **Step 1: Enable Debug Logs**
Add this to `application.properties`:
```properties
logging.level.org.springframework.boot.autoconfigure=DEBUG
```

### **Step 2: Run the Application & Check Logs**
Look for:
```
DEBUG o.s.b.a.d.DataSourceAutoConfiguration - Skipping DataSource auto-configuration as a bean of type DataSource exists
```
> **This confirms that Spring Boot detected your custom DataSource and skipped auto-configuration.**

---

## **3. What If You Want to Use Auto-Configuration Again?**
If you remove the custom `DataSource` bean, Spring Boot **automatically re-enables `DataSourceAutoConfiguration`** and selects the default pool (`HikariCP`).

---

## **4. Summary**
| **Scenario** | **Behavior** |
|-------------|-------------|
| **No Custom DataSource** | Spring Boot auto-configures HikariCP |
| **Custom DataSource Defined** | Spring Boot **disables auto-configuration** |
| **Remove Custom DataSource** | Auto-configuration is **enabled again** |

---
