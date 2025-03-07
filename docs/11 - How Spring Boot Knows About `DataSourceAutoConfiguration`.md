# How Spring Boot Knows About `DataSourceAutoConfiguration`

## **1. Spring Boot Uses `spring.factories` for Auto-Configuration**
Spring Boot **discovers `DataSourceAutoConfiguration`** using the `spring-boot-autoconfigure` module, which contains a file:

📄 `META-INF/spring.factories`
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
```
> **This file tells Spring Boot** which auto-configurations to load.

---

## **2. `@EnableAutoConfiguration` Triggers the Auto-Configuration**
When you add `spring-boot-starter-data-jpa` or `spring-boot-starter-jdbc`, Spring Boot automatically includes:
```java
@SpringBootApplication  // Internally includes @EnableAutoConfiguration
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```
> **Internally, `@EnableAutoConfiguration` reads the `spring.factories` file** and loads `DataSourceAutoConfiguration`.

---

## **3. Proof: Listing Auto-Configuration Classes**
Run this command to list all auto-configured classes:
```sh
mvn spring-boot:run -Ddebug
```
Look for:
```
DataSourceAutoConfiguration matched
```
> **This proves that Spring Boot detected and loaded `DataSourceAutoConfiguration`.**

---

## **4. Summary: How Spring Boot Loads `DataSourceAutoConfiguration`**
| **Step** | **What Happens?** |
|----------|------------------|
| **1. Spring Boot Starts** | Reads `spring.factories` |
| **2. Finds Auto-Configurations** | Includes `DataSourceAutoConfiguration` |
| **3. Checks Conditions** | If `DataSource` and JDBC exist, loads configuration |
| **4. Configures DataSource** | Picks HikariCP, Tomcat, or DBCP2 |

---
