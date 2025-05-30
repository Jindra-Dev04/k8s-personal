# k8s-personal

Personal Kubernetes infrastructure configuration using GitOps principles with Argo CD.

## Overview

This repository contains configurations for a personal Kubernetes cluster, organized with a GitOps approach using Argo CD. It includes both Helm charts and Kustomize configurations for various applications and services.

## Repository Structure

```
├── bootstrap/                 # Argo CD ApplicationSet definitions
│   ├── helm.yaml              # ApplicationSet for Helm-based applications
│   └── kustomize.yaml         # ApplicationSet for Kustomize-based applications
├── helm/                      # Helm chart configurations
│   ├── create/                # Create application
│   ├── factorio/              # Factorio game server
│   ├── kostky/                # Kostky application with ingress configuration
│   ├── majnr/                 # Majnr application
│   ├── prometheus/            # Prometheus monitoring stack
│   ├── reflector/             # Reflector for K8s resource mirroring
│   └── traefik/               # Traefik ingress controller
└── kustomize/                 # Kustomize configurations
    ├── argocd/                # Argo CD configuration
    └── cloudflared/           # Cloudflare Tunnel configuration
```

## Bootstrap

The repository uses Argo CD ApplicationSets to manage applications:

- **Helm Applications**: Configured in `bootstrap/helm.yaml`, automatically detects and deploys applications in the `helm/` directory based on their `config.json` files.
- **Kustomize Applications**: Configured in `bootstrap/kustomize.yaml`, automatically detects and deploys applications in the `kustomize/` directory.

## Applications

### Helm Charts

1. **Traefik**: Ingress controller deployed in the `kube-system` namespace.
2. **Prometheus**: Monitoring stack with Prometheus, Alertmanager, and Grafana.
3. **Reflector**: Utility for mirroring Kubernetes resources across namespaces.
4. **Factorio**: Game server with custom ingress configuration.
5. **Create**: Custom application with ingress configuration.
6. **Majnr**: Custom application.

### Kustomize

1. **Argo CD**: GitOps continuous delivery tool configured with custom settings.
2. **Cloudflared**: Cloudflare Tunnel for secure external access to cluster services.

#### Cloudflare Tunnel

The repository includes a Cloudflared deployment to create secure tunnels from the internet to services running within the Kubernetes cluster, without requiring public IPs or opening firewall ports.

Key features:
- Deployed in a dedicated `cloudflared` namespace
- Configured with high availability using 2 replicas
- Uses a Cloudflare Tunnel token stored as a Kubernetes Secret
- Includes health checks via readiness and liveness probes
- Exposes metrics on port 2000

To use Cloudflare Tunnels:
1. Create a tunnel in the Cloudflare Zero Trust dashboard
2. Generate a tunnel token and replace the value in `kustomize/cloudflared/secrets.yaml`
3. Deploy the cloudflared service via Argo CD
4. Create DNS records in Cloudflare that point to your tunnel

## Getting Started

### Prerequisites

- Kubernetes cluster
- `kubectl` configured to communicate with your cluster
- `helm` (v3+)
- `kustomize`

### Bootstrapping the Cluster with Argo CD

1. **Clone this repository**:
   ```bash
   git clone https://github.com/Jindra-Dev04/k8s-personal.git
   cd k8s-personal
   ```

2. **Bootstrap Argo CD to your cluster**:
   ```bash
   # Create the argocd namespace
   kubectl create namespace argocd
   
   # Apply the Argo CD kustomization
   kubectl apply -k kustomize/argocd
   
   # Wait for all Argo CD pods to be ready
   kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
   
   # Apply the Kustomize and Helm ApplicationSets
   kubectl apply -f bootstrap/kustomize.yaml
   kubectl apply -f bootstrap/helm.yaml
   ```

After applying these commands, Argo CD will automatically start creating and syncing all applications defined in this repository.

### Bootstrapping Argo CD

Follow these steps to completely bootstrap your Argo CD installation:

1. **Access the Argo CD UI**:
   
   If using Ingress (configured in `kustomize/argocd/ingress.yaml`):
   ```bash
   # Get the ingress host
   kubectl get ingress -n argocd
   ```
   
   Alternatively, use port-forwarding:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   
   Then access Argo CD at: https://localhost:8080

2. **Get the initial admin password**:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```


The ApplicationSets will automatically detect and deploy all the Helm and Kustomize applications in your repository.

## Maintenance

- Add new Helm applications by creating a new directory under `helm/` with `Chart.yaml`, `config.json`, and `values.yaml`.
- Add new Kustomize applications by creating a new directory under `kustomize/` with a `kustomization.yaml` file.

## License

This project is for personal use.
