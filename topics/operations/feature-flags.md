---
id: feature-flags
title: "Feature Flags (Feature Toggles)"
type: pattern
status: learning
importance: 80
difficulty: medium
tags: [deployment, feature-management, risk-mitigation, decoupling]
canonical: true
aliases: [feature-toggles, feature-switches, conditional-features]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Feature Flags (Feature Toggles)

## TL;DR (BLUF)
- Feature flags decouple deployment from release: code ships, but features activate later.
- Use for gradual rollouts, A/B tests, kill switches, and trunk-based development.
- Key trade-off: flexibility and safety vs. technical debt (old flags accumulate).

## Definition
**What it is:** A software technique that allows enabling or disabling features at runtime without redeploying code, using conditional logic controlled by configuration.  
**Key terms:** toggle, flag state (on/off), toggle types (release/ops/experiment/permission), flag management system, flag cleanup, trunk-based development, dark launching.

## Why it matters
- **Decouples deployment from release:** Ship code to production with features off; activate when ready.
- **Reduces risk:** Enable features for 1% of users first; rollback by flipping flag (instant, no redeploy).
- **Enables A/B testing:** Show version A to 50%, version B to 50%; measure outcomes.
- **Supports trunk-based dev:** Merge incomplete features (behind flag) to main; no long-lived branches.
- **Interview relevance:** Common in system design (gradual rollouts, risk mitigation, testing in production).

## Scope & Non-goals
**In scope:**
- Feature flag types and use cases.
- When to use flags vs. other release strategies.
- Flag management and cleanup.

**Out of scope / NOT solved by this:**
- Specific tools (LaunchDarkly, Split.io—though concepts apply).
- Complex experimentation platforms (A/B test statistical significance).
- Infrastructure feature flags (use IaC feature toggles instead).

## Mental model / Intuition
Think of feature flags as **light switches for your code**:
- You install the wiring (code) but keep the switch off (flag disabled).
- You can flip the switch on for one room (1% of users) to test.
- If the light flickers (bugs), flip it back off instantly (no electrician needed).
- Eventually you remove the switch (cleanup flag after full rollout).

```go
// Light switch analogy in code
func HandleRequest(ctx context.Context, user User) {
    if featureFlags.IsEnabled("new-checkout-flow", user) {
        // New wiring (code)
        return newCheckoutFlow(ctx, user)
    }
    // Old wiring (existing code)
    return oldCheckoutFlow(ctx, user)
}
```

## Decision rules (When to use / When not to use)
### Use it when
- Deploying risky features (enable for 1% first, then 100%).
- Need instant rollback (toggle off, no redeploy).
- Running A/B tests or experiments (show variant to subset of users).
- Practicing trunk-based development (merge incomplete work behind flag).
- Building kill switches (disable expensive feature during incident).
- Coordinating releases across services (enable feature when all services ready).

### Avoid it when
- Feature is simple and low-risk (adds unnecessary complexity).
- You can't commit to flag cleanup (creates long-term technical debt).
- Decision is static (use config file, not flag).
- Feature is a one-time migration (just deploy; no gradual rollout needed).

## How I would use it (practical)
- **Context:** Launching new payment processor integration (risky; affects revenue).
- **Steps:**
  1. **Implement feature behind flag:**
     ```go
     func ProcessPayment(ctx context.Context, user User, amount int) error {
         if featureFlags.IsEnabled("stripe-payment-processor", user) {
             return stripeProcessor.Charge(ctx, amount)
         }
         return legacyPaymentProcessor.Charge(ctx, amount)
     }
     ```
  2. **Deploy code with flag OFF:**
     - Code ships to production.
     - All users still use legacy processor (no change).
  3. **Enable for internal users (1%):**
     - Test in production with real traffic.
     - Monitor errors, latency, success rate.
  4. **Gradual rollout:**
     - Enable for 5% of users (if metrics look good).
     - Increase to 25%, 50%, 100% over days/weeks.
  5. **Full rollout:**
     - Flag is ON for 100% of users.
     - Monitor for 1-2 weeks to ensure stability.
  6. **Cleanup:**
     - Remove flag logic; delete old code path.
     - Update flag system (mark flag as retired).
- **What success looks like:** Zero downtime; instant rollback capability; gradual validation; clean code after removal.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Decouples deployment from release (deploy anytime, release when ready).
  - Instant rollback (flip flag off; no redeploy).
  - Gradual rollout (validate with small percentage first).
  - A/B testing (measure impact before full rollout).
  - Kill switches (disable feature during incident).
