I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.

Q1) What is the primary purpose of a Kubernetes Operator?
A) Provide a UI for cluster administration
B) Encode operational knowledge as a controller managing CRDs and automating lifecycle tasks
C) Replace the kube-scheduler for custom node placement
D) Act as a network proxy between pods
Correct: B
Why correct: Operators extend Kubernetes with CRDs and controllers to automate application-specific operations (provisioning, backups, upgrades) using reconciliation loops.
Why A is wrong: Operators are controllers, not user interfaces.
Why C is wrong: The kube-scheduler handles node placement; operators manage application lifecycle.
Why D is wrong: Operators do not function as network proxies.

Q2) Which tool/library is most commonly used to write production-grade Operators in Go?
A) controller-runtime (kubebuilder / Operator SDK)
B) kubectl plugins framework
C) Helm templates
D) Terraform provider SDK
Correct: A
Why correct: `controller-runtime` and scaffolding via kubebuilder/Operator SDK are the standard Go libraries/tooling for controllers and Operators.
Why B is wrong: kubectl plugins extend CLI only, not controller frameworks.
Why C is wrong: Helm is for templated installs; it is not a controller runtime.
Why D is wrong: Terraform providers manage infra but do not integrate with Kubernetes reconciliation loop idioms.

Q3) What is a reconcile loop in an Operator responsible for?
A) Continuously ensuring observed state matches desired state defined in the CR
B) Scheduling nodes for cluster autoscaling
C) Serving HTTP traffic for the application
D) Encrypting Secrets at rest
Correct: A
Why correct: The reconcile loop reads the custom resource, computes desired child resources, and applies changes to converge state.
Why B is wrong: Node autoscaling is separate (cluster-autoscaler); reconcile focuses on resource state.
Why C is wrong: Operators don't serve app traffic; they manage infrastructure.
Why D is wrong: Encryption of Secrets is an API/storage concern, not the generic reconcile loop's job.

Q4) Why are finalizers used in CRD design?
A) To prevent any deletion of resources permanently
B) To allow the controller to perform cleanup actions before the resource is removed
C) To speed up garbage collection of pods
D) To grant RBAC permissions to users
Correct: B
Why correct: Finalizers block resource deletion until the controller performs necessary cleanup (external resource teardown) and removes the finalizer.
Why A is wrong: Finalizers don't prevent deletion forever — they enable controlled cleanup.
Why C is wrong: Finalizers don't affect pod GC directly.
Why D is wrong: Finalizers are not related to RBAC.

Q5) What is the recommended practice for long-running tasks (e.g., backups) in a reconcile loop?
A) Perform the entire backup synchronously inside reconcile
B) Offload the work to a Kubernetes Job or child controller and keep reconcile fast and idempotent
C) Block reconcile and wait indefinitely for completion
D) Exit reconcile without recording intent
Correct: B
Why correct: Reconcile should be quick and idempotent; long tasks should be delegated to Jobs or separate controllers to avoid slow reconciles and API saturation.
Why A is wrong: Long synchronous tasks block the controller and can lead to retries and instability.
Why C is wrong: Blocking indefinitely is unsafe and will impact controller health.
Why D is wrong: Not recording intent loses visibility and control over the operation.

Q6) In a multi-replica operator deployment, what ensures only one replica acts on a given resource at a time?
A) Pod anti-affinity
B) Leader election mechanism provided by controller-runtime
C) Using `kubectl port-forward`
D) Disabling replicas in the deployment manifest
Correct: B
Why correct: Leader election ensures a single active controller instance performs reconciliation for shared resources, preventing conflicts.
Why A is wrong: Anti-affinity controls scheduling, not active replica leadership.
Why C is wrong: Port-forward is for local access; irrelevant to leader election.
Why D is wrong: Disabling replicas would reduce availability; leader election allows HA with safe single-actor behavior.

Q7) Which testing technique is recommended for controller logic written in Go?
A) Manual testing only on production cluster
B) Unit tests for reconcile with fake clients, and integration tests using `envtest` (api-server control plane)
C) Only end-to-end tests in prod can validate behavior
D) UI-driven tests
Correct: B
Why correct: Controller code benefits from unit tests (fake clients) and integration tests via envtest to run controller behavior against a test API server.
Why A is wrong: Manual prod testing is unsafe and insufficient.
Why C is wrong: E2E in prod alone is risky; layered tests ensure safety.
Why D is wrong: Operators are controller logic; UI tests don't exercise reconcile correctness.

Q8) What is a common cause of infinite reconcile loops in Operators?
A) Using status subresource for updates
B) Updating fields in the spec during reconcile instead of status, causing constant diff
C) Applying child resources idempotently
D) Using controller-runtime's `client.Status()` API correctly
Correct: B
Why correct: Modifying spec fields or annotations that the reconcile compares to desired state can create persistent diffs that retrigger reconcile loops.
Why A is wrong: Using status subresource is best practice for status updates, preventing spec changes from causing loops.
Why C is wrong: Idempotent child updates prevent unnecessary reconciliation churn.
Why D is wrong: Proper use of Status API is safe and recommended.

Q9) Which RBAC rule is essential for an Operator managing CRDs to update the status subresource?
A) `get` on pods
B) `update` on `<crd-plural>/status`
C) `list` on nodes
D) `create` on namespaces
Correct: B
Why correct: Updating status requires permission to `update` the status subresource of the CRD.
Why A is wrong: `get` on pods is unrelated unless the operator manages pods.
Why C is wrong: Listing nodes is unrelated.
Why D is wrong: Creating namespaces is not required for status updates.

Q10) What is the typical packaging/installation method for Operators in an enterprise Kubernetes platform?
A) Using `kubectl run` per operator pod
B) Operator Lifecycle Manager (OLM) or Helm charts to install CRDs and operator deployments
C) Copying binaries to all nodes manually
D) Running as a kubelet plugin
Correct: B
Why correct: OLM and Helm are common ways to install operators and CRDs in a cluster, providing lifecycle management and upgrades.
Why A is wrong: `kubectl run` is insufficient for operator lifecycle and CRD installation.
Why C is wrong: Manually copying binaries to nodes bypasses container orchestration.
Why D is wrong: Operators are controllers in Kubernetes, not kubelet plugins.
