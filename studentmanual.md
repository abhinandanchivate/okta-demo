
---

### ✅ Expanded Student Manual: Sections 1–19

---

### **1. Overall Architecture / Fundamentals**

* **OAuth 2.0** is not about authentication; it is a delegated authorization protocol. It allows clients (e.g., Angular apps) to access user data via tokens from an Authorization Server (e.g., Okta).

* Key roles:

  * **Resource Owner**: User authorizing access.
  * **Client**: App that requests access to the resource.
  * **Authorization Server**: Okta — issues access/ID tokens.
  * **Resource Server**: Backend API which validates access tokens.

* Tokens:

  * **Access Token**: Used by client to access APIs.
  * **ID Token**: Identity information (used in OIDC).
  * **Refresh Token**: Long-lived token to renew access tokens.

* Key security guidelines:

  * Always use HTTPS.
  * Use short-lived tokens.
  * Apply least-privilege principles to scopes.
  * Rotate client secrets.

**Real-world usage**: In a banking SPA, the Angular frontend never handles tokens directly; Spring Boot backend (BFF) stores them securely in the session.

---

### **2. OAuth 2.0 / OIDC – Fundamentals**

* **OAuth 2.0**: Manages user authorization — who can do what.

* **OIDC (OpenID Connect)**: Layer over OAuth to manage authentication — who the user is.

* OIDC introduces:

  * **ID Token**: A signed JWT issued by the identity provider (Okta).
  * **Scopes**: `openid`, `profile`, `email`, etc.
  * **UserInfo endpoint**: Optional call to fetch full profile.

* ID Token example (JWT):

```json
{
  "sub": "abc123",
  "name": "John Doe",
  "email": "john@example.com",
  "iat": 1710000000,
  "exp": 1710003600
}
```

---

### **3. Use Cases – What to Use When**

| Scenario         | Flow                      | Justification                     |
| ---------------- | ------------------------- | --------------------------------- |
| SPA (Angular)    | Authorization Code + PKCE | Prevent token leakage in browser  |
| Mobile App       | Authorization Code + PKCE | Protect with device trust         |
| Web App (Server) | Authorization Code        | Tokens stay on server (secure)    |
| M2M              | Client Credentials        | No user involved                  |
| IoT Devices      | Device Code Flow          | No input interface for user login |

---

### **4. End-to-End OAuth 2.0 / OIDC Flows in Auth0**

* **SPA Flow (PKCE)**:

  1. Angular redirects to Auth0.
  2. User logs in → gets Auth Code.
  3. Angular exchanges Auth Code + Code Verifier for tokens.
  4. Uses Access Token for API calls.

* **M2M Flow**:

  1. Backend sends client ID/secret to `/token`.
  2. Receives Access Token.
  3. Makes authorized calls to Resource Server.

**Best practice**: Use rotating refresh tokens for SPAs and manage expiration via silent renew or background polling.

---

### **5. User Management in Okta**

* Admin can:

  * Create users.
  * Assign them to groups.
  * Apply RBAC (roles define scopes/permissions).

* SCIM protocol support: Auto-provisioning/de-provisioning users.

* Token customization:

  * Add group or department to ID token using Okta claims engine.

---

### **6. Auth0 Tenant and Organization**

* **Tenant**: Logical unit — your sandbox.
* **Organization**: Logical container for B2B use cases.

**Example**:

* One SaaS tenant serves `acme.com` and `zenith.io`.
* Each has its own logo, login policy, and role mapping.

---

### **7. Management API and Security for Management API**

* Manage:

  * Users
  * Applications
  * Clients
  * Roles
* Secure with:

  * `client_credentials` flow
  * Scopes like `read:users`, `update:clients`
  * IP allowlisting

**Tip**: Never expose Management API tokens to frontend.

---

### **8. Demo: BFF Framework (Java + Spring Boot + API Gateway)**

* Angular only talks to Spring Boot BFF, which:

  * Initiates login with Okta
  * Exchanges Auth Code
  * Stores tokens in `HttpSession`
  * Calls internal microservices using Access Token

**Advantage**: No token exposed to browser. Stateless API security is handled server-side.

---

### **9. Best Practices – Exposing REST APIs to 3rd Party Systems**

* Use JWT token validation middleware.
* Validate:

  * Signature
  * Issuer (`iss`)
  * Audience (`aud`)
  * Expiry
* Rate-limit per API key/client ID.

**Recommendation**: Include a `/health` and `/introspect` endpoint with proper protection.

---

### **10. Best Practices – Mobile App Integration**

* Always use PKCE.
* Store refresh tokens in `Keystore` (Android) or `Keychain` (iOS).
* Prefer biometric-based reauthentication.
* Use token introspection or silent renew for background updates.

---

### **11. How to Log In and Generate Token (Spring Boot + Okta)**

* Configure in `application.yml`:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            client-id: xyz
            client-secret: abc
        provider:
          okta:
            issuer-uri: https://dev-XXXX.okta.com/oauth2/default
```

* Automatically configures `/login/oauth2/code/okta`.

* Controller:

```java
@GetMapping("/user")
public Map<String, Object> getUser(OAuth2AuthenticationToken token) {
    return token.getPrincipal().getAttributes();
}
```

---

### **12. Full Project Setup and Token Consumption (Spring Boot + Angular)**

* Angular → Okta Login → Auth Code
* Spring Boot → Token exchange
* `/user` returns user data
* All frontend requests add `Bearer <AccessToken>` header.

---

### **13. GitHub-Ready Sample Project Repository**

* Folder layout:

```
/frontend-angular
/backend-springboot
/docker-compose.yml
/.github/workflows/deploy.yml
```

* README with:

  * How to run
  * Setup Okta
  * Environment variables

---

### **14. Dockerized Setup for Fullstack Deployment**

* Backend:

```dockerfile
FROM eclipse-temurin:17
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

* Frontend:

```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build --prod
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

---

### **15. Swagger/OpenAPI Integration for Backend**

* Add:

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-ui</artifactId>
  <version>1.6.14</version>
</dependency>
```

* Secure with:

```java
@Configuration
public class SwaggerSecurityConfig { ... }
```

---

### **16. Downloadable ZIP of Full Working Project**

* Host ZIP on GitHub Release or AWS S3.
* Include:

  * `README.md`
  * `.env.example`
  * Setup scripts

---

### **17. Architecture Diagram (Okta + Angular + Spring Boot + Docker)**

* Structure:

```
[Browser] → [Angular] → [Spring Boot BFF] → [Okta]
                                     ↓
                              [Internal APIs]
```

* Tokens validated in backend only.

---

### **18. DevOps CI/CD Pipeline Example – GitHub Actions**

* Workflow:

```yaml
- checkout
- setup-java
- mvn build
- setup-node
- npm build
- docker build + push
```

* Use GitHub secrets:

  * `DOCKER_USERNAME`
  * `DOCKER_PASSWORD`

---

### **19. Complete Spring Boot Code with Explanation**

* Files:

  * `SecurityConfig.java` — OAuth2 client setup
  * `UserController.java` — Secured endpoint
  * `application.yml` — Okta integration
  * `pom.xml` — Spring Boot, OAuth2, OpenAPI

**Architecture Tip**: Split into:

* `controller`
* `service`
* `security`
* `config`

---

