---
id: kubernetes
title: "Kubernetes (platform basics)"
type: technology
status: learning
importance: 85
difficulty: medium
tags: [kubernetes, platform, containers, operations]
canonical: true
aliases: [k8s]
created_at: 2026-02-18
updated_at: 2026-02-18
---

# Kubernetes (platform basics)

## TL;DR (BLUF)
- Kubernetes orchestrates containers for scalable, self-healing, declarative deployments; it separates the runtime unit (Pod) from controllers (Deployment/StatefulSet) and stable access (Service/Ingress).
- Use it to run production microservices with autoscaling, rolling updates, and health-driven routing; costs: operational complexity and added failure modes.

## Definition
**What it is:** An open-source container orchestration system that manages scheduling, networking, storage, and lifecycle of containers across a cluster of machines.
**Key terms:** Pod, Deployment, StatefulSet, Service, Ingress, ConfigMap, Secret, HPA, liveness/readiness probes, kubelet, control plane.

## Why it matters
- Provides primitives for running containerized workloads reliably: desired-state reconciliation, autoscaling, rolling updates, and service discovery.
- Enables teams to standardize deployments, decouple infrastructure from app code, and operate at higher throughput with predictable behavior.

## Scope & Non-goals
**In scope:** core Kubernetes concepts and practical interview-ready answers for platform conversations.
**Out of scope / NOT solved by this:** running a managed control plane (EKS/GKE/AKS) operational playbooks and deep CNI internals (link to network layers).

## Mental model / Intuition
- Think of Kubernetes as a control loop system: you declare the desired state (manifests) and controllers continuously reconcile the cluster to match it.
- Pods are ephemeral runtimes; controllers (Deployment/StatefulSet/DaemonSet) ensure continuity and desired count; Services provide stable networking.

## Decision rules (When to use / When not to use)
### Use it when
- You need automated scheduling, horizontal scaling, rolling upgrades, or standardized platform primitives across teams.
- You run many services that benefit from shared platform features (ingress, service mesh, centralized observability).
### Avoid it when
- Your app surface is small, ops budget is tiny, or you can use a PaaS or serverless offering that reduces operational burden.

## How I would use it (practical)
- **Context:** Deploying a stateless API and a background worker.
- **Steps:** build small container images → write Deployment for each service (replicas, resource requests/limits) → expose via ClusterIP Service → add Ingress for external routing → define liveness/readiness probes → add HPA based on CPU/custom metrics → use ConfigMaps/Secrets for config.
- **What success looks like:** zero-downtime releases with rolling updates, autoscaling under load, and clear health metrics (pod restarts, readiness gating).

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** declarative, extensible, standard ecosystem (Helm, operators, CNI, CSI).
- **Cons / Risks:** operational complexity, version upgrades, control-plane availability, noisy neighbor effects if resources not isolated.
### Alternatives
- **PaaS / FaaS (e.g., Cloud Run, Lambda):** simpler ops; better when you want minimal infra work.
- **Managed Kubernetes (EKS/GKE/AKS):** reduces control-plane ops but still requires worker/infra ops.

## Failure modes & Pitfalls
- Misconfigured resource requests/limits → node OOM/kube-scheduler instability.
- Readiness vs liveness confusion → traffic routed to containers that aren’t ready or restarted unnecessarily.
- Overloaded control plane during large deploys → API throttling / slow reconciliation.
- Stateful workloads placed without PersistentVolume planning → data loss or incorrect access modes.

## Observability (How to detect issues)
**Metrics:** pod CPU/memory, pod restart count, replica available vs desired, API server latency, scheduler queue length, HPA scale events.
**Logs:** kubelet and controller-manager logs, pod logs (application), CSI/CNI plugin errors.
**Traces:** placement/reconciliation times for controllers, request paths through Ingress/Service Mesh.
**Alerts:** pod crashlooping > N, high node allocatable pressure, HPA flapping, persistent volume provisioning failures.

## Implementation notes (if applicable)
**Checklist**
- [ ] Define resource requests & limits for pods.
- [ ] Add liveness/readiness probes that reflect real readiness.
- [ ] Use ConfigMap/Secret for external config and credentials.
- [ ] Use Deployment for stateless, StatefulSet for stateful with stable identity.
- [ ] Use rolling updates and readiness gating for zero-downtime.

**Security / Compliance notes**
- Run pods with least privilege (`runAsNonRoot`, PodSecurityPolicies / PSP alternatives), network policies to restrict traffic, and encrypt Secrets at rest when possible.

**Performance notes**
- Right-size requests to improve bin-packing; set limits to prevent noisy neighbors; use vertical/horizontal autoscaling with sensible cooldowns.

**Operational notes**
- Keep kube-apiserver available (HA control plane), monitor etcd health and backups, practice cluster upgrades on staging.

## Mini example (YAML)
```yaml
# Deployment (stateless)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-api
  template:
    metadata:
      labels:
        app: my-api
    spec:
      containers:
        - name: my-api
          image: my-api:latest
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5

# Service (stable access)
---
apiVersion: v1
kind: Service
metadata:
  name: my-api-svc
spec:
  selector:
    app: my-api
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

## Common anti-patterns
- **Anti-pattern:** Running databases in ephemeral Pods without StatefulSet+PVC.
  - **Why it’s bad:** data loss and ordering issues.
  - **Better approach:** use managed DBs or StatefulSets with stable PVs and backups.

## Interview readiness
### “Explain it like I’m teaching”
- Kubernetes manages containers across machines: Pods run containers, Deployments ensure a desired set of Pods exists, and Services provide a stable network endpoint to reach those Pods.

### Trap questions (with answers)
1) **Q:** What’s the difference between a Pod and a Deployment?
   - **A:** Pod is the runtime unit; Deployment is a controller that ensures a desired number of Pods exist and manages updates/rollbacks.
2) **Q:** When would you use a StatefulSet instead of a Deployment?
   - **A:** When you need stable network IDs and persistent storage per replica (databases, Kafka brokers) and ordered start/stop semantics.
3) **Q:** Why are readiness probes important for rolling updates?
   - **A:** Readiness probes prevent traffic from reaching a Pod until it’s ready; during rolling updates, only ready Pods receive traffic, enabling zero-downtime.

### Quick self-check (5 items)
- [ ] I can explain Pod vs Deployment vs Service in one sentence each.
- [ ] I can sketch a Deployment + Service YAML from memory.
- [ ] I can list when to choose StatefulSet vs Deployment.
- [ ] I can explain readiness vs liveness probes and why they differ.
- [ ] I can name at least 3 operational alerts for a cluster.

## Links (NO duplication)
### Prerequisites
- [Networking basics](../operations/networking-basics.md)
- [Sidecar Pattern](../operations/sidecar.md)

### Related topics
- [Blue/green deployments](../operations/blue-green-deployments.md)
- [Canary deployments](../operations/canary-deployments.md)
- [Rolling deployments](../operations/rolling-deployments.md)
- [Service Discovery](../operations/service-discovery.md)

### Compare with
- [Terraform](../operations/terraform.md) — Terraform defines infra; Kubernetes manages runtime workloads.

## Notes / Inbox (optional)
- Consider adding a short guide for common `kubectl` commands and a one-page upgrade checklist.
