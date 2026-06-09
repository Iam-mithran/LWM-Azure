# Day 26 — AKS Advanced — Scaling, Ingress & CI/CD Integration

> Phase 7 — Containers + AKS

## Goal

Scale AKS workloads automatically and wire up a full Azure DevOps CI/CD pipeline that deploys to your cluster.

## Key Topics

- Horizontal Pod Autoscaler (HPA): automatically adjusts the number of pod replicas based on CPU or memory utilization — defined in a Kubernetes manifest, works the same on AKS as any Kubernetes cluster
- Cluster Autoscaler: automatically adds or removes nodes from a node pool based on pending pods — if pods can't be scheduled because nodes are full, a new node is added; idle nodes are removed to save cost
- KEDA (Kubernetes Event-Driven Autoscaling): scale pods based on external event sources — queue length, HTTP request rate, Cosmos DB throughput, Event Hub lag, and more — goes beyond CPU/memory
- Ingress controllers: a reverse proxy that routes external HTTP/HTTPS traffic to the right Service inside your cluster based on rules (host name, URL path)
  - **NGINX Ingress**: open-source, widely used, runs inside the cluster
  - **Application Gateway Ingress Controller (AGIC)**: uses Azure Application Gateway as the ingress — integrates with Azure WAF and SSL termination
- Full AKS CI/CD with Azure DevOps:
  1. Developer pushes code → triggers pipeline
  2. Pipeline builds Docker image and pushes to ACR
  3. Pipeline runs `kubectl apply` (or Helm upgrade) to update the Deployment in AKS
  4. Kubernetes performs a rolling update — zero downtime
- Helm charts basics: Helm is the package manager for Kubernetes — a Chart bundles your Kubernetes YAML manifests into a versioned, configurable package; deploy with `helm install`, upgrade with `helm upgrade`
- Persistent storage in AKS:
  - **Azure Disk**: block storage mounted to a single pod (ReadWriteOnce) — good for databases
  - **Azure Files**: SMB/NFS share mounted to multiple pods simultaneously (ReadWriteMany) — good for shared content
- AKS security:
  - Network policies: control which pods can talk to which other pods (Kubernetes-native firewall)
  - Workload Identity: replace pod-level credentials with Managed Identity — pods authenticate to Azure services (Key Vault, Storage) without any secrets

## Hands-On Demo

**Account Requirements:** All steps require a running AKS cluster (paid — continuation from Day 25). Instructor demo throughout; students follow along with free trial credits.

- 💳 Enable Cluster Autoscaler on the AKS node pool (min 1, max 3 nodes)
- 💳 Apply an HPA manifest for the sample app (scale when CPU > 50%)
- 💳 Deploy NGINX Ingress Controller via Helm and create an Ingress rule
- 💳 Build the full CI/CD pipeline: Azure DevOps → build image → push to ACR → `kubectl apply` → rolling update in AKS

## Summary

HPA scales your pods; Cluster Autoscaler scales your nodes — together they let your app handle variable load without manual intervention. NGINX Ingress is the entry point for all HTTP traffic into your cluster. A complete CI/CD pipeline from code commit to Kubernetes deployment is the gold standard for any container-based application.
