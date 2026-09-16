Stage 4 — Kubernetes Basics: App + PostgreSQL (Ingress)

Goal

Build the first complete Kubernetes deployment of the Scrob application and PostgreSQL database using Kubernetes Services, persistent storage, Secrets, ConfigMaps and an NGINX Ingress controller.

This stage focuses on understanding how a multi-container application is deployed and connected inside Kubernetes.

Architecture

                         Browser / curl
                              |
                              | http://scrob.local
                              v
                    +----------------------+
                    |   NGINX Ingress      |
                    |   scrob.local        |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |   Scrob Service      |
                    |      ClusterIP       |
                    |      Port 7330       |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |      Scrob Pod       |
                    |      Port 7330       |
                    +----------+-----------+
                               |
                               | DATABASE_URL
                               v
                    +----------------------+
                    | PostgreSQL Service   |
                    |      ClusterIP       |
                    |      Port 5432       |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |   PostgreSQL Pod     |
                    |      Port 5432       |
                    +----------+-----------+
                               |
                 +-------------+-------------+
                 |                           |
                 v                           v
          +-------------+             +-------------+
          | postgres-pvc|             |  scrob-pvc  |
          |    5 GiB    |             |    5 GiB    |
          +-------------+             +-------------+

Kubernetes Resources

Stage 4 contains:

Resource

Purpose

PostgreSQL Deployment

Runs PostgreSQL

PostgreSQL Service

Provides internal database access

PostgreSQL ConfigMap

Stores non-sensitive database configuration

PostgreSQL Secret

Stores the database password

PostgreSQL PVC

Provides persistent PostgreSQL storage

Scrob Deployment

Runs the Scrob application

Scrob Service

Provides internal access to Scrob

Scrob PVC

Provides persistent Scrob application storage

Ingress

Exposes Scrob through scrob.local

Project Structure

stage-4-kubernetes-improvements/
├── README.md
├── postgres-configmap.yaml
├── postgres-secret.yaml
├── postgres-pvc.yaml
├── postgres-deployment.yaml
├── postgres-service.yaml
├── scrob-pvc.yaml
├── scrob-deployment.yaml
├── scrob-service.yaml
└── ingress.yaml

1. PostgreSQL ConfigMap

File: postgres-configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
data:
  POSTGRES_DB: scrob
  POSTGRES_USER: scrob

The ConfigMap stores non-sensitive PostgreSQL configuration.

2. PostgreSQL Secret

File: postgres-secret.yaml

The repository contains only a safe template. Real credentials must never be committed to Git.

apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
stringData:
  POSTGRES_PASSWORD: CHANGE_ME

The actual Secret is created separately with kubectl.

3. PostgreSQL PersistentVolumeClaim

File: postgres-pvc.yaml

The PostgreSQL PVC requests 5 GiB of persistent storage using ReadWriteOnce.

The live PVC was verified as Bound with 5 GiB capacity.

4. PostgreSQL Deployment

File: postgres-deployment.yaml

The Deployment runs postgres:16-alpine, mounts postgres-pvc, uses the ConfigMap and Secret, exposes port 5432, and defines startup, readiness and liveness probes.

Resource configuration:

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"

The PostgreSQL health checks use pg_isready.

5. PostgreSQL Service

File: postgres-service.yaml

The PostgreSQL Service is a ClusterIP service on port 5432 and provides internal Kubernetes networking between Scrob and PostgreSQL.

Scrob Pod
    |
    v
postgres:5432
    |
    v
PostgreSQL Pod

6. Scrob PersistentVolumeClaim

File: scrob-pvc.yaml

Scrob persistent application data is stored at:

/app/backend/data

The PVC requests 5 GiB with ReadWriteOnce.

The live PVC was verified as Bound.

7. Scrob Deployment

File: scrob-deployment.yaml

The Scrob Deployment uses:

bellamy/scrob:latest

The application listens on port 7330.

It receives:

SECRET_KEY from scrob-secret

DATABASE_URL from scrob-secret

The persistent volume is mounted at:

/app/backend/data

The Deployment uses:

enableServiceLinks: false

This prevents automatically injected Kubernetes Service environment variables from interfering with Scrob's database configuration.

Scrob Health Probes

Scrob uses /login for HTTP startup, readiness and liveness checks.

The endpoint was tested from inside Kubernetes and returned:

HTTP/1.1 200 OK

The Deployment also defines CPU and memory requests/limits:

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"

8. Scrob Service

File: scrob-service.yaml

The Scrob Service is a ClusterIP service exposing port 7330.

The Ingress controller routes external HTTP requests to this Service.

9. Ingress

File: ingress.yaml

The Ingress routes:

scrob.local

to the Scrob Service on port 7330.

Traffic flow:

http://scrob.local/
        |
        v
NGINX Ingress Controller
        |
        v
Scrob Service :7330
        |
        v
Scrob Pod

10. Applying the Kubernetes Resources

Resources can be applied manually:

kubectl apply -f postgres-configmap.yaml
kubectl apply -f postgres-secret.yaml
kubectl apply -f postgres-pvc.yaml
kubectl apply -f postgres-deployment.yaml
kubectl apply -f postgres-service.yaml

kubectl apply -f scrob-pvc.yaml
kubectl apply -f scrob-deployment.yaml
kubectl apply -f scrob-service.yaml

kubectl apply -f ingress.yaml

Manifests were validated with:

kubectl apply --dry-run=client -f <file>

before being applied.

11. Verification

Nodes

kubectl get nodes

All three Kind nodes should be Ready.

Pods

kubectl get pods

Expected:

postgres-...   1/1   Running
scrob-...      1/1   Running

Services

kubectl get services

Expected services include:

postgres
scrob

PVCs

kubectl get pvc

Expected:

postgres-pvc   Bound   5Gi
scrob-pvc      Bound   5Gi

Ingress

kubectl get ingress

Expected host:

scrob.local

12. Internal Scrob Test

The Scrob Service was tested from inside the cluster:

kubectl run scrob-test   --restart=Never   --image=curlimages/curl:latest   --command --   curl -I http://scrob:7330/login

The response was:

HTTP/1.1 200 OK

The temporary test Pod was removed afterwards:

kubectl delete pod scrob-test

13. External Ingress Test

The application was tested through NGINX Ingress:

curl -I http://scrob.local/

The response was:

HTTP/1.1 302 Found
location: /login

This confirms:

Client
  |
  v
NGINX Ingress
  |
  v
Scrob Service
  |
  v
Scrob Pod

is functioning correctly.

14. PostgreSQL Connectivity Test

A temporary PostgreSQL client Pod was used to test database connectivity:

kubectl run postgres-client   --restart=Never   --image=postgres:16-alpine   --env="PGPASSWORD=$(kubectl get secret postgres-secret -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d)"   --command --   psql -h postgres -U scrob -d scrob   -c "SELECT current_database(), current_user;"

The test successfully connected to PostgreSQL through the Kubernetes Service.

The temporary client Pod was removed afterwards.

15. PostgreSQL Persistence Test

A test table and record were created:

CREATE TABLE IF NOT EXISTS stage4_persistence_test (
  id SERIAL PRIMARY KEY,
  message TEXT
);

INSERT INTO stage4_persistence_test (message)
VALUES ('Stage 4 PVC persistence test');

The record was returned successfully:

id | message
---+------------------------------
1  | Stage 4 PVC persistence test

The PostgreSQL Pod was deliberately deleted:

kubectl delete pod -l app=postgres

Kubernetes recreated the Pod.

The record was queried again and remained present.

Result

PostgreSQL data survived Pod replacement because it was stored on postgres-pvc.

16. Scrob Persistence Test

A test file was written to the Scrob persistent data directory:

kubectl exec deployment/scrob -- sh -c 'echo "Stage 4 Scrob persistence test" > /app/backend/data/stage4-test.txt'

The Scrob Pod was deliberately deleted:

kubectl delete pod -l app=scrob

Kubernetes created a replacement Pod.

The file was then read from the replacement:

kubectl exec deployment/scrob -- cat /app/backend/data/stage4-test.txt

Output:

Stage 4 Scrob persistence test

Result

Scrob application data survived Pod replacement because it was stored on scrob-pvc.

17. Troubleshooting Lessons

Scrob data directory

Scrob requires application data at:

/app/backend/data

The Scrob PVC is mounted at this location.

Kubernetes Service Links

Automatically injected Kubernetes Service environment variables interfered with Scrob's database configuration.

The Deployment therefore uses:

enableServiceLinks: false

PostgreSQL Startup

PostgreSQL uses a startup probe in addition to readiness and liveness probes. This gives PostgreSQL time to initialise or recover before normal health checks determine container health.

Non-root execution experiment

A non-root execution test was performed using the scrob user found inside the image:

scrob:x:1000:1000

Running the container as UID 1000 failed because the image startup process attempted to create:

/etc/localtime

and received:

Permission denied

The deployment was restored to the known-working configuration.

This demonstrates:

An image containing a non-root user does not necessarily mean the application can run successfully as that user.

18. Key Kubernetes Concepts Learned

Kubernetes Deployments

Pods

Services

ClusterIP

ConfigMaps

Secrets

PersistentVolumeClaims

Persistent storage

Ingress

NGINX Ingress Controller

HTTP routing

Startup probes

Readiness probes

Liveness probes

Resource requests

Resource limits

Kubernetes DNS/service discovery

Manual kubectl apply

Pod replacement

Application persistence

Database persistence

Kubernetes troubleshooting

19. Final Validation

Final application state:

PostgreSQL Pod       → Running
Scrob Pod            → Running
PostgreSQL Service   → Available
Scrob Service        → Available
PostgreSQL PVC       → Bound
Scrob PVC            → Bound
Ingress              → Available
Scrob /login         → HTTP 200
scrob.local          → HTTP 302 → /login
PostgreSQL data      → Persistent
Scrob data           → Persistent