- **Cons / Risks:**
  - Technical debt (old flags accumulate if not cleaned up).
  - Complexity (code has conditional paths; harder to test all combinations).
  - Performance overhead (flag evaluation on every request).
  - Configuration drift (flag states differ across environments).
  - Testing challenges (must test both paths; flag combinations explode).

### Alternatives
- **[Canary deployments](canary-deployments.md):** Similar gradual rollout, but at deployment level (not feature level).
- **[Blue/green deployments](blue-green-deployments.md):** Instant cutover, but entire deployment (not individual features).
- **[Rolling deployments](rolling-deployments.md):** Gradual instance replacement, but no feature-level control.
- **Branch by abstraction:** Refactor code to isolate feature; no flags (but longer integration time).

### How to choose
- **Feature-level control, instant rollback:** Feature flags.
- **Deployment-level validation:** Canary.
- **Instant infrastructure cutover:** Blue/green.
- **Simple incremental deployment:** Rolling.

## Failure modes & Pitfalls
- **Flag debt:** Flags never cleaned up; codebase full of if/else for old features (technical debt nightmare).
- **Flag combinatorial explosion:** 5 flags = 32 possible states; testing becomes impossible.
- **Stale flags:** Flag enabled in production but disabled in staging; environment drift causes bugs.
- **No default fallback:** Flag system unavailable; code crashes (should have safe default).
- **Over-reliance on flags:** Every feature behind flag; codebase becomes unmaintainable.
- **No monitoring:** Flag flipped without watching metrics; issues go undetected.

## Observability (How to detect issues)
- **Metrics:**
  - Flag evaluation count (how often flag is checked).
  - Flag state per environment (ensure consistency).
  - Feature adoption (% of users with flag enabled).
  - Error rate by flag state (compare enabled vs. disabled).
  - Latency by flag state (new feature slower?).
- **Logs:** Log flag state on request (debugging; know which code path executed).
- **Traces:** Tag spans with flag state (trace both paths separately).
- **Alerts:** Flag system unavailable, error rate spike after flag flip, unexpected flag state.

## Implementation notes
- **Checklist**
  - [ ] Define flag type (release, ops, experiment, permission).
  - [ ] Implement flag with safe default (if flag system fails, what happens?).
  - [ ] Add metrics for both code paths (compare enabled vs. disabled).
  - [ ] Test both flag states (enabled and disabled).
  - [ ] Document cleanup plan (when/how to remove flag).
  - [ ] Monitor during rollout (error rate, latency, business metrics).
  - [ ] Remove flag after full rollout (within 1-2 weeks of 100% enabled).

- **Security / Compliance notes**
  - Audit flag changes (who flipped flag, when, why).
  - Restrict flag access (not all engineers should toggle production flags).
  - Encrypt sensitive flags (API keys, feature access).

