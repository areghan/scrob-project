# Scrob — Kubernetes GitOps Deployment

A hands-on deployment of [Scrob](https://github.com/ellite/scrob) using Docker, PostgreSQL, Kubernetes, Istio and Argo CD.

This project is being built step-by-step, starting with a local Docker deployment and progressively moving towards a GitOps-managed Kubernetes deployment.

---

## 📋 Project Overview

The goal of this project is to deploy and operate Scrob while learning and documenting the underlying infrastructure and DevOps concepts.

Rather than jumping directly into Kubernetes, the project starts by understanding the application and its dependencies in Docker, then progressively moves towards Kubernetes, Istio and GitOps with Argo CD.

---

## 🛠️ Technology Stack

| Technology         | Purpose                      |
| ------------------ | ---------------------------- |
| **Scrob**          | Application                  |
| **PostgreSQL**     | Database                     |
| **Docker**         | Containerization             |
| **Docker Compose** | Local application deployment |
| **Kubernetes**     | Container orchestration      |
| **Kind**           | Local Kubernetes cluster     |
| **Istio**          | Service mesh and ingress     |
| **Argo CD**        | GitOps continuous delivery   |
| **GitHub**         | Source control               |

---

# 🏗️ Architecture

## Initial Docker Architecture

```text
                    Browser
                       |
                       | HTTP
                       v
                +-------------+
                |    Scrob    |
                |     App     |
                +------+------+
                       |
                       | PostgreSQL
                       v
                +-------------+
                | PostgreSQL  |
                +------+------+
                       |
                       v
                 Persistent
                   Storage
```

## Target Kubernetes Architecture

```text
                         GitHub
                            |
                            |
                            v
                         Argo CD
                            |
                            | GitOps Sync
                            v
                 +----------------------+
                 |     Kind Cluster     |
                 |                      |
                 |       Istio          |
                 |         |            |
                 |       Scrob          |
                 |         |            |
                 |    PostgreSQL         |
                 |         |            |
                 |        PVC            |
                 +----------------------+
```

---

# 🚀 Project Stages

## Stage 0 — Project Preparation

* [x] Create project directory
* [x] Initialise Git repository
* [x] Switch default branch to `main`
* [x] Create README
* [x] Create `.gitignore`
* [ ] Create initial Git commit
* [ ] Create GitHub repository

---

## Stage 1 — Docker

* [x] Inspect the Scrob repository
* [x] Understand the application requirements
* [x] Understand the Docker image
* [x] Understand the PostgreSQL dependency
* [x] Create Docker Compose configuration
* [x] Configure PostgreSQL
* [x] Configure Scrob
* [x] Configure persistent storage
* [x] Start the application
* [x] Verify Scrob
* [x] Test PostgreSQL connectivity
* [x] Document the Docker architecture
* [x] Troubleshoot the Scrob persistent-data volume path
* [x] Verify container health
* [x] Verify HTTP access
---

## Stage 2 — Kind Kubernetes Cluster

* [ ] Create Kind cluster
* [ ] Configure cluster networking
* [ ] Configure port mappings
* [ ] Verify Kubernetes nodes
* [ ] Verify cluster connectivity
* [ ] Verify Kubernetes version
* [ ] Document the cluster configuration

---

## Stage 3 — PostgreSQL on Kubernetes

* [ ] Create Scrob namespace
* [ ] Create PostgreSQL Secret
* [ ] Create PersistentVolumeClaim
* [ ] Create PostgreSQL Deployment
* [ ] Create PostgreSQL Service
* [ ] Configure PostgreSQL environment variables
* [ ] Verify PostgreSQL pod
* [ ] Verify PostgreSQL Service
* [ ] Test database connectivity
* [ ] Test persistent storage
* [ ] Document PostgreSQL architecture

---

## Stage 4 — Scrob on Kubernetes

* [ ] Create Scrob configuration
* [ ] Create Scrob Secret
* [ ] Configure database connection
* [ ] Create Scrob Deployment
* [ ] Create Scrob Service
* [ ] Configure persistent storage if required
* [ ] Configure health checks
* [ ] Verify Scrob pod
* [ ] Verify Scrob Service
* [ ] Test application connectivity
* [ ] Document the Scrob Kubernetes deployment

---

## Stage 5 — Istio

* [ ] Install/verify Istio
* [ ] Enable sidecar injection
* [ ] Verify Istio components
* [ ] Create Istio Gateway
* [ ] Create VirtualService
* [ ] Configure application routing
* [ ] Configure external access
* [ ] Test traffic flow
* [ ] Verify Envoy sidecars
* [ ] Document Istio architecture

---

## Stage 6 — Argo CD

* [ ] Install/verify Argo CD
* [ ] Configure Argo CD access
* [ ] Create Git repository structure
* [ ] Create Kubernetes manifests
* [ ] Create Argo CD Application
* [ ] Connect Argo CD to GitHub
* [ ] Perform initial synchronization
* [ ] Verify application deployment
* [ ] Document Argo CD configuration

---

## Stage 7 — GitOps

* [ ] Make configuration changes through Git
* [ ] Commit changes
* [ ] Push changes to GitHub
* [ ] Observe Argo CD synchronization
* [ ] Verify Kubernetes changes
* [ ] Test automated synchronization
* [ ] Test rollback
* [ ] Document GitOps workflow

---

## Stage 8 — Troubleshooting

The project will intentionally include troubleshooting exercises.

* [ ] Investigate failed pods
* [ ] Investigate CrashLoopBackOff
* [ ] Investigate ImagePullBackOff
* [ ] Investigate database connectivity failures
* [ ] Investigate incorrect Secrets
* [ ] Investigate Service configuration
* [ ] Investigate DNS problems
* [ ] Investigate Istio routing problems
* [ ] Investigate storage problems
* [ ] Practice Kubernetes logs
* [ ] Practice `kubectl describe`
* [ ] Practice `kubectl exec`
* [ ] Practice Kubernetes events

---

## Stage 9 — Improvements

* [ ] Pin application image versions
* [ ] Avoid using `latest` in production-style manifests
* [ ] Improve Secrets management
* [ ] Configure resource requests
* [ ] Configure resource limits
* [ ] Configure readiness probes
* [ ] Configure liveness probes
* [ ] Improve PostgreSQL persistence
* [ ] Improve security
* [ ] Review network policies
* [ ] Review production considerations

---

# 📁 Repository Structure

The repository will evolve throughout the project.

```text
scrob-project/
│
├── README.md
├── .gitignore
│
├── stage-1-docker/
│   ├── docker-compose.yaml
│   ├── .env.example
│   └── README.md
│
├── stage-2-kind/
│   ├── kind-cluster.yaml
│   └── README.md
│
├── stage-3-postgres/
│   ├── namespace.yaml
│   ├── secret.yaml
│   ├── pvc.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── README.md
│
├── stage-4-scrob/
│   ├── secret.yaml
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── README.md
│
├── stage-5-istio/
│   ├── gateway.yaml
│   ├── virtualservice.yaml
│   └── README.md
│
└── stage-6-argocd/
    ├── application.yaml
    └── README.md
```

---

# 🎯 Learning Objectives

This project is designed to provide practical experience with:

* Git
* GitHub
* Linux
* Docker
* Docker Compose
* PostgreSQL
* Container networking
* Kubernetes
* Kind
* Kubernetes Deployments
* Kubernetes Services
* Kubernetes Secrets
* PersistentVolumeClaims
* Kubernetes networking
* Istio
* Argo CD
* GitOps
* Application troubleshooting

---

# 📚 Learning Approach

This project is intentionally being built from **Stage 0** rather than deploying everything at once.

Each stage will explain:

1. **What** we are building
2. **Why** we are building it
3. **How** it works
4. The configuration used
5. The commands used
6. How the deployment is verified
7. Problems encountered
8. How problems were solved
9. What was learned

The goal is not simply to make Scrob work.

The goal is to understand **why it works**.

---

# 🔐 Security

Secrets and credentials must never be committed to Git.

Sensitive values will be stored locally and excluded using `.gitignore`.

Examples include:

* Database passwords
* API tokens
* Application secrets
* Kubernetes credentials
* Environment files containing secrets

Where appropriate, the repository will contain safe example files such as:

```text
.env.example
```

instead of actual credentials.

---

# 🐳 Container Images

The project will use pre-built container images wherever possible rather than building the application from source.

Application image versions will eventually be pinned to specific releases to make deployments reproducible.

---

# ☸️ Kubernetes

The Kubernetes deployment will initially use standard Kubernetes manifests rather than relying entirely on Helm.

This is intentional.

The objective is to understand the individual Kubernetes resources before introducing higher-level deployment abstractions.

---

# 🌐 Service Mesh

Istio will be introduced after the basic Kubernetes deployment is working.

The project will explore:

* Ingress
* Gateways
* VirtualServices
* Envoy sidecars
* Traffic routing
* Service-to-service communication

---

# 🔄 GitOps

Argo CD will eventually become responsible for keeping the Kubernetes environment synchronized with the Git repository.

The intended workflow is:

```text
Developer
    |
    | Git commit
    v
GitHub
    |
    | Repository changes
    v
Argo CD
    |
    | Synchronization
    v
Kubernetes
    |
    v
Scrob
```

The desired end state is that **Git becomes the source of truth** for the Kubernetes deployment.

---

# 🧪 Verification

Every major stage will include verification commands and tests.

Examples:

```bash
kubectl get nodes
```

```bash
kubectl get pods
```

```bash
kubectl get services
```

```bash
kubectl logs <pod>
```

```bash
kubectl describe pod <pod>
```

The exact commands will be documented as each stage is completed.

---

# 📈 Project Status

| Stage                     | Status         |
| ------------------------- | -------------- |
| Stage 0 — Preparation     | ✅ Complete    |
| Stage 1 — Docker          | ✅ Complete    |
| Stage 2 — Kind            | 🔵 Next        |
| Stage 3 — PostgreSQL      | ⚪ Not Started |
| Stage 4 — Scrob           | ⚪ Not Started |
| Stage 5 — Istio           | ⚪ Not Started |
| Stage 6 — Argo CD         | ⚪ Not Started |
| Stage 7 — GitOps          | ⚪ Not Started |
| Stage 8 — Troubleshooting | ⚪ Not Started |
| Stage 9 — Improvements    | ⚪ Not Started |

---

# 📝 Project Notes

This repository is a practical learning project and deployment record.

Configuration decisions, troubleshooting steps and lessons learned will be added throughout the project.

The README will be updated as each stage is completed.

---

# 🏁 Final Goal

The final environment should provide a GitOps-managed Scrob deployment running on a local Kind Kubernetes cluster with:

```text
                         GitHub
                            |
                            v
                         Argo CD
                            |
                            v
                    +---------------+
                    | Kind Cluster  |
                    |               |
                    |    Istio      |
                    |      |        |
                    |    Scrob      |
                    |      |        |
                    | PostgreSQL    |
                    |      |        |
                    |     PVC       |
                    +---------------+
```

**From Stage 0 → Docker → Kubernetes → Istio → Argo CD → GitOps.**

