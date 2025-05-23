**Student Reference Manual: Okta OAuth2.0 / OIDC & Secure App Integration**

---

... (previous content unchanged)

---

### **21. Angular Frontend (PKCE + OIDC Integration)**

#### 🔧 Required Package:

```bash
npm install angular-oauth2-oidc --save
```

#### 🧩 `app.module.ts`

```ts
import { OAuthModule } from 'angular-oauth2-oidc';

@NgModule({
  imports: [
    OAuthModule.forRoot({
      resourceServer: {
        allowedUrls: ['http://localhost:8080/user'],
        sendAccessToken: true,
      },
    })
  ]
})
export class AppModule {}
```

#### ⚙️ `auth.config.ts`

```ts
export const authConfig = {
  issuer: 'https://dev-XXXX.okta.com/oauth2/default',
  redirectUri: window.location.origin + '/index.html',
  clientId: 'YOUR_CLIENT_ID',
  responseType: 'code',
  scope: 'openid profile email',
  showDebugInformation: true,
  usePkce: true,
};
```

#### 🚀 `app.component.ts`

```ts
import { OAuthService } from 'angular-oauth2-oidc';
import { authConfig } from './auth.config';

constructor(private oauthService: OAuthService) {
  this.configureAuth();
}

configureAuth() {
  this.oauthService.configure(authConfig);
  this.oauthService.loadDiscoveryDocumentAndTryLogin();
}

login() {
  this.oauthService.initCodeFlow();
}

logout() {
  this.oauthService.logOut();
}
```

---

### **22. Integrate Swagger for API Testing**

#### 🧩 `pom.xml`

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-ui</artifactId>
  <version>1.6.14</version>
</dependency>
```

#### 🌐 Swagger UI Endpoint:

```
http://localhost:8080/swagger-ui.html
```

SpringDoc automatically scans controller endpoints and exposes them for testing and visualization.

---

### **23. Containerize with Docker**

#### 📄 `Dockerfile` for Spring Boot

```dockerfile
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 📄 `Dockerfile` for Angular

```dockerfile
# Stage 1: Build
FROM node:18 as build
WORKDIR /app
COPY . .
RUN npm install && npm run build --prod

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/dist/your-app-name /usr/share/nginx/html
```

#### 📄 `docker-compose.yml`

```yaml
version: '3.8'
services:
  backend:
    build: ./backend-springboot
    ports:
      - '8080:8080'
  frontend:
    build: ./frontend-angular
    ports:
      - '4200:80'
```

---

### **24. Kubernetes Manifests**

#### 🗂️ `deployment-backend.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: your-registry/springboot-app:latest
        ports:
        - containerPort: 8080
```

#### 🗂️ `service-backend.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

#### 🗂️ `deployment-frontend.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: angular-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: your-registry/angular-app:latest
        ports:
        - containerPort: 80
```

#### 🗂️ `service-frontend.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```
