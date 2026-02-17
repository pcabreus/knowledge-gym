---
id: sli-slo-sla-mcq-1
topic: ../topics/operations/service-level-objective.md
kind: mcq
difficulty: medium
timebox_minutes: 20
created_at: 2026-01-23
updated_at: 2026-01-23
---

# SLI/SLO/SLA — MCQ 1

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Q1) What best describes a Service Level Indicator (SLI)?
A) A measurable user-centric metric such as % of requests under 300ms.
B) A contractual promise with penalties for downtime.
C) A deployment strategy that reduces blast radius.
D) A database index that speeds up queries.

Q2) What best describes a Service Level Objective (SLO)?
A) The target threshold for an SLI over a time window.
B) The legal contract between provider and customer.
C) A log aggregation technique.
D) A cache invalidation policy.

Q3) What best describes a Service Level Agreement (SLA)?
A) An external contractual guarantee with remedies if missed.
B) A metric that measures reliability.
C) An internal objective set only by engineers.
D) A circuit breaker configuration.

Q4) Which statement is correct about SLOs and SLAs?
A) SLOs are internal targets; SLAs are external contracts.
B) SLAs are always stricter than SLOs.
C) SLOs are only for ops teams.
D) SLAs replace the need for SLIs.

Q5) You measure “% of checkout requests under 400ms.” What is this?
A) Service Level Indicator (SLI).
B) Service Level Objective (SLO).
C) Service Level Agreement (SLA).
D) Error budget.

Q6) What is an error budget?
A) The allowed amount of SLO violations in the window.
B) The total number of incidents per year.
C) The maximum allowed CPU usage.
D) A legal penalty clause in a contract.

Q7) Your SLO is 99.9% availability over 30 days. Which is the closest error budget in downtime?
A) ~43 minutes.
B) ~4.3 hours.
C) ~4.3 minutes.
D) Zero minutes.

Q8) When should you tighten SLOs?
A) When user expectations rise and you can sustain higher reliability.
B) After a single brief incident with no user impact.
C) When you cannot measure the SLI well.
D) When you want to ship faster and accept more risk.

Q9) Which is a common SLI pitfall?
A) Using CPU utilization as the primary SLI.
B) Measuring p95 latency per endpoint.
C) Tracking error rate by request type.
D) Using good events/total events.

Q10) Which is the most defensible relationship between SLA and SLO?
A) SLO should be tighter than SLA to provide buffer.
B) SLA should always be tighter than SLO.
C) They should be identical to avoid confusion.
D) You don’t need SLOs if you have an SLA.

## Answer format (what “good” looks like)
- Correct letter per question
- Short justification focused on definitions and decision rules

## Answer key / Rubric
### Correct answer (for MCQ)
Q1)
- Correct: A
- Why correct: SLIs are measurable, user-centric signals like latency or availability.
- Why A is wrong: N/A
- Why B is wrong: That describes an SLA.
- Why C is wrong: Deployment strategies are unrelated to SLIs.
- Why D is wrong: Indexes are not reliability indicators.

Q2)
- Correct: A
- Why correct: SLOs are targets for SLIs over a defined window.
- Why A is wrong: N/A
- Why B is wrong: That is an SLA.
- Why C is wrong: Log aggregation is not an objective.
- Why D is wrong: Cache policies are unrelated.

Q3)
- Correct: A
- Why correct: SLAs are contractual commitments with remedies.
- Why A is wrong: N/A
- Why B is wrong: That is an SLI.
- Why C is wrong: That is an SLO.
- Why D is wrong: Circuit breakers are not contracts.

Q4)
- Correct: A
- Why correct: SLOs guide internal targets; SLAs define external commitments.
- Why A is wrong: N/A
- Why B is wrong: SLAs are usually looser than SLOs.
- Why C is wrong: SLOs are shared by product and engineering.
- Why D is wrong: You still need SLIs to measure performance.

Q5)
- Correct: A
- Why correct: It is a measurable indicator of user experience.
- Why A is wrong: N/A
- Why B is wrong: An SLO would set a target like “95% under 400ms.”
- Why C is wrong: SLAs are external contracts.
- Why D is wrong: Error budgets are derived from SLOs.

Q6)
- Correct: A
- Why correct: Error budgets are the allowed amount of failure within an SLO window.
- Why A is wrong: N/A
- Why B is wrong: Incidents are not the same as SLO violations.
- Why C is wrong: CPU is not an error budget.
- Why D is wrong: Contract penalties are SLA terms.

Q7)
- Correct: A
- Why correct: 0.1% of 30 days is about 43.2 minutes.
- Why A is wrong: N/A
- Why B is wrong: That is 1% downtime.
- Why C is wrong: That is 0.01% downtime.
- Why D is wrong: 100% uptime is not the SLO.

Q8)
- Correct: A
- Why correct: Tighten only when you can sustain it and user expectations demand it.
- Why A is wrong: N/A
- Why B is wrong: Overreacting to a single incident is risky.
- Why C is wrong: You need good measurement first.
- Why D is wrong: Tighter SLOs reduce velocity, not increase it.

Q9)
- Correct: A
- Why correct: CPU is internal and may not reflect user experience.
- Why A is wrong: N/A
- Why B is wrong: Per-endpoint p95 is a good SLI.
- Why C is wrong: Error rate by request type is a good SLI.
- Why D is wrong: Good/total events is a standard SLI formulation.

Q10)
- Correct: A
- Why correct: SLOs are internal and typically stricter than SLAs to provide buffer.
- Why A is wrong: N/A
- Why B is wrong: SLAs are usually looser than SLOs.
- Why C is wrong: Identical targets remove buffer and increase breach risk.
- Why D is wrong: SLIs/SLOs are still needed for internal management.

## Common mistakes
- Treating SLIs as targets rather than metrics.
- Confusing SLOs with SLAs.
- Ignoring error budgets in release decisions.
