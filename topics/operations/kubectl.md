---
id: kubectl
title: "kubectl (CLI quick reference and practice)"
type: skill
status: learning
importance: 70
difficulty: easy
tags: [kubectl, kubernetes, ops, troubleshooting]
canonical: true
aliases: []
created_at: 2026-02-18
updated_at: 2026-02-18
---

# kubectl (CLI quick reference and practice)

## TL;DR (BLUF)
- `kubectl` is the Kubernetes command-line tool for inspecting and managing cluster resources; mastering a few commands (get/describe/logs/exec/apply/rollout) lets you debug and operate workloads effectively.
- Practice scenarios: debugging CrashLoopBackOff, checking readiness probe failures, rolling back a bad deploy, port-forwarding a service for local testing.

## Definition
**What it is:** The primary command-line interface to talk to the Kubernetes API server and manipulate resources (Pods, Deployments, Services, ConfigMaps, etc.).
**Key terms:** context, namespace, resource, manifest, apply, rollout, port-forward, exec.

## Why it matters
- `kubectl` is the day-to-day operational tool for developers and SREs to inspect cluster state, perform rollouts, and debug live systems. Knowing the right commands reduces mean-time-to-detect (MTTD) and mean-time-to-repair (MTTR).

## Scope & Non-goals
**In scope:** common `kubectl` commands, best-practice workflows, quick debugging patterns, and short practice exercises.
**Out of scope:** deep API extensions, writing custom controllers, or kubectl plugin development (consider as TODO).

## Mental model / Intuition
- `kubectl` is a thin client speaking HTTP to the API server. Most commands are read-only (get/describe/logs) or declarative (apply/delete). Treat it as your control plane window — be cautious with destructive commands.

## Decision rules (When to use / When not to use)
### Use it when
- You need to inspect or mutate resources quickly, debug a pod, or deploy a small manifest change.
### Avoid it when
- You need repeatable, audited infra changes — prefer GitOps or CI pipelines for production changes.

## How I would use it (practical)
- **Context:** Debugging a service that returns 500s in production.
- **Steps:**
  1. `kubectl get pods -l app=my-api -o wide` → find failing pods and nodes.
  2. `kubectl describe pod <pod>` → inspect events and probe failures.
  3. `kubectl logs <pod> -c my-api` → check application logs.
  4. If needed, `kubectl exec -it <pod> -- /bin/sh` → run lightweight checks from inside the pod.
  5. If rollout caused issue: `kubectl rollout status deployment/my-api` and `kubectl rollout undo deployment/my-api`.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** immediate feedback, flexible, powerful for on-call/debugging.
- **Cons:** manual actions can lead to inconsistent state; overuse in prod bypasses CI/GitOps safeguards.
### Alternatives
- **GitOps / CI pipelines:** automates and audits changes (preferred for production).
- **kubectl plugins / wrappers:** for repetitive tasks and safety checks.

## Failure modes & Pitfalls
- Running `kubectl delete` without namespace/context → accidentally remove resources in wrong cluster/namespace.
- Using `kubectl edit` for complex changes -> merge conflicts or accidental truncation.
- Relying on `kubectl logs` alone when pods restart quickly → miss logs before rotation; use log aggregation.

## Observability (How to detect issues)
**Metrics/indicators:** frequent `kubectl get pods` restarts, CrashLoopBackOff, readiness failures, high restart counts.
**Logs:** API server error messages when apply fails, events showing failed mounts or probe failures.

## Implementation notes (cheat-sheet)
- Common commands:
  - Cluster & nodes: `kubectl cluster-info`, `kubectl get nodes -o wide`
  - Inspect: `kubectl get pods --all-namespaces`, `kubectl get deployments`, `kubectl get svc`
  - Describe: `kubectl describe pod <pod>`, `kubectl describe node <node>`
  - Logs: `kubectl logs <pod> [-c <container>]`, `kubectl logs -f <pod>`
  - Exec/debug: `kubectl exec -it <pod> -- /bin/sh`
  - Apply: `kubectl apply -f <file.yaml>`; Diff: `kubectl diff -f <file.yaml>`
  - Rollouts: `kubectl rollout status deployment/<name>`, `kubectl rollout undo deployment/<name>`
  - Scale: `kubectl scale deployment/<name> --replicas=5`
  - Port-forward: `kubectl port-forward svc/<service-name> 8080:80`
  - Contexts/namespaces: `kubectl config get-contexts`, `kubectl config use-context <ctx>`, `kubectl config set-context --current --namespace=<ns>`

## Mini examples
```bash
# Show pods in namespace 'prod'
kubectl get pods -n prod -o wide

# Follow logs from a pod
kubectl logs -f my-api-5d9c8f6d7b-abcde -c my-api

# Exec into a pod
kubectl exec -it my-api-5d9c8f6d7b-abcde -- /bin/sh

# Apply change and view rollout
kubectl apply -f deploy.yaml
kubectl rollout status deployment/my-api
```

## Exercises / Practice (do these mentally or in a sandbox cluster)
1) Debug CrashLoopBackOff: list steps and commands you run to find the root cause; include `describe`, `logs`, events.
2) Simulate a bad deployment: apply a manifest that fails readiness probe, then practice `rollout undo` and explain why readiness probes helped.
3) Port-forward the payments Service to localhost and curl `/health` locally.
4) Demonstrate scaling a deployment from 2 → 6 replicas and show how to verify pods are scheduled across nodes.
5) Use `kubectl diff` followed by `kubectl apply` to show a safe change workflow.

## Common anti-patterns
- **Anti-pattern:** Using `kubectl` as the only deployment mechanism for production.
  - **Why it’s bad:** lacks audit trail and repeatability.
  - **Better approach:** use `kubectl` for debugging and CI/GitOps for deployment.

## Interview readiness
### “Explain it like I’m teaching”
- `kubectl` is the CLI to query and manipulate Kubernetes resources: `get` to list, `describe` for events, `logs` for container output, `exec` to run commands, and `apply`/`rollout` for deployments. Use declarative manifests via `apply` and keep manual `kubectl` usage for debugging and emergencies.

### Trap questions (with answers)
1) **Q:** What’s the difference between `kubectl get` and `kubectl describe`?
   - **A:** `get` lists resources and shows concise fields; `describe` shows detailed resource state including events and recent failure reasons.
2) **Q:** When is `kubectl apply` preferable to `kubectl replace`?
   - **A:** `apply` is declarative and merges changes with the server-side last-applied config; `replace` overwrites the resource and can remove fields unintentionally.
3) **Q:** Why prefer readiness probes over liveness probes for blocking traffic during startup?
   - **A:** Readiness controls when the pod receives traffic; failing readiness keeps the pod alive but out of service so it can finish initialization; liveness causes restarts.

### Quick self-check (5 items)
- [ ] I can list 6 essential `kubectl` commands from memory.
- [ ] I can explain how to inspect probe failures and pod events.
- [ ] I can roll back a failed deployment with `kubectl rollout undo`.
- [ ] I can port-forward a Service and test an endpoint locally.
- [ ] I can switch contexts and namespaces safely.

## Links (NO duplication)
### Prerequisites
- [Kubernetes (platform basics)](../operations/kubernetes.md)

### Related topics
- [Rolling deployments](../operations/rolling-deployments.md)
- [Service Discovery](../operations/service-discovery.md)
- [Observability basics](../operations/observability-basics.md)

## Notes / Inbox (optional)
- Consider adding a one-page printable `kubectl` cheat-sheet and a short KinD script with example manifests for hands-on practice.
