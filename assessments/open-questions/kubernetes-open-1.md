Questions:
Q1) What is a Pod in Kubernetes and why might you place a sidecar container next to your application container inside the same Pod?
Q2) Explain the roles of `Deployment`, `ReplicaSet`, and `Service` and how they work together to provide availability and stable access.
Q3) Describe a safe rollout strategy for a new API version. What probes and checks would you add to avoid downtime?
Q4) Given a Pod in `CrashLoopBackOff`, list the exact `kubectl` commands you would run and the sequence of checks you would perform to debug the issue.
Q5) When would you choose `StatefulSet` over `Deployment`? Provide a short example scenario and explain the observable differences.
Q6) How do ConfigMaps and Secrets differ in lifecycle, usage, and security considerations? Give examples of what you store in each.
Q7) Design a minimal manifest for a payment-service: 3 replicas, ClusterIP Service, and an Ingress rule routing `/payments` to the service — describe each section and reasoning.
Q8) What are the operational risks of running many small Pods without proper resource requests/limits? How would you detect and mitigate them?
Q9) Explain how Horizontal Pod Autoscaler (HPA) works and what metrics or signals you would use in production beyond CPU.
Q10) Trap question: What happens to ephemeral Pod-local files when a Pod reschedules to another node and how would you design around it?
