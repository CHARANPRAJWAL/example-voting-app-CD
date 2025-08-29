```markdown
# example-voting-app-CD

Kubernetes manifests for the classic **Docker Samples: Voting App**—structured for **continuous delivery (CD/GitOps)**.  
Track this repo with Argo CD or Flux, or apply the manifests directly with `kubectl`.

## What’s inside

```

.
├── db-manifest.yaml        # Postgres database (Deployment/StatefulSet + Service + PVC)
├── redis-manifest.yaml     # Redis queue (Deployment + Service)
├── vote-manifest.yaml      # Vote web app (Python/Flask) – Service (ClusterIP/LoadBalancer)
├── result-manifest.yaml    # Result web app (Node.js) – Service (ClusterIP/LoadBalancer)
└── worker-manifest.yaml    # Background worker (DotNet) – Deployment

```

> Images are pinned by tag or digest for repeatable, audit-friendly CD.  
> Update images by changing the tag/digest in the manifest and pushing to `main`—your GitOps controller will reconcile.

## Architecture

- **vote** → writes votes to **Redis**
- **worker** → drains Redis and persists to **Postgres**
- **result** → reads tallies from Postgres and renders UI

```

\[ vote ] --(enqueue)--> \[ redis ] --(worker)--> \[ postgres ] <--(read)-- \[ result ]

````

## Prerequisites

- Kubernetes 1.23+
- `kubectl` configured for your cluster
- (Optional) **Argo CD** or **Flux** for GitOps
- A namespace to deploy into (default below: `voting-app`)

## Quick start (kubectl)

```bash
# 1) Create namespace
kubectl create namespace voting-app

# 2) Apply all components
kubectl apply -n voting-app -f db-manifest.yaml
kubectl apply -n voting-app -f redis-manifest.yaml
kubectl apply -n voting-app -f worker-manifest.yaml
kubectl apply -n voting-app -f vote-manifest.yaml
kubectl apply -n voting-app -f result-manifest.yaml

# 3) Check status
kubectl get pods,svc -n voting-app
````

### Access the apps

If Services are `ClusterIP`, port-forward locally:

```bash
# vote frontend
kubectl port-forward -n voting-app svc/vote 8080:80
# result frontend
kubectl port-forward -n voting-app svc/result 8081:80
```

* Vote UI: [http://localhost:8080](http://localhost:8080)
* Result UI: [http://localhost:8081](http://localhost:8081)

> If Services are `LoadBalancer`, use `kubectl get svc -n voting-app` and open the external IPs.

## GitOps with Argo CD (recommended)

Point an Argo CD `Application` at this repo path:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: voting-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<your-org>/example-voting-app-CD.git
    targetRevision: main
    path: .
  destination:
    namespace: voting-app
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Commit changes to this repo; Argo CD will sync them to the cluster automatically.

## Operations

### Update images (CD flow)

1. Edit the relevant manifest (e.g., `vote-manifest.yaml`) and change the `image:` to a new **tag** or **digest**.
2. Commit to `main`.
3. Argo CD/Flux reconciles and rolls out the change.

### Scale components

```bash
kubectl -n voting-app scale deploy/vote --replicas=3
kubectl -n voting-app scale deploy/result --replicas=2
kubectl -n voting-app scale deploy/worker --replicas=2
```

### Environment/config

* Postgres and Redis connection strings are wired via Service DNS names (e.g., `redis`, `db`) inside the namespace.
* If you need persistence for Postgres, ensure the `db-manifest.yaml` uses a `PersistentVolumeClaim` compatible with your cluster’s storage class.

## Verification

```bash
# Pods healthy?
kubectl get pods -n voting-app

# Services reachable?
kubectl get svc -n voting-app

# Worker logs moving votes from Redis to Postgres?
kubectl logs -n voting-app deploy/worker
```

## Cleanup

```bash
kubectl delete namespace voting-app
```

## Notes / Troubleshooting

* **Image pull errors:** ensure the images are public or configure an `imagePullSecret`.
* **CrashLoopBackOff on db:** your cluster must support the PVC/storageClass referenced in `db-manifest.yaml`.
* **Service type:** switch between `ClusterIP` and `LoadBalancer` in `vote/result` Services to suit your environment.
* **Ingress:** if you use an Ingress controller, add an `Ingress` resource and hostnames rather than port-forwarding.

---

### License

This repo only contains Kubernetes manifests. The underlying application is from the Docker Samples Voting App (MIT-licensed).

```

