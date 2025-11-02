# Postgres + pgAdmin Kubernetes Manifests

This repository contains Kubernetes manifests to run a PostgreSQL database and pgAdmin (web UI) locally on a Kubernetes cluster (Docker Desktop, minikube, kind, etc.). The manifests provide deployments, services, a PersistentVolume/Claim for storage, and an Ingress resource for HTTP access.

## Files
- `postgres-deploy.yaml`  — Deployment, Service, PV and PVC for PostgreSQL.
- `pgadmin-deploy.yaml`   — Deployment, Service, PVC, and Ingress for pgAdmin.
- `postgres-secrets.yaml` — (ignored) Example/expected secret for Postgres credentials (not committed).
- `pgadmin-secrets.yaml`  — (ignored) Secret for pgAdmin default password (not committed).

## Prerequisites
- kubectl configured to talk to your cluster (Docker Desktop Kubernetes, minikube, etc.)
- Docker Desktop with Kubernetes enabled or another local k8s provider
- (Optional) an Ingress controller (e.g., ingress-nginx) if you want to use the provided Ingress

## Quick start (local)
1. Create secrets (these files are gitignored — create them locally or create secrets from the command line). Example commands:

```bash
kubectl create secret generic postgres-secret \
  --from-literal=postgres-root-username=postgres \
  --from-literal=postgres-root-password='example-password' 

kubectl create secret generic pgadmin-secret \
  --from-literal=pgadmin-default-password='pgadmin-password'
```

2. Apply the manifests

```bash
kubectl apply -f postgres-deploy.yaml
kubectl apply -f pgadmin-deploy.yaml
```

3. Check resources

```bash
kubectl get all -n default
```

Accessing pgAdmin
- NodePort (should work on Docker Desktop): open http://localhost:30200

- Port-forward (reliable regardless of cluster networking):

```bash
kubectl port-forward svc/pgadmin 8080:80 -n default
# then open http://localhost:8080
```

- Ingress: If you prefer to use Ingress (the manifest uses `spec.ingressClassName: "nginx"`) you must have an ingress controller running. Example to install ingress-nginx:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

Then access via http://localhost (if your Ingress host is `localhost`). Make sure the ingress controller's class name matches `ingressClassName`.


## Useful commands summary

```bash
# apply manifests
kubectl apply -f postgres-deploy.yaml
kubectl apply -f pgadmin-deploy.yaml

# view pods/services/pvc
kubectl get pods,svc -n default
kubectl get pvc,pv
kubectl describe pod -l app=pgadmin -n default
kubectl logs -l app=pgadmin -n default --tail=200

# restart deployment
kubectl rollout restart deployment/pgadmin -n default

# port-forward
kubectl port-forward svc/pgadmin 8080:80 -n default

# inspect PV
kubectl get pv postgres-pv -o yaml
kubectl get pv pgadmin-pv -o yaml

# cleanup
kubectl delete -f pgadmin-deploy.yaml
kubectl delete -f postgres-deploy.yaml
kubectl delete secret pgadmin-secret postgres-secret || true

# kill port-forwards 
pgrep -af 'kubectl port-forward'
ps aux | grep '[k]ubectl port-forward'
# kill by PID
kill <PID>
# or kill all port-forwards (careful)
pkill -f 'kubectl port-forward'
```

## Reference

- https://blog.devgenius.io/how-to-deploy-postgresql-db-server-and-pgadmin-in-kubernetes-a-how-to-guide-57952b4e29a8
