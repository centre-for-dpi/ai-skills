---
name: dpi-cloud
description: Cloud infrastructure for DPI — CDPI cloud-agnostic strategy, sovereign cloud, Kubernetes, IaC, multi-region resilience. Use when making cloud decisions for DPI deployments.
---

# DPI Cloud Infrastructure

## CDPI's Position: Cloud-Agnostic Design

> **Reference**: https://docs.cdpi.dev/technical-notes/cloud-agnostic-deployment-of-dpi-and-data-portability

CDPI's official recommendation is **cloud-agnostic design** for all DPI deployments. This is the strategic default — not multi-cloud for its own sake, but designing so that no single provider can hold DPI hostage.

### The Strategic Case for Cloud Agnosticism

**Sovereign Control, Economic Flexibility, and Risk Mitigation**

Digital Public Infrastructure demands architectural decisions that preserve sovereignty, maintain economic flexibility, and mitigate concentration risk. Cloud-agnostic design ensures that country authorities retain the benefits of cloud speed and flexibility whilst preserving the ability to:

- **Migrate workloads across providers** without wholesale re-architecture, reducing switching costs when better alternatives emerge
- **Negotiate from strength** with cloud service providers, leveraging competitive pricing and avoiding cost escalation from vendor lock-in
- **Maintain operational continuity** independent of any single vendor's business decisions, outages, or security incidents
- **Respond to geopolitical and regulatory shifts** that may affect cloud service availability, without disruption to essential services
- **Adopt hybrid strategies** that optimize cost-performance ratios across providers and on-premises infrastructure
- **Evolve incrementally** — adopt new technologies without forced wholesale migration

### CDPI Architectural Principles for Cloud-Agnostic DPI

> **Reference**: https://docs.cdpi.dev/technical-notes/cloud-agnostic-deployment-of-dpi-and-data-portability/architectural-principles-for-cloud-agnostic-dpi

1. **Container-first deployment**: All DPI components run in containers (Docker/OCI); orchestrated by Kubernetes — portable across any cloud or on-premise
2. **Open-source infrastructure layer**: Prefer open-source databases (PostgreSQL), message buses (Kafka), and service meshes (Istio) over cloud-native proprietary equivalents
3. **Infrastructure as Code**: All infrastructure defined in cloud-neutral IaC (Terraform) with provider-specific backends swappable
4. **Data portability by design**: Data in open formats (PostgreSQL, Parquet, JSON) exportable without vendor tooling
5. **No managed service lock-in for critical paths**: Avoid managed services that have no open-source equivalent for identity-critical or payment-critical components
6. **Abstraction layers**: Use Kubernetes operators and Helm charts to abstract provider differences from application code
7. **Secrets and key management portability**: Use HashiCorp Vault or equivalent that runs on any cloud; avoid cloud-native KMS as the only option for DPI keys

## Cloud Strategy for DPI

DPI systems must balance three cloud imperatives:
1. **Sovereignty**: National data stays under national control
2. **Scale**: Cloud elasticity to handle 1B+ population
3. **Resilience**: No single point of failure

### Cloud Deployment Models for DPI

| Model | Description | Pros | Cons | Use Cases |
|-------|-------------|------|------|-----------|
| **Public Cloud** | AWS, Azure, GCP | Fast, elastic, managed services | Data sovereignty concerns, vendor lock-in | Dev/test, non-sensitive workloads |
| **Government Cloud** | Cloud with government controls | Compliance, sovereignty | Higher cost, less features | Sensitive DPI components |
| **Sovereign Cloud** | Operated in-country under national law | Full sovereignty | Most complex, expensive | Core identity, biometrics |
| **On-Premise** | Government-owned DC | Maximum control | CapEx, maintenance burden | Air-gapped systems |
| **Hybrid** | Mix of above | Balanced | Complex integration | Typical for mature DPI |
| **Community Cloud** | Shared across agencies | Cost sharing | Governance complexity | Multi-ministry DPI |

### Data Residency Requirements for DPI

Typical classification:
```
Biometric Data          → Must remain in-country; on-premise or sovereign cloud
National ID Records     → Must remain in-country; government cloud acceptable
Authentication Tokens   → Can be in public cloud (transit only; no storage)
Payment Transactions    → Central bank governed; in-country required
Health Records          → In-country; regulated by health ministry
Consent Artifacts       → In-country preferred; some flexibility
Analytics/Telemetry     → Can be aggregated/anonymized and stored anywhere
```

## Cloud-Native Architecture for DPI

### Kubernetes as the DPI Platform

Kubernetes (K8s) is the de facto standard for DPI workload orchestration:

```yaml
# Typical DPI namespace structure
namespaces:
  - mosip-kernel        # Shared kernel services
  - mosip-idrepo        # Identity repository
  - mosip-ida           # ID authentication
  - mosip-regproc       # Registration processor
  - mosip-preregistration
  - mosip-resident
  - mosip-esignet
  - mosip-inji          # Inji wallet backend
  - kafka               # Message bus
  - postgres            # Databases
  - keycloak            # IAM
  - nginx               # Ingress
  - monitoring          # Prometheus/Grafana
  - logging             # ELK/Loki
```

