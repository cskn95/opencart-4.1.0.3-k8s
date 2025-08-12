# OpenCart on Kubernetes — Quickstart (Windows)

This repo includes Kubernetes manifests to run OpenCart with MySQL. Below are minimal, copy‑pasteable steps for Docker Desktop Kubernetes or Minikube on Windows PowerShell.

## Prerequisites

- Windows with PowerShell 5+ (you are here)
- One Kubernetes option:
	- Docker Desktop with Kubernetes enabled, or
	- Minikube (Hyper-V or Docker driver)
- kubectl installed and pointing to your cluster
- Docker (same daemon used by your cluster)

Notes
- The image tag used by the deployment is `opencart-4103-opencart:k8s` (built from `Dockerfile.k8s`).
- MySQL data is ephemeral in this setup (no PVC). Good for local dev. Don’t use in prod.

## 1) Build the application image

In the repo root:

```powershell
docker build -f Dockerfile.k8s -t opencart-4103-opencart:k8s .
```

Minikube users: ensure the image is available inside Minikube. Either build inside its Docker daemon or load the image.

Option A — build inside Minikube’s Docker:
```powershell
minikube -p minikube docker-env | Invoke-Expression
docker build -f Dockerfile.k8s -t opencart-4103-opencart:k8s .
```

Option B — build locally then load into Minikube:
```powershell
docker build -f Dockerfile.k8s -t opencart-4103-opencart:k8s .
minikube image load opencart-4103-opencart:k8s
```

Kind users:
```powershell
docker build -f Dockerfile.k8s -t opencart-4103-opencart:k8s .
kind load docker-image opencart-4103-opencart:k8s
```

## 2) Deploy MySQL and OpenCart

Apply the included manifests from the repo root:

```powershell
kubectl apply -f mysql-deployment.yaml; kubectl apply -f mysql-service.yaml
kubectl apply -f opencart-deployment.yaml; kubectl apply -f opencart-service.yaml
```

Optional: Adminer for DB management

```powershell
kubectl apply -f adminer-deployment.yaml; kubectl apply -f adminer-service.yaml
```

## 3) Wait for pods to be ready

```powershell
kubectl get pods -w
```

You should see `mysql-*` and `opencart-*` go to `Running` (1/1). Then Ctrl+C to stop watching.

## 4) Access the storefront

- The `opencart` Service is NodePort 30080.
	- Docker Desktop Kubernetes: http://localhost:30080/
	- Minikube: get the URL with
		```powershell
		minikube service opencart --url
		```

On first run, the pod auto‑installs OpenCart via CLI using:

- URL: `http://localhost:30080/`
- Admin user: `admin`
- Admin password: `admin`
- DB: `mysql` host, user `root`, password `opencart`, database `opencart`

Change credentials immediately in the admin panel.

## 5) Check logs (if needed)

```powershell
kubectl logs deploy/opencart
kubectl logs deploy/mysql
```

## 6) Adminer (optional)

The `adminer` Service is ClusterIP. Port‑forward to access locally:

```powershell
kubectl port-forward svc/adminer 8080:8080
```

Then open http://localhost:8080/ (Server: `mysql`, User: `root`, Password: `opencart`).

## 7) Cleanup

```powershell
kubectl delete -f opencart-service.yaml; kubectl delete -f opencart-deployment.yaml
kubectl delete -f mysql-service.yaml; kubectl delete -f mysql-deployment.yaml
kubectl delete -f adminer-service.yaml; kubectl delete -f adminer-deployment.yaml
```

## What’s happening under the hood

- `Dockerfile.k8s` builds a PHP 8.2 + Apache image, installs PHP extensions, copies `upload/` to `/var/www/html`, and enables rewrite.
- `opencart-deployment.yaml` starts the container, waits for MySQL, runs the CLI installer if the DB is empty, and then `apache2-foreground`.
- `mysql-deployment.yaml` runs MySQL 5.7 with a default DB `opencart` and root password `opencart`.
- `opencart-service.yaml` exposes the web app as NodePort 30080.

## Notes / Next steps

- Data persistence: Add a PVC to MySQL for persistent storage before using beyond dev.
- Ingress: Instead of NodePort, create an Ingress for a friendly hostname and TLS.
- Secrets: Move DB credentials to a Secret and use envFrom/values (the sample files exist but aren’t wired yet).

