Stage 3 — Kubernetes Basics + NGINX Ingress

This stage deploys Scrob and PostgreSQL into the Kind Kubernetes cluster and exposes Scrob through an NGINX Ingress controller.

Architecture

Browser / Windows Host
        |
        | http://scrob.local
        v
127.0.0.1:80
        |
        v
Kind Control Plane
        |
        v
NGINX Ingress Controller
        |
        v
Scrob Ingress
        |
        v
Scrob ClusterIP Service :7330
        |
        v
Scrob Pod
        |
        | DATABASE_URL
        v
PostgreSQL ClusterIP Service :5432
        |
        v
PostgreSQL Pod
        |
        v
PostgreSQL PVC

Scrob also uses its own persistent storage:

Scrob Pod
   |
   v
/app/backend/data
   |
   v
scrob-pvc

Objectives

This stage demonstrates:

Kubernetes Deployments and Pods

ClusterIP Services

Kubernetes DNS and EndpointSlices

ConfigMaps and Secrets

PersistentVolumeClaims

NGINX Ingress Controller

IngressClass and host-based routing

Kind extraPortMappings

Node labels and node selection

External HTTP traffic into Kubernetes

Scrob-to-PostgreSQL communication

Troubleshooting real Kubernetes configuration and networking issues

Scrob is exposed through Ingress, not NodePort.

Files

stage-3-kubernetes/
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

File

Purpose

postgres-configmap.yaml

Non-sensitive PostgreSQL configuration

postgres-secret.yaml

Template for the PostgreSQL password Secret

postgres-pvc.yaml

5Gi PostgreSQL persistent storage

postgres-deployment.yaml

PostgreSQL Deployment

postgres-service.yaml

Internal PostgreSQL Service

scrob-pvc.yaml

5Gi Scrob persistent storage

scrob-deployment.yaml

Scrob Deployment

scrob-service.yaml

Internal Scrob Service

ingress.yaml

Routes scrob.local to Scrob

PostgreSQL

PostgreSQL uses the postgres:16-alpine image.

The ConfigMap contains:

POSTGRES_DB: scrob
POSTGRES_USER: scrob

The password is stored in a Kubernetes Secret. The repository contains only a template with CHANGE_ME; the real password was created separately with kubectl and must not be committed.

PostgreSQL stores data at:

/var/lib/postgresql/data

through postgres-pvc.

The PostgreSQL Service is:

postgres:5432

and is a ClusterIP Service, so it is internal to the cluster.

PostgreSQL Verification

PostgreSQL was verified with:

kubectl get pods
kubectl get pvc
kubectl get svc postgres
kubectl get endpointslice -l kubernetes.io/service-name=postgres

A temporary PostgreSQL client Pod successfully executed:

SELECT current_database(), current_user;

and returned:

scrob | scrob

This verified Kubernetes DNS, Service routing, PostgreSQL connectivity, and authentication.

The temporary client Pod was deleted after testing.

Scrob

Scrob uses:

bellamy/scrob:latest

and listens on port 7330.

The Deployment receives:

SECRET_KEY
DATABASE_URL

from the scrob-secret Kubernetes Secret.

The database connection uses the Kubernetes PostgreSQL Service:

postgresql+asyncpg://scrob:<password>@postgres:5432/scrob

The real credentials are not stored in Git.

Scrob Persistent Storage

Scrob requires:

/app/backend/data

The first Kubernetes deployment failed because this directory was not available to the container:

chown: cannot access '/app/backend/data': No such file or directory

A scrob-pvc was created and mounted at /app/backend/data, resolving the issue.

Kubernetes Service Environment Variable Issue

A second issue occurred because Kubernetes automatically injects Service environment variables into Pods.

Because the PostgreSQL Service was named postgres, Scrob received a variable similar to:

POSTGRES_PORT=tcp://10.96.184.92:5432

Scrob expected POSTGRES_PORT to be an integer and failed with:

Input should be a valid integer
input_value='tcp://10.96.184.92:5432'

The Scrob Pod was configured with:

enableServiceLinks: false

This prevented automatic Service environment-variable injection and removed the collision.

PostgreSQL Authentication Issue

After fixing the Service-variable collision, Scrob reached PostgreSQL but initially failed authentication:

asyncpg.exceptions.InvalidPasswordError:
password authentication failed for user "scrob"

The important lesson is that changing the Kubernetes Secret does not automatically change the password of an already-initialized PostgreSQL database.

The Scrob Secret was recreated using the same password that PostgreSQL had been initialized with.

Scrob then successfully connected to PostgreSQL and completed its database migrations.

Scrob Service

Scrob uses a ClusterIP Service:

scrob:7330

The Service was verified as:

scrob   ClusterIP   10.96.155.17   <none>   7330/TCP

Its EndpointSlice contained the Scrob Pod:

10.244.2.8:7330

A temporary curl Pod tested:

