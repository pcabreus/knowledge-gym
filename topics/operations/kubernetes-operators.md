---
id: kubernetes-operators
title: "Kubernetes Operators (custom controllers)"
type: technology
status: learning
importance: 78
difficulty: medium
tags: [kubernetes, operators, controller, go, platform]
canonical: true
aliases: [operator]
created_at: 2026-02-18
updated_at: 2026-02-18
---

# Kubernetes Operators (custom controllers)

## TL;DR (BLUF)
- Operators are Kubernetes controllers that extend the API with Custom Resource Definitions (CRDs) and automate operational knowledge (deploy, backup, scale, heal) for complex applications. Most production operators are written in Go using controller-runtime / Operator SDK because Go maps well to Kubernetes APIs and concurrency patterns.

## Definition
**What it is:** An Operator is a Kubernetes controller paired with CRDs that encodes domain-specific operational logic (reconciliation) to manage an application lifecycle automatically.
**Key terms:** Controller, reconciliation loop, CRD (CustomResourceDefinition), Operator SDK, controller-runtime, finalizers, informers.

## Why it matters
- Operators allow platform engineers to capture runbooks as code: automate day-2 operations (backup/restore, scaling, configuration rollouts) and provide higher-level abstractions for app teams.
- They reduce manual toil, improve reliability, and make complex stateful workloads cloud-native and manageable via kubectl and GitOps.

## Scope & Non-goals
**In scope:** CRD design, reconcile patterns, finalizers, leader election, testing operators, and how Go fits into operator development.
**Out of scope / NOT solved by this:** replacing an orchestration platform; operators automate app lifecycle but don't remove the need for monitoring, capacity planning, or proper platform governance.

## Mental model / Intuition
- Think of an Operator as a Unix daemon watching resources and enforcing the desired state: it observes CR objects, calculates the delta, and performs imperative steps (create/update child resources) until the world matches the desired spec.

## Decision rules (When to build an Operator)
### Build an Operator when
- Operational complexity is high (stateful lifecycle, backups, graceful rolling upgrades, complex config changes).
- You need consistent automations across clusters and teams, and want to expose a higher-level API to application owners.
### Avoid building an Operator when
- The operational actions are simple (stateless apps where Deployments + Helm suffice), or you lack capacity to maintain it long-term.

## How I would use it (practical)
- **Context:** Automate Postgres lifecycle (provision, backup, failover).
- **Steps:**
  1. Define a CRD `PostgresCluster` with spec (replicas, storageClass, backupPolicy).
  2. Implement controller reconcile loop: observe `PostgresCluster`, ensure StatefulSet, Services, and backups exist and are configured.
  3. Add finalizers to handle cleanup and async actions.
  4. Write e2e tests and run operator via Operator SDK in a staging cluster.
  5. Package operator as an image and deliver via Helm/OLM.

## How Go is involved (why Go for Operators)
- Kubernetes control plane and client libraries (`client-go`, `controller-runtime`) are native Go libraries; they are mature, efficient, and provide strongly-typed APIs and idiomatic concurrency primitives (goroutines, channels) making reconciliation code straightforward.
- Go binaries are single static executables easy to containerize (small base images like distroless), and performance characteristics (low latency, low GC impact) suit long-running controllers.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** Operators provide powerful automation, integrate tightly with K8s events, and expose clear APIs for app teams.
- **Cons:** Building and operating operators requires maintenance, testing, and lifecycle management; bugs in controllers can cause widespread cluster impact.
### Alternatives
- **Helm charts + CI/CD workflows:** simpler for deployment; less capable for ongoing automation.
- **External automation (Terraform, scripts):** can orchestrate resources but won't get event-driven reconciliation semantics inside the cluster.

## Failure modes & Pitfalls
- Unsafe reconciliation (e.g., destructive updates) can cause data loss — include safety checks and dry-run support.
- Operator race conditions if controllers don't use leader election in multi-replica setups.
- Unbounded retries on transient errors can cause tight crash loops; implement backoff and error classification.

