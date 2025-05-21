**Student Reference Manual: Okta OAuth2.0 / OIDC & Secure App Integration**

---


---

### **19. Complete Spring Boot Code with Explanation**

This example demonstrates how to secure a Spring Boot application using OAuth 2.0 with Okta.

---

#### 📁 **Project Structure**

```
backend-springboot/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/demo/
│       │       ├── DemoApplication.java
│       │       ├── config/SecurityConfig.java
│       │       └── controller/UserController.java
│       └── resources/
│           └── application.yml
└── pom.xml
```

---

### 🧱 **Step 1: pom.xml**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>okta-springboot</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>okta-springboot</name>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-oauth2-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-ui</artifactId>
      <version>1.6.14</version>
    </dependency>
  </dependencies>
</project>
```

---

### ⚙️ **Step 2: application.yml**

```yaml
server:
  port: 8080

spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            client-id: YOUR_OKTA_CLIENT_ID
            client-secret: YOUR_OKTA_CLIENT_SECRET
            scope: openid, profile, email
        provider:
          okta:
            issuer-uri: https://dev-YOURDOMAIN.okta.com/oauth2/default
```

---

### 🔐 **Step 3: SecurityConfig.java**

```java
package com.example.demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {
  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      .authorizeRequests()
        .anyRequest().authenticated()
        .and()
      .oauth2Login();
    return http.build();
  }
}
```

---

### 📘 **Step 4: DemoApplication.java**

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```

---

### 👤 **Step 5: UserController.java**

```java
package com.example.demo.controller;

import org.springframework.security.oauth2.client.authentication.OAuth2AuthenticationToken;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
public class UserController {
  @GetMapping("/user")
  public Map<String, Object> userInfo(OAuth2AuthenticationToken authentication) {
    return authentication.getPrincipal().getAttributes();
  }
}
```

**What this does:**

* On visiting `http://localhost:8080/user`, users are redirected to Okta login.
* Upon successful authentication, user attributes are returned as JSON.

---


