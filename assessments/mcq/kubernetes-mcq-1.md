I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.

Q1) What is the primary role of a Kubernetes `Deployment`?
A) Store persistent data for Pods
B) Ensure a desired number of Pod replicas and manage rolling updates
C) Provide a stable virtual IP for a set of Pods
D) Enforce network policies between namespaces
Correct: B
Why correct: A `Deployment` is a controller that maintains the desired state for a set of Pods (replica count, template) and manages rolling updates and rollbacks.
Why A is wrong: Persistent data is handled by PersistentVolumes/PersistentVolumeClaims and StatefulSets, not Deployments.
Why C is wrong: A Service provides a stable virtual IP; Deployments manage Pods but do not give stable IPs.
Why D is wrong: Network policies enforce traffic rules; Deployments don't manage networking policies.

Q2) Which `Service` type exposes a Service externally via a cloud provider's load balancer?
A) ClusterIP
B) NodePort
C) LoadBalancer
D) ExternalName
Correct: C
Why correct: `LoadBalancer` requests a cloud provider provision an external load balancer to expose the Service externally.
Why A is wrong: `ClusterIP` is internal-only and not exposed externally by default.
Why B is wrong: `NodePort` opens a port on each node, but doesn't provision a cloud LB automatically.
Why D is wrong: `ExternalName` maps the Service to an external DNS name, not a cloud load balancer.

Q3) Which probe should be used to keep traffic away from a Pod until the application is ready to serve requests?
A) Liveness probe
B) Readiness probe
C) Startup probe
D) HealthService probe
Correct: B
Why correct: Readiness probes indicate when a Pod is ready to receive traffic; failing readiness removes it from Service endpoints without restarting the container.
Why A is wrong: Liveness probes indicate when to restart a container; they don't control traffic routing during startup.
Why C is wrong: Startup probes are for very slow-starting containers to avoid false liveness failures; readiness still controls traffic.
Why D is wrong: `HealthService` is not a Kubernetes probe type.

Q4) When would you choose a `StatefulSet` instead of a `Deployment`?
A) For stateless web APIs that scale horizontally
B) For workloads that need stable network IDs and stable storage per replica
C) When you need a stable ClusterIP Service
D) For running Daemon processes on every node
Correct: B
Why correct: `StatefulSet` provides stable identities, ordered deployment, and stable storage for each replica — suitable for databases and systems needing stable identity.
Why A is wrong: Stateless web APIs are best with Deployments.
Why C is wrong: ClusterIP is a Service type unrelated to StatefulSet choice; many controllers use ClusterIP.
Why D is wrong: DaemonSet ensures one Pod per node; StatefulSet does not.

Q5) What is the best reason to set resource `requests` for a Pod?
A) To allow unlimited resource consumption
B) To improve scheduling by letting the scheduler place Pods on nodes with sufficient capacity
C) To configure HPA scaling thresholds directly
D) To automatically evict unrelated pods
Correct: B
Why correct: `requests` tell the scheduler how much resource the Pod needs, enabling bin-packing and correct placement decisions.
Why A is wrong: Requests do not grant unlimited resources; they are minimum guaranteed allocation.
Why C is wrong: HPA uses metrics (like CPU usage) not directly the `requests` field; though requests affect utilization percentages.
Why D is wrong: Eviction is influenced by node pressure and QoS classes; requests alone do not evict unrelated pods.

Q6) Which command combination is safest for previewing changes before applying them?
A) `kubectl apply -f deploy.yaml` then `kubectl get pods`
B) `kubectl delete -f deploy.yaml` then `kubectl apply -f deploy.yaml`
C) `kubectl diff -f deploy.yaml` then `kubectl apply -f deploy.yaml`
D) `kubectl edit deployment/my-api`
Correct: C
Why correct: `kubectl diff` shows server-side differences before applying, allowing review; then `kubectl apply` performs the declarative change.
Why A is wrong: Applying without preview risks unintended changes.
Why B is wrong: Deleting then applying can cause downtime and loss of ephemeral state; it's not a preview.
Why D is wrong: `kubectl edit` edits live resource directly and is not a preview-first workflow.

Q7) What is a common anti-pattern when using readiness and liveness probes?
A) Using readiness to delay traffic until app initializes
B) Using identical checks for readiness and liveness causing unnecessary restarts
C) Using readiness to remove the Pod from Service endpoints
D) Using liveness to detect and recover from deadlocks
Correct: B
Why correct: Using identical checks can cause the liveness probe to restart a Pod that's not yet ready (or intended to be excluded), leading to flapping. Readiness and liveness should serve distinct purposes.
Why A is wrong: That is good practice.
Why C is wrong: That is the intended use of readiness.
Why D is wrong: That is a valid use of liveness.

Q8) Which is the correct description of a Kubernetes `Pod` networking model?
A) Each container gets its own IP address and network namespace
B) A Pod has a shared network namespace and a single IP for all containers inside it
C) Pods share host networking by default
D) Each Pod routes traffic through an external load balancer
Correct: B
Why correct: Containers in a Pod share the same network namespace and IP, enabling localhost communication between containers.
Why A is wrong: Containers in a Pod do not get separate IPs; they share the Pod IP.
Why C is wrong: Pods do not share host networking by default; hostNetwork must be enabled explicitly.
Why D is wrong: Pods do not route traffic through external LBs by default; Services or Ingress handle that.

Q9) Which HPA (Horizontal Pod Autoscaler) metric is commonly supported out-of-the-box?
A) Disk IOPS
B) CPU utilization
C) Custom business metric (orders/sec) without additional setup
D) Number of open sockets
Correct: B
Why correct: CPU utilization is a standard metric supported by the Kubernetes metrics API and HPA by default.
Why A is wrong: Disk IOPS typically requires custom metrics collection.
Why C is wrong: Custom business metrics require a metrics adapter and additional setup.
Why D is wrong: Open sockets are not a standard HPA metric without custom exporters.

Q10) What is the primary benefit of using `kubectl port-forward` during debugging?
A) It creates a permanent external service for traffic
B) It allows secure, temporary local access to a Pod or Service without changing cluster networking
C) It scales the service to more replicas
D) It configures Ingress rules
Correct: B
Why correct: `kubectl port-forward` creates a secure temporary tunnel from localhost to a Pod/Service, useful for testing endpoints without modifying cluster network configuration.
Why A is wrong: It is temporary and local-only, not a permanent external service.
Why C is wrong: Port-forwarding does not change replica counts.
Why D is wrong: It doesn't modify Ingress configuration.
