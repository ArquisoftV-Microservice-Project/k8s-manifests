# k8s-manifests

# 🚀 Kubernetes Manifests for Microservices Platform

This repository contains the Kubernetes manifests for deploying a microservices-based platform. It is structured to separate apps by namespace and includes infrastructure components like monitoring and ingress.

---

## 📁 Project Structure

```

.
├── manifests/
│ ├── apps/ # All application manifests grouped by service
│ │ ├── auth/ # Auth API deployment and service
│ │ ├── frontend/ # Frontend app deployment and service
│ │ ├── grafana/ # Grafana deployment, service, config, and storage
│ │ ├── log-message-processor/
│ │ ├── prometheus/ # Prometheus config, deployment, and service
│ │ ├── redis/ # Redis deployment and service
│ │ ├── todos/ # Todos API deployment and service
│ │ ├── users/ # Users API deployment and service
│ │ └── zipkin/ # Zipkin deployment and service
│ └── ingress/ # Ingress rules for routing
├── LICENSE
└── README.md

```

Each service has its own directory with logically split manifests (`deployment.yaml`, `service.yaml`, `configmap.yaml`, etc.), promoting maintainability and clarity.

---

## 📄 Kubernetes Manifests Structure — Resource Explanation

This repository contains the Kubernetes manifests for deploying a microservices-based application, divided by component. Each application or tool (like `frontend`, `auth`, `prometheus`, etc.) includes only the necessary Kubernetes resources for its specific functionality. Here's a breakdown:

---

### 📦 Common Kubernetes Resources

| Resource     | Description                                                                                |
| ------------ | ------------------------------------------------------------------------------------------ |
| `Deployment` | Defines the desired state for the application pods (e.g., replicas, image, env vars, etc). |
| `Service`    | Exposes the Deployment internally (ClusterIP) or externally (LoadBalancer, NodePort, etc). |
| `ConfigMap`  | Used to inject configuration into pods (non-sensitive data).                               |
| `Datasource` | Specifically used by Grafana to define data sources like Prometheus for dashboards.        |

---

### 🧩 Application Breakdown

### ✅ `auth`, `frontend`, `todos`, `users`, `zipkin`

- **Deployment**: Yes — Each of these is a running microservice that requires a pod.
- **Service**: Yes — Each needs to be discoverable within the cluster (DNS) and/or accessed via Ingress.
- **ConfigMap**: ❌ — These services do not currently require external configuration via ConfigMaps.
- **Datasource**: ❌ — Not applicable (they are not monitoring tools like Grafana).

---

### ✅ `redis`

- **Deployment**: Yes — Runs Redis as a pod.
- **Service**: Yes — Other services access Redis through this internal service.
- **ConfigMap**: ❌ — Redis uses default configuration and doesn't need external config.
- **Datasource**: ❌ — Redis isn't visualized in Grafana.

---

### ✅ `log-message-processor`

- **Deployment**: Yes — Runs a background job or processor.
- **Service**: ❌ — It does not expose an API or need to be reached by other services directly.
- **ConfigMap**: ❌ — No configuration required externally.
- **Datasource**: ❌ — Not used for metrics.

---

### ✅ `prometheus`

- **Deployment**: Yes — Runs the Prometheus monitoring service.
- **Service**: Yes — Grafana and other tools may query Prometheus.
- **ConfigMap**: ✅ — Prometheus requires a `ConfigMap` for its scraping and alerting configuration.
- **Datasource**: ❌ — Datasource configuration is handled inside Grafana.

---

### ✅ `grafana`

- **Deployment**: Yes — Runs the Grafana dashboard service.
- **Service**: Yes — Used to access Grafana’s UI.
- **ConfigMap**: ✅ — Sets up default dashboards or UI settings.
- **Datasource**: ✅ — Contains data source config (e.g., to connect to Prometheus).

---

### 🌐 `ingress`

- **Manifest Type**: Ingress rule
- **Purpose**: Defines HTTP routing (e.g., which path goes to which service). Exposes services like `frontend`, `grafana`, etc. outside the cluster via a single entry point.

---

## ⚙️ GitHub Actions: CI/CD Workflow

This repo includes an automated deployment pipeline defined in `.github/workflows/deploy.yml`.

### 🧩 Workflow Name: `Deploy Manifests`

It runs automatically on:

- Push to the `main` branch.
- Manual trigger via the GitHub UI (`workflow_dispatch`).

### 🔄 What it does:

1. **Checkout Code**
   Pulls the contents of the repository.

2. **Authenticate with GCP**
   Uses a GitHub secret (`GCP_SA_CREDENTIALS`) to log in with a service account.

3. **Setup Google Cloud SDK**
   Installs and configures `gcloud` with the target project.

4. **Fetch GKE Cluster Credentials**
   Connects to your Kubernetes cluster using `kubectl` by authenticating via GKE.

5. **Apply All Manifests**
   Recursively applies every YAML file inside the `manifests/` folder.

6. **Log Pods in Namespaces**
   Lists the pods in relevant namespaces (`frontend`, `backend`, `database`, `monitoring`) to show the current status post-deployment.

---

## 🔐 Required GitHub Secrets and Vars

### Secrets:

- `GCP_SA_CREDENTIALS`: JSON key for a Google Cloud service account with Kubernetes and Artifact Registry access.
- `GCP_PROJECT`: Google Cloud project ID.
- `GKE_CLUSTER`: Name of your GKE cluster.

### Variables:

- `GCP_ZONE`: Zone where your GKE cluster is running (e.g., `us-east5-c`).

---

## ✅ Usage

To deploy:

1. Commit your changes to `main` through a PR due to branch protection, or
2. Manually trigger the workflow from the **Actions** tab in GitHub.

Manifests will be automatically applied to the GKE cluster.

---

## 🌐 Ingress

The `manifests/ingress/ingress.yml` file defines the ingress rules for routing HTTP traffic to the appropriate frontend service based on the URL path. It assumes an Ingress controller is installed in the cluster, see the infrastructure repo where using terraform the ingress nginx controller is installed.

---

## 📊 Monitoring Stack

This repo includes manifests for:

- **Prometheus** – Metrics collection.
- **Grafana** – Metrics visualization, with preconfigured datasource via ConfigMap.
- **Zipkin** – Distributed tracing.

---

## 🧹 Notes

- Namespace conventions: services are assumed to run in logical namespaces (`frontend`, `backend`, etc.).
- Ensure amespaces exist before applying the manifests.
- It should be applied after the [Terra-Infrastructure](https://github.com/ArquisoftV-Microservice-Project/Terra-Infrastructure) is running