kubectl run scrob-client   --restart=Never   --image=curlimages/curl:latest   --command --   curl -I http://scrob:7330

The response was:

HTTP/1.1 302 Found
location: /login

This is expected because unauthenticated users are redirected to /login.

The temporary Pod was deleted after the test.

NGINX Ingress Controller

NGINX Ingress Controller was installed using the Kind-specific ingress-nginx manifest.

The controller version used for this stage was:

v1.15.1

The controller runs in:

ingress-nginx

namespace.

Verification:

kubectl get pods -n ingress-nginx

The controller reached:

1/1 Running

The IngressClass is:

nginx

with controller:

k8s.io/ingress-nginx

Ingress Controller Placement

The Kind cluster was configured in Stage 2 with the control-plane node labelled:

ingress-ready=true

Kind also maps:

host port 80  -> control-plane port 80
host port 443 -> control-plane port 443

Initially, NGINX was scheduled on a worker node. Internal Ingress traffic worked, but the external request to:

curl -I -H "Host: scrob.local" http://127.0.0.1/

returned:

curl: (56) Recv failure: Connection reset by peer

The controller was then configured to use the ingress-ready=true node.

After moving NGINX to the control-plane node, the external request succeeded.

Lesson

With Kind host-port mappings, the node receiving the mapped host traffic needs to be the node running the host-port-dependent ingress controller.

Ingress Configuration

The final ingress.yaml is:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: scrob-ingress
spec:
  ingressClassName: nginx

  rules:
    - host: scrob.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: scrob
                port:
                  number: 7330

The routing rule is:

scrob.local
    /
    |
    v
scrob Service :7330

The Ingress targets the Service, not the Pod directly.

Ingress
   |
   v
Service
   |
   v
Pod

Ingress Verification

The Ingress was verified with:

kubectl get ingress

Result:

scrob-ingress   nginx   scrob.local   localhost   80

The NGINX controller logs confirmed that the IngressClass was valid and that the Ingress was scheduled for sync.

An internal NGINX test also returned:

HTTP/1.1 302 Found
location: /login

This proved:

NGINX
  |
  v
Ingress rule
  |
  v
Scrob Service
  |
  v
Scrob Pod

Windows and WSL Hostname Resolution

The application hostname is:

scrob.local

WSL contains:

127.0.0.1 scrob.local

The Windows hosts file also contains:

127.0.0.1 scrob.local

This is necessary because Windows and WSL have separate host-resolution contexts.

After flushing the Windows DNS cache, the hostname worked from Windows.

The final test was:

curl -I http://scrob.local/

and returned:

HTTP/1.1 302 Found
location: /login

The application is therefore accessible in the browser at:

http://scrob.local

Final Verification

Final application Pods:

postgres                         1/1 Running
scrob                            1/1 Running
ingress-nginx-controller         1/1 Running

Services:

postgres   ClusterIP   :5432
scrob      ClusterIP   :7330

PersistentVolumeClaims:

postgres-pvc   Bound   5Gi   RWO
scrob-pvc      Bound   5Gi   RWO

Ingress:

scrob-ingress   nginx   scrob.local   localhost   80

IngressClass:

nginx   k8s.io/ingress-nginx

Final end-to-end HTTP test:

curl -I http://scrob.local/

Result:

HTTP/1.1 302 Found
location: /login

Key Lessons

ClusterIP vs NodePort

Scrob is exposed internally using ClusterIP. External HTTP access is provided by the Ingress controller rather than a NodePort.

Service vs Pod

Ingress routes to:

scrob:7330

rather than directly to the Pod IP. The Service provides a stable endpoint while Pods can be replaced.

Kubernetes DNS

Inside the cluster:

postgres
scrob

resolve to their corresponding Kubernetes Services.

ConfigMap vs Secret

ConfigMap:

POSTGRES_DB
POSTGRES_USER

Secret:

POSTGRES_PASSWORD
SECRET_KEY
DATABASE_URL

Sensitive credentials should never be committed to Git.

PersistentVolumeClaim

Both PostgreSQL and Scrob have persistent storage through their own PVCs.

Ingress

Ingress provides HTTP host-based routing:

scrob.local

to:

scrob Service :7330

Useful Commands

kubectl get pods
kubectl get pods -A
kubectl get svc
kubectl get pvc
kubectl get ingress
kubectl get ingressclass
kubectl get endpointslice -l kubernetes.io/service-name=scrob
kubectl get endpointslice -l kubernetes.io/service-name=postgres
kubectl logs deployment/scrob
kubectl logs deployment/postgres
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
curl -I http://scrob.local/

Stage 3 Status

COMPLETED

Scrob is now running on Kubernetes with PostgreSQL, persistent storage, ClusterIP Services, Kubernetes Secrets and ConfigMaps, and NGINX Ingress.

The application is accessible through:

http://scrob.local

The next stage can build on this working Kubernetes foundation by improving the application/database deployment and then introducing the service mesh.
