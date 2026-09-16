# Stage 5 — Helm Chart: Scrob + PostgreSQL

## Objective

Package the Scrob application and PostgreSQL Kubernetes resources into a reusable Helm chart.

This stage introduces:

- Helm chart structure
- Configurable values through `values.yaml`
- Templated Deployments, Services, ConfigMap, Secrets, PVCs, and Ingress
- Helm linting and template rendering
- Helm installation and upgrades
- Release history and applied values
- Scaling the Scrob Deployment from one to two replicas

## Directory Structure

```text
stage-5-helm/
└── scrob/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── configmap.yaml
        ├── deployment.yaml
        ├── ingress.yaml
        ├── pvc.yaml
        ├── secret.yaml
        └── service.yaml
```

## Chart Details

- **Chart name:** `scrob`
- **Chart version:** `0.1.0`
- **Application version:** `latest`
- **Namespace used for testing:** `scrob-helm`

The chart deploys:

- Scrob Deployment and ClusterIP Service
- PostgreSQL Deployment and ClusterIP Service
- PostgreSQL ConfigMap
- PostgreSQL and Scrob Secrets
- PostgreSQL and Scrob PersistentVolumeClaims
- Optional Ingress resource

## Validation

Run these commands from the chart directory:

```bash
helm lint .
```

Render the templates locally:

```bash
helm template scrob . \
  --set-string secrets.postgresPassword='test-password' \
  --set-string secrets.scrobSecretKey='test-secret-key' \
  > /tmp/scrob-rendered.yaml
```

Confirm that no Helm template expressions remain:

```bash
grep -n '{{\|}}' /tmp/scrob-rendered.yaml
```

The command should return no matches.

## Installation

Create the namespace:

```bash
kubectl create namespace scrob-helm
```

Install the chart with test credentials and disable Ingress to avoid conflicting with the Stage 4 Ingress:

```bash
helm install scrob . \
  --namespace scrob-helm \
  --set ingress.enabled=false \
  --set-string secrets.postgresPassword='test-password' \
  --set-string secrets.scrobSecretKey='test-secret-key'
```

Check the release:

```bash
helm status scrob -n scrob-helm
helm list -n scrob-helm
```

## Upgrade and Scaling Test

The Scrob Deployment was upgraded from one to two replicas:

```bash
helm upgrade scrob . \
  --namespace scrob-helm \
  --set scrob.replicaCount=2 \
  --set ingress.enabled=false \
  --set-string secrets.postgresPassword='test-password' \
  --set-string secrets.scrobSecretKey='test-secret-key'
```

Verify the rollout:

```bash
kubectl rollout status deployment/scrob-scrob \
  -n scrob-helm \
  --timeout=120s
```

Inspect the Deployment and Pods:

```bash
kubectl get deployment scrob-scrob -n scrob-helm
kubectl get pods \
  -n scrob-helm \
  -l app.kubernetes.io/name=scrob \
  -o wide
```

The test completed successfully with two available Scrob replicas.

## Internal Service Test

The Scrob Service was tested from inside the Kubernetes namespace:

```bash
kubectl run curl-test \
  -n scrob-helm \
  --rm -it \
  --restart=Never \
  --image=curlimages/curl \
  -- curl -I http://scrob-scrob:7330/login
```

The service returned `HTTP/1.1 200 OK`, confirming internal connectivity to Scrob.

## Helm Release History

Review release revisions:

```bash
helm history scrob -n scrob-helm
```

View user-supplied values:

```bash
helm get values scrob -n scrob-helm
```

The tested release reached revision 3 after the scaling upgrade.

## Important Lessons

### 1. Helm upgrades do not automatically restart Pods for every Secret change

Updating a Secret through Helm does not necessarily restart existing Pods. A Deployment checksum annotation or an explicit rollout restart can be used when configuration changes need to trigger a new Pod rollout.

### 2. Database URL format matters

Scrob requires the asynchronous PostgreSQL driver format used in the working deployment:

```text
postgresql+asyncpg://...
```

Using a URL that requires an unavailable driver can cause the application to enter a CrashLoopBackOff state.

### 3. Secrets require careful handling

The values used during this learning exercise are test credentials. Do not commit real credentials to Git. Passing secrets through `--set-string` can expose them in shell history and Helm release values. Production deployments should use a safer secret-management approach.

### 4. Replica scaling needs application and storage consideration

The Scrob Deployment was scaled to two replicas as a Helm learning exercise. The chart currently uses a `ReadWriteOnce` persistent volume, so this configuration should not automatically be treated as production-ready. PostgreSQL remains at one replica.

### 5. Ingress is disabled for this test

The Stage 4 resources remain separate and unchanged. Ingress was disabled in the Helm release to avoid a host or routing conflict with the existing Stage 4 Ingress.

## Troubleshooting Commands

```bash
kubectl get all -n scrob-helm
kubectl describe pod <pod-name> -n scrob-helm
kubectl logs deployment/scrob-scrob -n scrob-helm
kubectl logs deployment/scrob-postgres -n scrob-helm
helm status scrob -n scrob-helm
helm get manifest scrob -n scrob-helm
```

## Cleanup

To remove the Helm release:

```bash
helm uninstall scrob -n scrob-helm
```

The namespace can then be removed if it is no longer needed:

```bash
kubectl delete namespace scrob-helm
```

> Run cleanup commands only when you are certain that the Stage 5 resources are no longer required.

