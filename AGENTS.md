# Repository Guidelines

## Project Structure & Module Organization

- Root contains orchestration and docs: `skaffold.yaml`, `cloudbuild.yaml`, `README.md`, `docs/`.
- Kubernetes assets:
  - Plain manifests: `kubernetes-manifests/`
  - Kustomize overlays/components: `kustomize/` (includes `base/`, `components/`, `tests/`)
  - Istio configs: `istio-manifests/`
  - Helm chart: `helm-chart/`
  - Release bundle: `release/kubernetes-manifests.yaml`
- Services source code: `src/` with one directory per microservice (Go, Java, Node.js, Python, .NET/C#).
- Protobuf contracts: `protos/`
- CI/CD and workflows: `.github/workflows/`, `cloudbuild.yaml`.
- Terraform/IaC: `terraform/`
- Editor config and repo settings: `.editorconfig`, `.gitattributes`, `.gitignore`.

## Build, Test, and Development Commands

```bash
# Build and deploy to current kube-context (first run can take ~20 min)
skaffold run

# Live development with rebuilds and redeploys on change
skaffold dev

# Use Google Cloud Build (no local Docker cache)
skaffold run -p gcb --default-repo=us-docker.pkg.dev/PROJECT_ID/microservices-demo

# Deploy prebuilt release bundle (GKE quickstart)
kubectl apply -f ./release/kubernetes-manifests.yaml

# Port-forward frontend locally (after deploy)
kubectl port-forward deployment/frontend 8080:8080
```

## Coding Style & Naming Conventions

- Indentation (from `.editorconfig`):
  - Default: spaces, size 2
  - C#/.py/.java/Dockerfile: spaces size 4
  - Go: tabs
- File naming: follow language norms per service (e.g., Go package files in `src/*service`, Java under `src/adservice/src/main/java/...`).
- Functions/variables: use idiomatic naming per language ecosystem.
- Linting/formatting: follow language-standard tools (e.g., `gofmt` for Go). No central linter config is provided in repo.

## Testing Guidelines

- Frameworks vary by language per service. No top-level unified test runner is defined.
- Cartservice has tests under `src/cartservice/tests/`. Other services may include language-specific tests within their directories.
- Run tests using the language’s tooling inside each service container/env (e.g., `go test ./...` for Go services, `pytest` for Python services, etc.).
- Coverage requirements are not specified in this repository.

## Commit & Pull Request Guidelines

- Conventional Commit style is used in history:
  - Examples: `chore(deps): update ...`, `fix(deps): update ...`.
- PRs should include: clear description, service(s) touched, and deployment impact.
- Branch naming: not enforced here; match Conventional style when possible (e.g., `feat/`, `fix/`, `chore/`).

---

# Repository Tour

## 🎯 What This Repository Does

Online Boutique (microservices-demo) is a cloud-first microservices e‑commerce application used to demonstrate modern application development and operations on Kubernetes and Google Cloud.

**Key responsibilities:**
- Provide a polyglot microservices reference implementation
- Showcase Kubernetes deployment patterns (manifests, Kustomize, Helm, Skaffold)
- Demonstrate observability, service mesh, and cloud integrations

---

## 🏗️ Architecture Overview

### System Context
```
[User Browser] → [frontend (HTTP)] → [gRPC microservices]
                                 ↘︎ [Redis/Datastores via services]
```

### Key Components
- frontend (Go) – Serves the web UI and orchestrates calls to backend services.
- cartservice (C#) – Persists shopping carts in Redis; exposes gRPC.
- productcatalogservice (Go) – Serves product data from JSON; gRPC API.
- currencyservice (Node.js) – Currency conversion; high-QPS; gRPC.
- paymentservice (Node.js) – Mock charge and transaction IDs; gRPC.
- shippingservice (Go) – Shipping cost estimates; mock shipping; gRPC.
- emailservice (Python) – Sends mock order confirmations; gRPC.
- checkoutservice (Go) – Orchestrates order flow across services; gRPC.
- recommendationservice (Python) – Recommends products; gRPC.
- adservice (Java) – Contextual text ads; gRPC.
- loadgenerator (Python/Locust) – Generates synthetic traffic.

### Data Flow
1. User interacts with frontend over HTTP.
2. Frontend invokes backend services via gRPC for catalog, cart, currency, etc.
3. Stateful components (e.g., cartservice) interact with backing stores (e.g., Redis) through their own logic/manifests.
4. Responses are aggregated by frontend and returned as HTML/JSON to the user.

---

## 📁 Project Structure [Partial Directory Tree]

```
microservices-demo/
├── src/
│  ├── adservice/                    # Java service (Gradle)
│  ├── cartservice/                  # C# service (+ tests)
│  ├── checkoutservice/              # Go service
│  ├── currencyservice/              # Node.js service
│  ├── emailservice/                 # Python service
│  ├── frontend/                     # Go HTTP frontend (static assets/templates)
│  ├── loadgenerator/                # Traffic generation (Locust)
│  ├── paymentservice/               # Node.js service
│  ├── productcatalogservice/        # Go service
│  ├── recommendationservice/        # Python service
│  ├── shippingservice/              # Go service
│  └── shoppingassistantservice/     # Optional AI assistant
├── protos/                          # Shared protobuf specs (gRPC/health)
├── kubernetes-manifests/            # Base manifests applied by Skaffold
├── kustomize/                       # Components/overlays (istio, spanner, etc.)
├── istio-manifests/                 # Istio-specific YAML (if using service mesh)
├── helm-chart/                      # Helm chart for alternative packaging
├── release/kubernetes-manifests.yaml# Prebuilt manifest bundle
├── docs/                            # Development & release docs
├── skaffold.yaml                    # Build/deploy pipeline (multi-config)
└── cloudbuild.yaml                  # Cloud Build pipeline for GKE deploy
```

### Key Files to Know

| File | Purpose | When You'd Touch It |
|------|---------|---------------------|
| `skaffold.yaml` | Defines images, build platforms, manifests and profiles | Change build/deploy flow or add a service image |
| `release/kubernetes-manifests.yaml` | One-shot deploy of the default app | Quickstart on a cluster without local builds |
| `cloudbuild.yaml` | Cloud Build steps invoking Skaffold | CI/CD on GCP |
| `kubernetes-manifests/` | App manifests deployed by Skaffold | Edit service/deployment YAML |
| `kustomize/components/*` | Optional variations (istio, spanner, etc.) | Enable features/overlays |
| `src/*/` | Service source code | Develop a microservice |
| `protos/` | Shared protobuf definitions | Update cross-service contracts |
| `docs/development-guide.md` | Local dev instructions | Onboarding and troubleshooting |

---

## 🔧 Technology Stack

### Core Technologies
- Language: Polyglot (Go, Java, Node.js, Python, C#)
- Frameworks/APIs: gRPC + Protocol Buffers
- Orchestration: Kubernetes (manifests, Kustomize, Helm)
- Build/Deploy: Skaffold v3 configs; optional Google Cloud Build
- Cloud Integrations (optional): Istio/CSM, Spanner, Memorystore, AlloyDB, Gemini

### Key Libraries
- Protobuf/gRPC across services (see `protos/` and service-specific deps).

### Development Tools
- Skaffold – Multi-service build/deploy workflow
- kubectl – Apply and interact with cluster resources
- Docker – Image build runtime

---

## 🌐 External Dependencies

- Kubernetes cluster with sufficient resources (local or GKE).
- Container registry when pushing images (e.g., Artifact Registry via `--default-repo`).
- Redis backing store for cartservice is provisioned via manifests when deploying the default bundle.

### Environment Variables [Optional]

- Skaffold default repo: set via `--default-repo` or environment as per Skaffold docs.
- GCP variables for CI/CD examples: `_ZONE`, `_CLUSTER` used by `cloudbuild.yaml` substitutions.

---

## 🔄 Common Workflows

- Local development:
  - Start cluster (minikube/kind or Docker Desktop with k8s)
  - `skaffold dev` for iterative development
- One-off deploy:
  - `kubectl apply -f release/kubernetes-manifests.yaml`
- GCP CI/CD:
  - Cloud Build triggers `skaffold run` with `--default-repo` in `cloudbuild.yaml`

---

## 📈 Performance & Scale

- Services are independently scalable via Kubernetes Deployments. Use HPA or adjust replicas in manifests.

---

## 🚨 Things to Be Careful About

- Ensure your kube-context points to the intended cluster before running Skaffold.
- Using the GCB profile disables local caching; builds are slower but don’t require local Docker.
- Protobuf changes require regenerating stubs inside each affected service.


*Updated at: 2025-10-14*
