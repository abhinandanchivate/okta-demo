**Student Reference Manual: Okta OAuth2.0 / OIDC & Secure App Integration**

---


---

### **20. Combined Workflow: Angular + Spring Boot + Okta (End-to-End)**

This diagram and explanation combine both the **external SPA-to-Okta flow** and **internal Spring Boot handling** to illustrate the complete OAuth2.0 authentication and authorization process.

#### 🔁 **End-to-End OAuth2 + OIDC Workflow**

1. **Browser accesses secured Angular route**
2. **Angular detects no session → initiates Authorization Request to Okta**
3. **User logs in via Okta UI**
4. Okta redirects back with **Authorization Code** to Angular
5. Angular forwards code to Spring Boot (BFF pattern)
6. **Spring Security** requests **Access Token + ID Token** from Okta
7. **Tokens are validated** by Spring using `NimbusJwtDecoder`
8. **Claims from ID Token are extracted**, stored in `SecurityContextHolder`
9. Controller endpoint `/user` is accessed with the token and returns user info

#### ⚙️ **Spring Boot Internals with Spring Security**

* Spring Security auto-configures `OAuth2LoginAuthenticationFilter`
* This filter intercepts `/login/oauth2/code/okta` redirection endpoint
* The `AuthorizationCodeAuthenticationProvider` handles:

  * Validating the code
  * Requesting tokens from Okta
  * Creating an authenticated principal (`DefaultOidcUser`)
* The `JwtDecoder` validates the ID Token signature using Okta's public keys (via JWKS)
* SecurityContext is populated with authenticated principal
* The `OAuth2AuthenticationToken` object can now be injected into controllers

#### 🔐 **How Okta Validates the Details**

* Angular initiates login using **Authorization Code + PKCE**
* Okta presents hosted login page (customizable)
* After login, it redirects with **auth code** to redirect URI
* Spring Boot sends POST request to Okta token endpoint
* Okta validates:

  * Client ID, Secret
  * Redirect URI
  * Code expiration & integrity
* If valid:

  * Returns Access Token, ID Token, and optionally Refresh Token
* Okta signs the ID Token using its **private RSA key**
* Spring Boot retrieves Okta JWKS URI and fetches public keys to **verify token signature**
* Claims inside the ID Token (e.g., `email`, `sub`, `roles`) are parsed

#### 🖼️ **Unified Workflow Diagram**

```
Browser → Angular → Okta Login
   ↓         ↓         ↓
 Access  Auth Code   Redirect
   ↓         ↓         ↓
  → Spring Boot (/login/oauth2/code/okta)
       ↓
  OAuth2LoginAuthenticationFilter triggered
       ↓
  Spring sends Token Request to Okta
       ↓
  Okta returns ID + Access Tokens
       ↓
  JwtDecoder validates ID Token via Okta JWKS
       ↓
  Populate SecurityContextHolder with user principal
       ↓
  Controller `/user` returns user claims
```

