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
              +--------+--------+
              |                 |
              v                 v
          scrob_data          db_data
        Docker Volume       Docker Volume

The final Docker architecture separates the application from the database.

Why PostgreSQL Is Separate

Scrob provides an omnibus image containing PostgreSQL, but this project uses the standard deployment with PostgreSQL as a separate container.

This is intentional.

The eventual Kubernetes architecture will also separate:

Scrob
  |
  v
PostgreSQL

This allows us to learn:

Container-to-container networking
Database configuration
Environment variables
Persistent volumes
Service discovery
Application/database dependencies
Kubernetes Services
Kubernetes PersistentVolumeClaims
Container Images
Scrob
bellamy/scrob:latest

Scrob also has a GHCR mirror:

ghcr.io/ellite/scrob:latest

For the initial Docker learning stage, the documented Docker Hub image is used.

Later in the project we will pin the application to a specific release rather than relying on latest.

PostgreSQL
postgres:16-alpine

PostgreSQL 16 is used for the standard Scrob deployment.

Ports
Component	Container Port	Host Port	Purpose
Scrob	7330	7330	Web application
PostgreSQL	5432	Not exposed	Database

PostgreSQL does not need to be exposed to the host because Scrob communicates with it through the Docker network.

Scrob Configuration

The deployment requires:

SECRET_KEY
DATABASE_URL
SECRET_KEY

The secret key is used by Scrob for JWT signing.

A secure key can be generated with:

openssl rand -hex 32

The generated secret is stored only in the local .env file.

It must never be committed to Git.

DATABASE_URL

The PostgreSQL connection string follows this format:

postgresql+asyncpg://USER:PASSWORD@HOST:5432/DATABASE

For this Docker Compose deployment:

postgresql+asyncpg://scrob:PASSWORD@scrob-db:5432/scrob

The hostname is:

scrob-db

because Docker Compose provides service-name DNS resolution between containers.

PostgreSQL Configuration

The PostgreSQL container uses:

POSTGRES_USER=scrob
POSTGRES_PASSWORD=<secure password>
POSTGRES_DB=scrob

The database data is stored in the named Docker volume:

db_data
Persistent Storage

Two logical persistent data areas are used:

db_data
    |
    v
PostgreSQL data

scrob_data
    |
    v
Scrob application data

The Scrob volume is mounted at:

/app/backend/data

inside the Scrob container.

The PostgreSQL volume is mounted at:

/var/lib/postgresql/data

inside the PostgreSQL container.

Using named volumes means application and database data can survive container recreation.

Environment Variables

Sensitive values are not committed to Git.

The project uses:

.env

for local secrets.

A safe template is provided as:

.env.example

The real .env file is ignored by Git.

This was verified with:

git check-ignore -v stage-1-docker/.env
Docker Compose

The Stage 1 deployment uses:

stage-1-docker/docker-compose.yaml

with two services:

services:

  scrob-db:
    PostgreSQL 16

  scrob:
    Scrob application

The Scrob service depends on PostgreSQL becoming healthy before starting.

PostgreSQL Health Check

PostgreSQL uses:

pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}

Docker Compose uses this health check to determine when PostgreSQL is ready.

Scrob then starts after PostgreSQL reports a healthy state.

Deployment

The Compose configuration was validated with:

docker compose config --quiet

The services were then started with:

docker compose up -d

Docker automatically uses locally available images and pulls missing images when required.

The deployed containers are:

scrob
scrob-db
Troubleshooting
Scrob container repeatedly restarting

During the initial deployment, the Scrob container repeatedly restarted with:

chown: cannot access '/app/backend/data': No such file or directory

The initial volume mapping was:

volumes:
  - scrob_data:/app/data

However, the current Scrob image expects its application data directory at:

/app/backend/data

The mapping was therefore changed to:

volumes:
  - scrob_data:/app/backend/data

The Compose configuration was then validated again:

docker compose config --quiet

