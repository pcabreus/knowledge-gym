---
id: docker
title: "Docker (images & best practices for platform engineers)"
type: technology
status: learning
importance: 70
difficulty: easy
tags: [docker, containers, images, go, ci-cd]
canonical: true
aliases: []
created_at: 2026-02-18
updated_at: 2026-02-18
---

# Docker (images & best practices for platform engineers)

## TL;DR (BLUF)
- Docker builds and runs container images used as the unit of packaging for applications. For platform engineers, understanding image creation, layering, security, and CI/CD integration (including Go build optimizations) is essential for reproducible deployments on Kubernetes.

## Definition
**What it is:** Docker is a tooling and runtime ecosystem around containers and images, enabling packaging of applications and their dependencies into portable artifacts.
**Key terms:** image, container, Dockerfile, layers, registry, multistage build, distroless, OCI.

## Why it matters
- Containers are the deployment unit for Kubernetes. Efficient images reduce startup time, attack surface, and operational costs. For Go services, static binaries enable minimal base images and fast startup — important for autoscaling and platform efficiency.

## Scope & Non-goals
**In scope:** Dockerfile best practices, multi-stage builds, small images for Go, image scanning, signing, registries, CI integration.
**Out of scope:** container runtimes internals (runc/CNI internals), though links to deeper topics can be added.

## Mental model / Intuition
- Think of an image as a layered filesystem snapshot produced by Dockerfile instructions. Each RUN/COPY/ADD creates a layer; smaller layers and fewer layers lead to more cacheable and efficient builds.

## Decision rules (When to use / When not to use)
### Use Docker when
- You need reproducible, portable packaging across environments and an image that runs on Kubernetes or other container platforms.
### Avoid Docker when
- You need raw VM-level isolation (use VMs) or when your CI/CD policy mandates alternative build systems (but Docker-compatible OCI images still work).

## How I would use it (practical)
- **Context:** Build a Go microservice for Kubernetes.
- **Steps:**
  1. Use multi-stage Dockerfile: build Go binary in builder stage (`golang:1.20-alpine`) and copy binary into small runtime image (`gcr.io/distroless/static` or `alpine` if needed).
  2. Pin base image versions and minimize layers.
  3. Scan images with vulnerability scanner in CI and sign images before pushing to registry.
  4. Push to private registry with immutability by tag/sha and deploy via GitOps.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** portability, standard runtime model, wide ecosystem (registries, scanners, builders).
- **Cons:** image bloat, possible security misconfigurations, and need for registry governance.
### Alternatives
- **Buildpacks / distros / containerless platforms:** simplify image creation for some languages; still produce OCI images.

## Failure modes & Pitfalls
- Large images increase pull time and slow rollouts.
- Running as root in container increases security risk.
- Unpinned base images cause non-reproducible builds.

## Observability (How to detect issues)
**Indicators:** slow image pulls, frequent OOMs due to oversized images, security scanner alerts, high container startup latency.

## Implementation notes (cheat-sheet)
- Dockerfile best practices for Go:
  - Use multi-stage builds.
  - Build static binary: `CGO_ENABLED=0 go build -o /app`.
  - Use small runtime image: `gcr.io/distroless/static` or `scratch` when possible.
  - Avoid secrets in images; inject via Secrets/Env at runtime.

**Example Dockerfile (Go)**
```dockerfile
# builder
FROM golang:1.20-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /app ./cmd/myservice

# runtime
FROM gcr.io/distroless/static
COPY --from=builder /app /app
ENTRYPOINT ["/app"]
```

## Exercises / Practice
1) Convert a multi-stage Dockerfile for a Node app into a smaller build using an official runtime image.
2) For a Go service, create a Dockerfile that builds a static binary and explain the security advantages of distroless images.
3) Add image scanning and signing steps to a CI pipeline (describe tools and stages).

## Common anti-patterns
- **Anti-pattern:** Embedding secrets in image layers.
  - **Why it’s bad:** Secrets become part of image history and can be retrieved from layers.
  - **Better approach:** Use runtime Secrets (Kubernetes Secret, Vault), and avoid `RUN echo secret` in Dockerfile.

## Interview readiness
### “Explain it like I’m teaching”
- Docker packages apps as images built from Dockerfiles. For Go apps, multi-stage builds produce small static binaries that make secure, fast images perfect for Kubernetes deployments.

### Trap questions (with answers)
1) **Q:** Why use multi-stage builds?  
   - **A:** To separate build dependencies from runtime image, producing smaller, secure runtime images.
2) **Q:** When would you use `scratch` vs `distroless` for a Go binary?  
   - **A:** Use `scratch` for absolute minimal image if the binary needs no system libraries and debugging is not required; use `distroless` for minimal base with some system conveniences and better defaults.
3) **Q:** How does Docker relate to Kubernetes images and why is image immutability important?  
   - **A:** Kubernetes deploys OCI images; immutable images (by digest) ensure reproducible deployments and safer rollbacks.

### Quick self-check (5 items)
- [ ] I can write a multi-stage Dockerfile for a Go service from memory.
- [ ] I can explain why image size matters for scaling and rollouts.
- [ ] I can name 2 image scanning/signing tools.
- [ ] I can explain why containers should run non-root.
- [ ] I can describe how CI integrates with registries and GitOps.

## Links (NO duplication)
### Prerequisites
- [Kubernetes (platform basics)](kubernetes.md)

### Related topics
- [Container images & registries (TODO)](TODO)

## Notes / Inbox (optional)
- Consider adding CI examples for GitHub Actions or GitLab CI to build, scan, sign, and push images.