- **Performance notes**
  - Cache flag state (don't query flag system on every request).
  - Use local evaluation (download flag rules to service; evaluate locally).

- **Operational notes**
  - Document flag lifecycle (create → rollout → cleanup).
  - Schedule flag cleanup sprints (quarterly review of all flags).
  - Use flag naming convention (e.g., `release-YYYY-MM-feature-name`).

## Mini example (Go with simple flag system)
```go
package main

import (
    "context"
    "fmt"
)

// Simple in-memory flag system (use LaunchDarkly/Split.io in production)
type FeatureFlags struct {
    flags map[string]bool
}

func NewFeatureFlags() *FeatureFlags {
    return &FeatureFlags{
        flags: map[string]bool{
            "new-checkout-flow": false, // Default: disabled
        },
    }
}

func (ff *FeatureFlags) IsEnabled(flagName string, user User) bool {
    // In production: check user targeting rules, percentage rollout, etc.
    enabled, exists := ff.flags[flagName]
    if !exists {
        return false // Safe default: disabled
    }
    return enabled
}

type User struct {
    ID string
}

// Feature implementation
func HandleCheckout(ctx context.Context, ff *FeatureFlags, user User) {
    if ff.IsEnabled("new-checkout-flow", user) {
        fmt.Println("Using new checkout flow (Stripe)")
        // New implementation
        return
    }
    fmt.Println("Using legacy checkout flow (PayPal)")
    // Old implementation
}

func main() {
    ff := NewFeatureFlags()
    user := User{ID: "user-123"}

    // Before rollout: flag disabled
    HandleCheckout(context.Background(), ff, user)
    // Output: Using legacy checkout flow (PayPal)

    // After gradual rollout: flag enabled
    ff.flags["new-checkout-flow"] = true
    HandleCheckout(context.Background(), ff, user)
    // Output: Using new checkout flow (Stripe)

    // After cleanup: remove flag logic entirely (just call new flow)
}
```

## Common anti-patterns
- **Anti-pattern:** Never cleaning up flags (permanent if/else in code).
  - **Why it's bad:** Technical debt accumulates; codebase becomes spaghetti.
  - **Better approach:** Schedule flag cleanup 1-2 weeks after 100% rollout; remove flag and old code.

- **Anti-pattern:** Using flags for static configuration (e.g., database URL).
  - **Why it's bad:** Flags are for dynamic behavior; static config belongs in environment variables.
  - **Better approach:** Use config files or env vars for static values; use flags only for runtime decisions.

- **Anti-pattern:** No safe default (flag system fails → code crashes).
  - **Why it's bad:** Flag system becomes single point of failure.
  - **Better approach:** Always have safe default (e.g., return false if flag system unavailable).

- **Anti-pattern:** Testing only one flag state (e.g., only test enabled).
  - **Why it's bad:** Disabled path has bugs; surprises when rollback is needed.
  - **Better approach:** Test both states in CI; ensure both paths work.

## Interview readiness
### "Explain it like I'm teaching"
Feature flags let you deploy code to production but keep features turned off until you're ready to release them. For example, you add a new payment processor to your code, but wrap it in an if statement: `if (flag.isEnabled("new-payment")) { use new processor }`. The code is deployed, but the flag is off, so users still see the old payment flow. When ready, you flip the flag to on for 1% of users, monitor metrics, and gradually increase to 100%. If issues arise, you flip the flag back off instantly—no redeployment needed. The main benefit is decoupling deployment from release and enabling instant rollback. The downside is technical debt: you must clean up flags after rollout or your codebase becomes full of old if/else statements.

### Trap questions (with answers)
1) **Q:** What's the difference between a feature flag and a configuration variable?
   - **A:** 
     - **Feature flag:** Dynamic; changes at runtime; controls feature behavior; can target specific users.
     - **Configuration variable:** Static; set at deployment time; doesn't change without redeploy; global setting.
     - Example: Flag = "show new UI to 10% of users"; Config = "database connection string".

2) **Q:** What are the different types of feature flags?
   - **A:** 
     - **Release flags:** Enable incomplete features; short-lived (cleanup after rollout).
     - **Ops flags:** Kill switches for expensive features (e.g., disable recommendations during incident).
     - **Experiment flags:** A/B tests; measure impact of variants.
     - **Permission flags:** Access control (e.g., premium features for paid users).

3) **Q:** Why is flag cleanup important?
   - **A:** Old flags create technical debt. Each flag adds conditional logic (if/else), making code harder to read, test, and maintain. With 10 flags, you have 1,024 possible states—impossible to test. Cleanup keeps code simple and removes unused paths.

4) **Q:** How do feature flags support trunk-based development?
   - **A:** In trunk-based dev, all engineers merge to main daily (no long-lived branches). Incomplete features would break production. Flags let you merge incomplete code (behind disabled flag), avoiding merge conflicts while keeping main deployable. When feature is done, flip flag on and remove it.

5) **Q:** What's the performance impact of feature flags?
   - **A:** Flag evaluation happens on every request. Without caching, this adds latency (e.g., network call to flag service). Mitigate by: (1) caching flag state locally, (2) downloading flag rules and evaluating in-process, (3) using CDN for flag distribution. Well-designed systems add <1ms overhead.

### Quick self-check (5 items)
- [ ] I can explain how feature flags decouple deployment from release.
- [ ] I can describe the 4 types of flags (release, ops, experiment, permission).
- [ ] I can articulate why flag cleanup is critical (technical debt).
- [ ] I can compare feature flags to canary/blue-green deployments.
- [ ] I can explain how to implement a safe default for flag failures.

## Links (NO duplication)
### Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)

### Related topics
- [Canary deployments](canary-deployments.md)
- [Blue/green deployments](blue-green-deployments.md)
- [Rolling deployments](rolling-deployments.md)

### Compare with
- [Canary deployments](canary-deployments.md) — Canary is deployment-level gradual rollout; flags are feature-level.

## Notes / Inbox
- Add examples from real projects: feature flags for Heimdall experiments, Opportunity Actions feature gating.
- Consider adding section on flag management tools (LaunchDarkly, Split.io, Unleash).
- Link to A/B testing details when created.
