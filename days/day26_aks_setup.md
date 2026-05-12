# Day 25 — Azure Kubernetes Service (AKS) — Setup & Core Concepts

> Phase 7 — Containers + AKS

## Goal

Deploy a managed Kubernetes cluster on Azure and run a real application on it.

## Key Topics

- Kubernetes recap (brief): pods are the smallest deployable unit (one or more containers); Deployments manage replica sets; Services expose pods to network traffic; Namespaces isolate resources within a cluster
- AKS: Azure's fully managed Kubernetes service — Microsoft manages the control plane (API server, etcd, scheduler) at no charge; you pay only for the worker nodes (VMs)
- AKS architecture:
  - **Control Plane**: managed by Microsoft — you never see or manage these machines; free
  - **Node Pool**: a group of VMs that run your workloads — you choose the VM size and count
  - **System Node Pool**: runs Kubernetes system pods (CoreDNS, metrics-server); required
  - **User Node Pool**: runs your application pods; optional additional pools
- Choosing node VM sizes: use Standard_D2s_v3 (2 vCPU, 8GB RAM) as a starting point for most workloads
- AKS networking modes:
  - **Kubenet**: simpler, nodes get VNet IPs but pods get their own internal range — NAT applied
  - **Azure CNI**: every pod gets a VNet IP directly — required for fine-grained NSG policies on pods
- AKS authentication: integrate with Entra ID so your team's Azure identities control who can run `kubectl` commands — Kubernetes RBAC enforced by Entra ID group membership
- Connecting to AKS: `az aks get-credentials --resource-group myRG --name myCluster` — merges the cluster kubeconfig into your local `~/.kube/config`
- AKS add-ons:
  - Azure Monitor for containers: scrape metrics and logs from pods into Log Analytics
  - Application Gateway Ingress Controller (AGIC): use Azure Application Gateway as the ingress for your cluster

## Hands-On Demo

**Account Requirements:** AKS control plane is free; worker node VMs are paid. Running 1–2 nodes of Standard_B2s (~$30–60/month) — instructor demo; students can use free trial credits to follow along.

- 💳 Create an AKS cluster via Azure Portal (1 system node pool, Standard_B2s)
- 💳 Run `az aks get-credentials` and verify connection with `kubectl get nodes`
- 💳 Deploy a 2-tier sample app (frontend + backend) using YAML manifests (`kubectl apply -f`)
- 💳 Expose the frontend via a LoadBalancer Service — wait for the public IP and test in a browser

## Summary

AKS removes the hardest part of running Kubernetes — managing the control plane. You pay only for your worker nodes and get Microsoft's SLA on the API server. Entra ID integration means your team's Azure identities and RBAC apply directly to Kubernetes — no separate user management.