The containers were recreated:

docker compose down
docker compose up -d

The existing named volumes were deliberately preserved by not using:

docker compose down -v

After the correction, the Scrob container started successfully and reported as healthy.

Lesson learned

A Docker volume mapping is not simply an arbitrary directory.

The mounted path must match the directory expected by the application inside the container.

This was diagnosed by reading the Scrob container logs:

docker compose logs scrob

The error message identified the expected path:

/app/backend/data

This demonstrates an important container troubleshooting workflow:

Application fails
      |
      v
Check container status
      |
      v
Read container logs
      |
      v
Identify expected path/configuration
      |
      v
Correct Compose configuration
      |
      v
Recreate container
      |
      v
Verify health
Verification
Container Health

The deployment was checked with:

docker ps | grep -E "scrob|postgres"

The result showed:

scrob       Up ... (healthy)
scrob-db    Up ... (healthy)

Both Scrob and the PostgreSQL containers were healthy.

Persistent Volume

The Scrob Docker volume was verified with:

docker volume ls | grep scrob

Result:

stage-1-docker_scrob_data

This confirms that the persistent Scrob volume exists.

HTTP Verification

The Scrob HTTP endpoint was tested with:

curl -I http://localhost:7330

The response was:

HTTP/1.1 302 Found
location: /login

This is a successful application response.

The 302 Found response means Scrob is redirecting an unauthenticated request to:

/login

This confirms that the Scrob application is reachable and responding to HTTP requests.

Web Interface

The Scrob web interface was successfully accessed through:

http://localhost:7330

The application loaded successfully in the browser.

First Application Setup

Scrob is available at:

http://localhost:7330

The first user can create an account through the web interface.

Additional application configuration, such as the TMDB Read Access Token, is application-level configuration and is separate from proving that the Docker deployment itself is functioning.

Useful Commands
View running containers
docker compose ps
View Scrob logs
docker compose logs scrob
Follow Scrob logs
docker compose logs -f scrob
View PostgreSQL logs
docker compose logs scrob-db
Enter the Scrob container
docker exec -it scrob sh
Enter PostgreSQL
docker exec -it scrob-db psql -U scrob -d scrob
List Docker volumes
docker volume ls
Inspect the Scrob volume
docker volume inspect stage-1-docker_scrob_data
Stage 1 Checklist
 Inspect official Scrob Docker configuration
 Create .env.example
 Create local .env
 Create docker-compose.yaml
 Pull/use PostgreSQL image
 Pull/use Scrob image
 Start PostgreSQL
 Start Scrob
 Verify container health
 Verify persistent Docker volumes
 Verify HTTP response
 Open Scrob web interface
 Troubleshoot Scrob volume path
 Document problems and solutions
 Configure TMDB token
 Complete additional application-level testing
 Update root README
 Commit Stage 1 completion
Learning Outcome

At the end of Stage 1, we understand how Scrob depends on PostgreSQL and how the two containers communicate.

We also understand:

Docker Compose services
Container networking
Service-name DNS
Environment variables
Secret handling
Docker named volumes
PostgreSQL persistence
Health checks
Service dependencies
Container logs
Troubleshooting volume mount paths
HTTP verification

This Docker architecture becomes the reference architecture when the application is later migrated into Kubernetes.

Next Stage

The next stage is:

Stage 2 — Kubernetes with Kind

We will recreate the application architecture inside a local Kubernetes cluster and begin translating the Docker concepts into Kubernetes concepts:

Docker Compose          Kubernetes

service             →   Deployment
service name        →   Service
environment         →   ConfigMap / Secret
named volume        →   PersistentVolumeClaim
depends_on          →   Kubernetes readiness/health mechanisms

The goal is to progressively move from:

Docker Compose
      |
      v
Kubernetes
      |
      v
Kind
      |
      v
Istio
      |
      v
Argo CD
      |
      v
GitOps

while keeping the architecture and lessons from each stage documented.
