# Stage 8 — Argo CD GitOps

## Objective

Deploy the Scrob application using Argo CD and Helm from a Git repository.

## Components

- Argo CD
- Kubernetes kind cluster
- Helm chart
- Scrob application
- PostgreSQL database
- GitHub repository

## Deployment

The Argo CD Application pulls the Helm chart from:

https://github.com/areghan/scrob-project

Chart path:

stage-5-helm/scrob

Target namespace:

scrob-gitops

## GitOps Configuration

- Automated synchronisation enabled
- Pruning enabled
- Self-healing enabled
- Namespace creation enabled

## Validation

Argo CD application status:

- Sync status: Synced
- Health status: Healthy

Both Scrob and PostgreSQL pods were running successfully.

Persistent volume claims were bound:

- scrob-postgres-pvc — 5Gi
- scrob-scrob-pvc — 5Gi

Internal service testing returned:

HTTP 302 Found

Redirect destination:

/login

## Notes

This is a local learning environment. The current Helm configuration uses test credentials passed through the Argo CD Application manifest. Production environments should use a dedicated secret-management solution.
