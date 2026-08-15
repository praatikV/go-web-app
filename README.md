# Go Web Application — Devopsified

A simple Go website, taken from a bare `net/http` app to a full CI/CD + GitOps pipeline. All the DevOps tooling in this repo — CI, containerization, Kubernetes manifests, Helm chart, and the GitOps release flow — was built and wired up by me ([@praatikV](https://github.com/praatikV)).

## App

The app itself is a small Go server using only the standard library `net/http` package to serve a few static HTML pages (home, courses, about, contact).

```bash
go run main.go
```

Visit `http://localhost:8080/courses`.

## What I built on top of it

| Piece | What it does |
|---|---|
| **CI (GitHub Actions)** | On every push to `main`: build the Go binary, run tests, lint with `golangci-lint` |
| **Containerization** | Multi-stage `Dockerfile` — compiles in a `golang:1.22.5` build stage, ships a minimal `distroless` runtime image |
| **CD (GitHub Actions)** | Builds and pushes the Docker image to Docker Hub, tagged with the GitHub Actions run ID |
| **GitOps sync** | CI automatically bumps the image tag in the Helm chart's `values.yaml` and commits it back to the repo |
| **Kubernetes** | Deployment, Service, and Ingress manifests, available both as raw YAML and as a Helm chart |

### Pipeline flow

```
push to main
   │
   ▼
CI: build → test → lint
   │
   ▼
Docker image built and pushed to Docker Hub
(tag = GitHub Actions run ID)
   │
   ▼
CI commits new image tag into helm/go-web-app-chart/values.yaml
   │
   ▼
Cluster picks up the new chart version and deploys
```

## Stack

- **Language:** Go 1.22
- **CI/CD:** GitHub Actions
- **Containerization:** Docker (multi-stage build, distroless final image)
- **Registry:** Docker Hub
- **Orchestration:** Kubernetes
- **Packaging:** Helm
- **Linting:** golangci-lint

## Repo structure

```
.
├── main.go                     # app entrypoint
├── main_test.go                # handler tests
├── Dockerfile                  # multi-stage build → distroless image
├── static/                     # HTML pages served by the app
├── .github/workflows/ci.yaml   # CI/CD pipeline
├── K8s/manifests/               # raw Kubernetes manifests (Deployment, Service, Ingress)
└── helm/go-web-app-chart/      # Helm chart for the same deployment
```

## Running it

### Locally
```bash
go run main.go
```

### With Docker
```bash
docker build -t go-web-app .
docker run -p 8080:8080 go-web-app
```

### On Kubernetes

Raw manifests:
```bash
kubectl apply -f K8s/manifests/
```

Or via Helm:
```bash
helm install go-web-app ./helm/go-web-app-chart
```

## CI/CD pipeline details

Defined in `.github/workflows/ci.yaml`, runs on every push to `main` (ignoring changes to `k8s/` and `README.md`):

- **`build`** — compiles the binary, runs `go test ./...`
- **`code-quality`** — runs `golangci-lint`
- **`push`** *(needs `build`)* — builds the Docker image and pushes it to Docker Hub tagged with the GitHub Actions run ID
- **`update-newtag-in-helm-chart`** *(needs `push`)* — updates the image tag in `helm/go-web-app-chart/values.yaml` and commits the change back to the repo, so the deployed chart always points at the latest built image

## Credits

Base application originally from [iam-veeramalla/go-web-app](https://github.com/iam-veeramalla/go-web-app). All DevOps implementation (CI/CD, Docker, Kubernetes, Helm, GitOps tagging flow) added by me.
