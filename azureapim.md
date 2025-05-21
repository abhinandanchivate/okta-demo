 **fully consolidated, production-grade implementation guide Azure APIM** for your **Java-based Backend-for-Frontend (BFF) architecture**, with:

✅ Azure Key Vault
✅ Spring Boot Microservices
✅ Azure API Management (APIM)
✅ Prometheus + Grafana
✅ Docker, AKS, Helm
✅ Swagger, CI/CD, Postman

---

# 🌐 OVERALL ARCHITECTURE

```
[ Angular SPA / Mobile App ]
        │
        ▼
[ Azure API Management (APIM) ]
  - JWT validation
  - Rate limiting
  - Routing
        │
        ▼
[ Java Spring Boot BFF ]
  - Token verification
  - Aggregates + transforms data
  - Calls microservices
        │
 ┌──────┼────────────┐
 ▼      ▼            ▼
User-MS Order-MS   Product-MS
        │
        ▼
[ Azure Key Vault ]
  - Secrets (DB passwords, API keys)
        │
        ▼
[ Prometheus + Grafana ]
  - Metrics via Actuator
  - Dashboard visualization
```

---

# 🔹 COMPONENTS USED

| Layer             | Technology                     |
| ----------------- | ------------------------------ |
| Gateway/API Mgmt  | Azure API Management           |
| BFF Layer         | Java + Spring Boot             |
| Microservices     | Spring Boot (REST)             |
| Secrets Mgmt      | Azure Key Vault                |
| Observability     | Prometheus + Grafana           |
| Container Runtime | Docker                         |
| Orchestration     | AKS (Azure Kubernetes Service) |
| CI/CD             | GitHub Actions                 |
| Docs & Testing    | Swagger, Postman               |

---

# 🔐 AZURE KEY VAULT INTEGRATION

## ✅ Step 1: Create Key Vault + Store Secrets

```bash
az keyvault create --name orvyn-kv --resource-group orvyn-bff-rg --location eastus
az keyvault secret set --vault-name orvyn-kv --name "DbPassword" --value "SecurePass123!"
```

---

## ✅ Step 2: Assign Permissions to AKS

```bash
# Enable managed identity on AKS
az aks update --name orvyn-bff-cluster \
  --resource-group orvyn-bff-rg \
  --enable-managed-identity

# Get AKS Managed Identity ID
az aks show --name orvyn-bff-cluster --resource-group orvyn-bff-rg --query "identity.principalId" -o tsv

# Assign access to Key Vault
az keyvault set-policy \
  --name orvyn-kv \
  --object-id <AKS_MANAGED_IDENTITY_ID> \
  --secret-permissions get list
```

---

## ✅ Step 3: Configure Spring Boot (application.yml)

```yaml
spring:
  cloud:
    azure:
      keyvault:
        secret:
          enabled: true
          property-sources:
            - name: orvyn-kv
          endpoint: https://orvyn-kv.vault.azure.net/
```

> 💡 If you stored `DbPassword`, access it in Spring as `${DbPassword}`.

---

# 📦 `application.yml` (Full)

```yaml
spring:
  application:
    name: bff-service

  cloud:
    azure:
      keyvault:
        secret:
          enabled: true
          property-sources:
            - name: orvyn-kv
          endpoint: https://orvyn-kv.vault.azure.net/

management:
  endpoints:
    web:
      exposure:
        include: "*"
  metrics:
    export:
      prometheus:
        enabled: true
  endpoint:
    prometheus:
      enabled: true
```

---

# 📄 `openapi.yaml` (For APIM Import)

```yaml
openapi: 3.0.1
info:
  title: BFF API
  version: v1
servers:
  - url: https://your-apim-url/bff
paths:
  /dashboard:
    get:
      summary: Aggregated user dashboard
      responses:
        '200':
          description: Returns user and order info
          content:
            application/json:
              schema:
                type: object
                properties:
                  user:
                    type: object
                  orders:
                    type: array
                    items:
                      type: object
```

---

# 🤖 GitHub Actions CI/CD (`.github/workflows/bff-ci-cd.yml`)

```yaml
name: BFF CI/CD

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Java
      uses: actions/setup-java@v3
      with:
        java-version: '17'

    - name: Build the BFF app
      run: mvn clean package -f bff/pom.xml

    - name: Build Docker image
      run: docker build -t youracr.azurecr.io/bff:latest ./bff

    - name: Docker login
      run: echo "${{ secrets.ACR_PASSWORD }}" | docker login youracr.azurecr.io -u ${{ secrets.ACR_USERNAME }} --password-stdin

    - name: Push image to ACR
      run: docker push youracr.azurecr.io/bff:latest

    - name: Deploy to AKS
      uses: Azure/k8s-deploy@v1
      with:
        manifests: |
          ./k8s/bff-deployment.yml
        images: |
          youracr.azurecr.io/bff:latest
        kubectl-version: 'latest'
```

---

# 📊 Prometheus + Grafana Setup on AKS

## ✅ Install with Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

kubectl create namespace monitoring

helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.scrapeInterval="10s" \
  --set grafana.adminPassword="admin"
```

---

## ✅ Enable Actuator in Spring Boot (already in `application.yml`)

* Exposes `/actuator/prometheus`
* Metrics will be auto-scraped if `ServiceMonitor` is defined.

---

## ✅ Grafana Setup

* Access via `kubectl get svc -n monitoring`
* Default credentials: `admin/admin`
* Import Dashboard ID: `4701` or your custom JSON

---

# 🔎 Postman Collection (BFF Testing)

```json
{
  "info": {
    "name": "BFF Demo Collection",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "GET /dashboard",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Authorization",
            "value": "Bearer {{access_token}}"
          }
        ],
        "url": {
          "raw": "{{bff_url}}/dashboard",
          "host": ["{{bff_url}}"],
          "path": ["dashboard"]
        }
      }
    }
  ]
}
```

> Set Postman variables:

* `{{access_token}}` → Okta or Azure AD token
* `{{bff_url}}` → APIM endpoint

---

# ✅ FINAL CHECKLIST

| Feature                             | Status |
| ----------------------------------- | ------ |
| Azure API Management (JWT, Routing) | ✅      |
| Java Spring Boot BFF Layer          | ✅      |
| Feign Clients to Microservices      | ✅      |
| Key Vault Secret Access             | ✅      |
| Prometheus + Grafana on AKS         | ✅      |
| CI/CD via GitHub Actions            | ✅      |
| OpenAPI Spec for APIM Import        | ✅      |
| Postman Collection for Testing      | ✅      |

---