## Observability (How to detect issues)
**Metrics:** reconcile loop duration, error counts, reconcile queue length, number of resources managed.
**Logs:** structured logs per reconciliation with correlation to resource UID and spec hash.
**Traces:** span reconcile execution and downstream API calls.
**Alerts:** high reconcile errors, operator restarts, long reconcile durations.

## Implementation notes (if applicable)
**Checklist**
- [ ] Design CRD with clear spec/status separation.
- [ ] Implement reconcile idempotently.
- [ ] Use controller-runtime for client and manager.
- [ ] Add leader election for HA.
- [ ] Add tests: unit for reconcile, integration with envtest, e2e on KinD.

**Operator lifecycle**
- Develop locally with `operator-sdk` or `kubebuilder`.
- Run with `make run` in dev; build image and deploy to staging for integration tests.

## Mini example (Go reconcile skeleton)
```go
func (r *MyController) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    var cr myv1alpha1.MyResource
    if err := r.Get(ctx, req.NamespacedName, &cr); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 1) compute desired child resources from CR spec
    desired := desiredStateFor(cr)

    // 2) ensure child resources exist and are up-to-date
    if err := r.ensureChildren(ctx, &cr, desired); err != nil {
        return ctrl.Result{RequeueAfter: time.Minute}, err
    }

    // 3) update status
    cr.Status.Ready = true
    if err := r.Status().Update(ctx, &cr); err != nil {
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil
}
```

## Exercises / Practice
1) Design a CRD for a cache cluster (replicas, persistence, eviction policy). Describe reconcile steps for scaling and config change.
2) Implement idempotent reconcile pseudo-logic for ensuring a backup Job runs weekly.
3) Explain how you'd add leader election and why it's needed when running multiple operator replicas.

## Common anti-patterns
- **Anti-pattern:** Operators that perform heavy, long-running synchronous work in reconcile without offloading to Jobs; causes slow reconciles and API saturation.
  - **Better approach:** create Jobs or child controllers for long-running tasks and keep reconcile fast and idempotent.

## Interview readiness
### “Explain it like I’m teaching”
- Operators are controllers plus CRDs that encode operational knowledge. They observe a custom resource, compute desired state, and perform imperative actions to reach that state — enabling automation of complex app lifecycle tasks.

### Trap questions (with answers)
1) **Q:** Why use Go for Operators instead of Python or Node?  
   - **A:** Go has mature Kubernetes libraries (`client-go`, `controller-runtime`), compiles to a single static binary (easy containerization), and its concurrency model fits controller patterns; you can write operators in other languages, but Go is the practical default.
2) **Q:** How do you avoid infinite reconcile loops?  
   - **A:** Reconcile idempotently, compare observed vs desired (use a spec hash), and avoid modifying fields that will re-trigger unchanged reconciles; use status subresource and annotations carefully.
3) **Q:** When should you prefer Helm+hooks vs an Operator?  
   - **A:** Use Helm for templated deploys and simple lifecycle; build an Operator when you need active reconciliation and automation of complex day-2 operations.

### Quick self-check (5 items)
- [ ] I can describe reconcile loop in 30 seconds.
- [ ] I can list when to build an Operator vs use Helm.
- [ ] I can sketch a CRD spec and typical status fields.
- [ ] I can explain leader election, finalizers, and idempotency basics.
- [ ] I can point to controller-runtime and Operator SDK as Go-based toolchains.

## Links (NO duplication)
### Prerequisites
- [Kubernetes (platform basics)](kubernetes.md)

### Related topics
- [StatefulSet](../operations/rolling-deployments.md)
- [Backups & operators patterns (TODO)](TODO)

## Notes / Inbox (optional)
- Consider adding a small KinD example operator and a CI workflow for building/testing operators.