### Helm Charts for DPI Deployment
```bash
# MOSIP deployment pattern
helm repo add mosip https://mosip.github.io/mosip-helm

# Deploy core infrastructure
helm install postgres mosip/postgres -n postgres \
  -f postgres-values.yaml

# Deploy MOSIP modules
helm install kernel mosip/kernel -n mosip-kernel \
  -f kernel-values.yaml

helm install idrepo mosip/id-repository -n mosip-idrepo \
  -f idrepo-values.yaml
```

### Container Architecture Patterns

**Sidecar pattern** (service mesh):
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: id-auth-service    # Main container
        image: mosip/id-auth:1.2
      - name: istio-proxy        # Sidecar (injected by Istio)
        image: istio/proxyv2:1.18
        # Handles: mTLS, tracing, metrics, traffic management
```

**Init container pattern** (DB migration):
```yaml
initContainers:
- name: db-migration
  image: mosip/liquibase:latest
  env:
  - name: DB_URL
    valueFrom:
      secretKeyRef: { name: db-secret, key: url }
  command: ["liquibase", "update"]
```

## Cloud Provider Options for DPI

### India
- **NIC Cloud (Meghraj)**: Government cloud; compliant for sensitive data
- **AWS India (Mumbai/Hyderabad)**: AWS with DPA with GoI
- **Azure India (Pune/Chennai)**: Microsoft with government SLAs
- **GCP India**: Google Cloud Mumbai/Delhi
- **ESDS**: Indian managed cloud for government
- **Tata Communications**: Sovereign cloud services

### Philippines
- **GovCloud PH**: DICT-operated government cloud
- **AWS Manila**: Available (no local data center as of 2024)
- **Azure Philippines**: Available via Singapore region

### Africa
- **AWS Africa (Cape Town)**: Covers Southern Africa
- **Azure South Africa (Jo'burg/Cape Town)**
- **GCP Johannesburg**
- **Huawei Cloud Africa**: Active in multiple African countries
- **Liquid Intelligent Technologies**: Pan-African cloud

### Key Selection Criteria
1. Data center in-country or in-region?
2. Government compliance certifications (ISO 27001, SOC 2, FedRAMP equivalent)
3. Air-gap / disconnected operations capability?
4. Support for HSM (Hardware Security Modules)?
5. Managed Kubernetes available?
6. Local support and SLA commitments?
7. Cost in local currency?
8. Vendor neutrality (avoid lock-in)?

## Infrastructure as Code (IaC) for DPI

### Terraform Patterns
```hcl
# DPI infrastructure module structure
modules/
  networking/       # VPC, subnets, security groups
  kubernetes/       # EKS/GKE/AKS cluster
  database/         # RDS/Cloud SQL/PostgreSQL
  storage/          # S3/GCS/Azure Blob
  hsm/              # CloudHSM / Key Vault
  monitoring/       # Prometheus, Grafana, alerting
  ingress/          # ALB/NLB/Nginx configuration

# Example: DPI Kubernetes cluster
module "dpi_cluster" {
  source  = "./modules/kubernetes"
  
  cluster_name    = "dpi-prod-${var.country_code}"
  region          = var.region
  node_pools = {
    general = {
      machine_type = "n2-standard-8"
      min_count    = 3
      max_count    = 20
    }
    biometric = {
      machine_type = "n2-standard-16"  # Higher compute for ABIS
      min_count    = 2
      max_count    = 10
    }
  }
  private_cluster = true
  enable_workload_identity = true
}
```

### GitOps with ArgoCD
```yaml
# ArgoCD Application for MOSIP ID Auth
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mosip-ida
  namespace: argocd
spec:
  project: mosip
  source:
    repoURL: https://github.com/country/mosip-config
    targetRevision: HEAD
    path: charts/id-authentication
  destination:
    server: https://kubernetes.default.svc
    namespace: mosip-ida
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

## Cloud Security for DPI

### Network Security Architecture
```
Internet
    ↓
CloudFlare / Akamai (DDoS + CDN)
    ↓
Cloud Load Balancer (TLS termination)
    ↓
WAF (Web Application Firewall)
    ↓
Ingress Controller (Nginx / Istio Gateway)
    ↓
Service Mesh (Istio) — mTLS everywhere
    ↓
Application Pods (No direct internet access)
    ↓
Private Database Subnet (No public IPs)
```

### Identity and Access Management (Cloud IAM)
```
Principles:
- Least privilege: Every service has minimum needed permissions
- Workload identity: Pods use cloud service accounts (not static keys)
- Short-lived credentials: Rotate automatically
- Audit everything: CloudTrail/Cloud Audit Logs for all API calls

AWS example:
- EKS IRSA (IAM Roles for Service Accounts)
- Each MOSIP service has its own IAM role
- Roles have scoped permissions (e.g., specific S3 bucket)

GCP example:
- Workload Identity Federation
- Each K8s service account mapped to GCP SA
- GCP SA has IAM bindings to specific resources
```

