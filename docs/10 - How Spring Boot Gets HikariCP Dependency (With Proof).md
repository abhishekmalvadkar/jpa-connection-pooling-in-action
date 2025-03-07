# How Spring Boot Gets HikariCP Dependency (With Proof)

## **1. Spring Boot Auto-Configuration for HikariCP**
Spring Boot follows this **dependency selection order** for connection pooling:
1. **HikariCP** (Default & Fastest 🚀)
2. **Tomcat JDBC Pool**
3. **Apache Commons DBCP2**

### **Where is HikariCP coming from?**
If you **don’t specify a connection pool**, Spring Boot **automatically pulls HikariCP** from the following dependencies:

#### ✅ **If you use JDBC (`spring-boot-starter-jdbc`)**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```
> **HikariCP is transitively included** inside this starter.

#### ✅ **If you use JPA (`spring-boot-starter-data-jpa`)**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
> **HikariCP is also included** in this starter automatically.

---

## **2. Proof: Checking HikariCP in the Dependency Tree**
Run this command in your Spring Boot project:
```sh
mvn dependency:tree | grep hikari
```
Expected output:
```
[INFO] com.zaxxer:HikariCP:jar:5.0.1:compile
```
> **This proves that Spring Boot has included HikariCP automatically.**

---

## **3. Proof: Checking Active Connection Pool in Logs**
Enable **debug logging** in `application.properties`:
```properties
logging.level.org.springframework.boot.autoconfigure.jdbc=DEBUG
```
Start the application and look for this log:
```
DEBUG o.s.b.a.d.DataSourceAutoConfiguration - Auto-configuring HikariCP as the default DataSource
```
> **This confirms that Spring Boot is using HikariCP by default.**

---

## **4. What If You Want a Different Connection Pool?**
If you want to **switch from HikariCP to another pool**, exclude it:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
Then, add **Tomcat JDBC Pool** manually:
```xml
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jdbc</artifactId>
</dependency>
```
> **Spring Boot will now use Tomcat JDBC Pool instead.**

---

## **5. Summary**
| **Method** | **Proof** |
|------------|----------|
| **Maven Dependency Check** | `mvn dependency:tree | grep hikari` |
| **Spring Boot Auto-Config Logs** | `DEBUG o.s.b.a.d.DataSourceAutoConfiguration - Auto-configuring HikariCP` |
| **Changing Connection Pool** | Exclude HikariCP & add another pool manually |

---
