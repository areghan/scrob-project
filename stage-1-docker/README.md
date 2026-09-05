# Stage 1 — Docker Deployment

## Objective

Deploy Scrob locally using Docker Compose with PostgreSQL as a separate database container.

This stage establishes the application architecture before moving the workload into Kubernetes.

---

## Architecture

```text
                    Browser
                       |
                       | HTTP :7330
                       v
                +-------------+
                |    Scrob    |
                |     App     |
                +------+------+
                       |
                       | PostgreSQL :5432
                       v
                +-------------+
                | PostgreSQL  |
                |     16      |
                +------+------+
                       |
                       v
                Docker Volume
```

---

## Why PostgreSQL Is Separate

Scrob provides an omnibus image containing PostgreSQL, but this project will use the standard deployment with PostgreSQL as a separate container.

This is intentional.

The eventual Kubernetes architecture will also separate:

```text
Scrob
  |
  v
PostgreSQL
```

This allows us to learn:

* Container-to-container networking
* Database configuration
* Environment variables
* Persistent volumes
* Service discovery
* Application/database dependencies
* Kubernetes Services
* Kubernetes PersistentVolumeClaims

---

## Container Images

### Scrob

```text
bellamy/scrob:latest
```

Scrob also has a GHCR mirror:

```text
ghcr.io/ellite/scrob:latest
```

For the initial Docker learning stage, the documented Docker Hub image will be used.

Later in the project we will pin the application to a specific release rather than relying on `latest`.

### PostgreSQL

```text
postgres:16-alpine
```

Scrob's current Docker documentation specifies PostgreSQL 16 for the standard deployment.

---

## Ports

| Component  | Container Port |    Host Port | Purpose         |
| ---------- | -------------: | -----------: | --------------- |
| Scrob      |           7330 |         7330 | Web application |
| PostgreSQL |           5432 | Not required | Database        |

PostgreSQL does not need to be exposed to the host because Scrob communicates with it through the Docker network.

---

## Scrob Configuration

The standard Scrob deployment requires:

```text
SECRET_KEY
DATABASE_URL
```

### SECRET_KEY

The secret key is used by Scrob for JWT signing.

A secure key can be generated with:

```bash
openssl rand -hex 32
```

### DATABASE_URL

The PostgreSQL connection string follows this format:

```text
postgresql+asyncpg://USER:PASSWORD@HOST:5432/DATABASE
```

For our Docker Compose deployment:

```text
postgresql+asyncpg://scrob:PASSWORD@scrob-db:5432/scrob
```

The hostname is `scrob-db` because Docker Compose provides service-name DNS resolution.

---

## PostgreSQL Configuration

The PostgreSQL container will use:

```text
POSTGRES_USER=scrob
POSTGRES_PASSWORD=<secure password>
POSTGRES_DB=scrob
```

The database data will be stored in a Docker volume.

---

## Persistent Storage

Two logical persistent data areas will be used:

```text
db_data
    |
    v
PostgreSQL data

scrob_data
    |
    v
Scrob application data
```

This prevents application and database data from being lost when containers are recreated.

---

## Environment Variables

Sensitive values will not be committed to Git.

The project will use:

```text
.env
```

for local secrets.

A safe template will be provided as:

```text
.env.example
```

The actual `.env` file must remain ignored by Git.

---

## Docker Compose

The final Stage 1 deployment will use:

```text
docker-compose.yaml
```

with two services:

```text
services:

  scrob-db:
    PostgreSQL 16

  scrob:
    Scrob application
```

The Scrob service will depend on PostgreSQL becoming healthy before starting.

---

## Health Check

PostgreSQL will use:

```bash
pg_isready -U scrob -d scrob
```

Docker Compose will use this health check to determine when PostgreSQL is ready.

Scrob will then start after PostgreSQL reports a healthy state.

---

## First Setup

Once the containers are running, Scrob should be available at:

```text
http://localhost:7330
```

The first user can create an account through the web interface.

Scrob requires a TMDB Read Access Token for metadata, search and images. This will be configured during the application setup.

---

## Verification

The deployment will be verified using:

```bash
docker compose ps
```

```bash
docker compose logs scrob
```

```bash
docker compose logs scrob-db
```

The Scrob web interface will then be tested at:

```text
http://localhost:7330
```

PostgreSQL connectivity will also be verified.

---

## Troubleshooting

Potential problems to investigate during this stage:

### PostgreSQL not ready

Check:

```bash
docker compose logs scrob-db
```

### Scrob cannot connect to PostgreSQL

Check:

```bash
docker compose logs scrob
```

Verify:

```text
DATABASE_URL
POSTGRES_USER
POSTGRES_PASSWORD
POSTGRES_DB
```

### Port 7330 already in use

Check:

```bash
sudo ss -ltnp | grep 7330
```

### Containers are running but Scrob is unavailable

Check:

```bash
docker compose ps
```

Then:

```bash
docker compose logs scrob
```

---

# Stage 1 Checklist

* [ ] Inspect official Scrob Docker configuration
* [ ] Create `.env.example`
* [ ] Create local `.env`
* [ ] Create `docker-compose.yaml`
* [ ] Pull PostgreSQL image
* [ ] Pull Scrob image
* [ ] Start PostgreSQL
* [ ] Start Scrob
* [ ] Verify container health
* [ ] Verify PostgreSQL connectivity
* [ ] Open Scrob web interface
* [ ] Complete first application setup
* [ ] Configure TMDB token
* [ ] Test application
* [ ] Document problems and solutions
* [ ] Update root README
* [ ] Commit Stage 1

---

## Learning Outcome

At the end of Stage 1, we should understand exactly how Scrob depends on PostgreSQL and how the two containers communicate.

This Docker architecture will become the reference architecture when the application is later migrated into Kubernetes.