### Secrets Management
```yaml
# External Secrets Operator pattern (K8s → Vault/AWS Secrets Manager)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: mosip-idrepo
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: db-credentials
    creationPolicy: Owner
  data:
  - secretKey: DB_PASSWORD
    remoteRef:
      key: mosip/idrepo/db
      property: password
```

### Container Security
```dockerfile
# DPI container security practices
FROM eclipse-temurin:17-jre-alpine  # Minimal base image
RUN addgroup -S mosip && adduser -S mosip -G mosip  # Non-root user
COPY --chown=mosip:mosip target/app.jar /app.jar
USER mosip
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
# Note: No secrets in Dockerfile; injected at runtime
```

## Observability Stack for DPI

### Metrics, Logs, Traces (Golden Triangle)

```yaml
# Prometheus scrape config for MOSIP
scrape_configs:
  - job_name: 'mosip-ida'
    kubernetes_sd_configs:
    - role: pod
      namespaces:
        names: ['mosip-ida']
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
```

### Key DPI SLIs and Alerts
```yaml
# SLI: Authentication success rate > 99.5%
alert: AuthSuccessRatelow
expr: rate(auth_requests_total{status="success"}[5m]) /
      rate(auth_requests_total[5m]) < 0.995
for: 2m
severity: critical

# SLI: Authentication P99 latency < 500ms  
alert: AuthLatencyHigh
expr: histogram_quantile(0.99, rate(auth_duration_seconds_bucket[5m])) > 0.5
for: 5m
severity: warning
```

### Distributed Tracing
```java
// OpenTelemetry in MOSIP services
@Traced
public AuthResponse authenticate(AuthRequest request) {
    Span span = tracer.spanBuilder("id-authentication")
        .setAttribute("txn_id", request.getTransactionId())
        .setAttribute("auth_type", request.getAuthType())
        .startSpan();
    // ... auth logic ...
    span.end();
}
```

## Multi-Cloud and Hybrid Architecture

### Active-Active Multi-Region
```
Region 1 (Primary)          Region 2 (Secondary)
────────────────            ────────────────────
K8s Cluster                 K8s Cluster
PostgreSQL Primary    ←—→   PostgreSQL Replica
Kafka Cluster         ←—→   Kafka MirrorMaker2
Vault Active          ←—→   Vault Standby
            ↕                      ↕
       Global Load Balancer (GeoDNS / Traffic Manager)
            ↕
         Internet Users
```

### Disaster Recovery Runbook Pattern
```bash
#!/bin/bash
# Failover script: promote secondary region
# Step 1: Promote PostgreSQL replica
kubectl exec -n postgres postgres-0 -- psql -c "SELECT pg_promote();"

# Step 2: Update application config to point to new DB
kubectl set env deployment/id-auth DB_HOST=postgres-secondary.region2

# Step 3: Update DNS/traffic manager
# ... cloud-specific commands ...

# Step 4: Verify health
kubectl rollout status deployment/id-auth -n mosip-ida
```

## Cost Optimization

### DPI Cost Patterns
| Component | Cost Pattern | Optimization |
|-----------|-------------|-------------|
| Authentication API | Bursty (business hours) | K8s HPA autoscaling |
| Enrollment | Batch workloads | Spot/preemptible instances |
| Biometric ABIS | Compute-intensive | GPU spot instances + queue |
| Analytics | Infrequent, large | Serverless (Lambda/Cloud Functions) |
| Storage (ID records) | Grows continuously | Tiered storage (hot/warm/cold) |

### Resource Requests & Limits
```yaml
resources:
  requests:
    cpu: 500m       # Guaranteed CPU
    memory: 1Gi     # Guaranteed memory
  limits:
    cpu: 2000m      # Max CPU (prevents noisy neighbor)
    memory: 4Gi     # Max memory (OOM kills pod if exceeded)
```

## Cloud Governance for National DPI

### Shared Responsibility Matrix
| Responsibility | Country | Cloud Provider |
|---------------|---------|---------------|
| Physical security | ✓ (sovereign cloud) / shared | ✓ (public cloud) |
| Network security | Shared | Shared |
| OS patching | Country (IaaS) / Provider (PaaS) | Provider (PaaS) |
| Data classification | Country | n/a |
| Access controls | Country | Shared |
| Data encryption keys | **Country** | Provider provides tooling |
| Compliance | **Country** | Provider certifications |

### Exit Strategy (Avoid Lock-in)
- Use open-source databases (PostgreSQL, not RDS Aurora)
- Kubernetes workloads are portable (avoid cloud-specific resources)
- Store data in open formats (Parquet, JSON, not proprietary)
- Document architecture to enable re-deployment elsewhere
- Regularly test migration procedures

### Procurement Considerations
- **BOOT model** (Build-Operate-Own-Transfer): Common for national DPI
- Cloud provider must sign DPA (Data Processing Agreement) with government
- Escrow of source code for critical components
- Exit clauses and data portability guarantees in contract
