---
id: conducting-effective-code-reviews
title: "Conducting Effective Code Reviews"
type: playbook
category: technical
difficulty: medium
importance: 90
tags: [code-review, quality, collaboration, feedback, mentorship]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Conducting Effective Code Reviews

## Situation
**When this applies:**
- Reviewing pull requests from team members
- Acting as tech lead or senior engineer
- Mentoring junior developers
- Maintaining code quality standards

**Common manifestations:**
- "Can you review my PR?"
- Large PR (500+ lines) that needs feedback
- Junior dev's first feature implementation
- Critical bug fix needs quick approval
- Architecture change proposal

## Why this matters
- Code reviews catch bugs before production (cheaper than debugging later)
- Transfer knowledge across team (context sharing)
- Ensure maintainability (you might debug this at 3am)
- Build team culture (how you review affects morale)
- Critical interview topic: demonstrates technical depth, communication, mentorship, balancing speed vs quality

## My Process (Step-by-step)

### 1. Understand context before reading code
- **What:** Read PR description, linked ticket, and requirements
- **Why:** Code makes sense only with context; avoid wasting time guessing intent
- **How:** Check: what problem does this solve? Why this approach? Any constraints?
- **Example:**
  - **PR Title:** "Add caching to user profile API"
  - **Read:** Ticket says "p95 latency = 800ms, SLA = 200ms; cache for 5min"
  - **Context:** Performance problem, cache-aside pattern expected, TTL = 5min

### 2. Check for correctness first
- **What:** Does the code do what it claims?
- **Why:** Correct > clean; bugs are highest priority
- **How:** Trace execution path, check edge cases, verify error handling
- **Example:**
  ```python
  # Code:
  def get_user(user_id):
      cached = redis.get(f"user:{user_id}")
      if cached:
          return json.loads(cached)
      user = db.get_user(user_id)
      redis.set(f"user:{user_id}", json.dumps(user), ex=300)
      return user
  
  # Check:
  - ✅ Cache hit returns cached data
  - ❌ What if db.get_user() returns None? (bug: caches None)
  - ❌ What if json.loads() fails? (bug: no exception handling)
  - ❌ What if redis is down? (bug: crashes instead of fallback)
  ```

### 3. Evaluate performance and scalability
- **What:** Will this handle production load?
- **Why:** Performance issues hard to fix after launch
- **How:** Look for: N+1 queries, unbounded loops, missing indexes, blocking calls, memory leaks
- **Example:**
  ```python
  # Code:
  def get_users_with_orders(user_ids):
      users = []
      for user_id in user_ids:
          user = db.get_user(user_id)  # ❌ N+1 query
          user['orders'] = db.get_orders(user_id)  # ❌ N+1 query
          users.append(user)
      return users
  
  # Comment:
  "This will make 2N database queries for N users. For 100 users = 200 queries.
  Suggest: batch load users in single query, then batch load orders by user_ids."
  ```

### 4. Check for maintainability and clarity
- **What:** Will this be easy to understand/modify in 6 months?
- **Why:** Code is read 10x more than written
- **How:** Assess: naming, function size, comments, abstraction, duplication
- **Example:**
  ```python
  # Code:
  def f(u, t):  # ❌ Unclear names
      if t == 1:  # ❌ Magic number
          return u * 0.9
      return u
  
  # Comment:
  "Suggest renaming: f → calculate_price, u → base_amount, t → user_type.
  Replace magic numbers with constants: PREMIUM_USER = 1, DISCOUNT_RATE = 0.1"
  ```

### 5. Verify security and error handling
- **What:** Are there security holes or crash scenarios?
- **Why:** Security bugs costly; unhandled errors cause outages
- **How:** Check: input validation, SQL injection, auth checks, exception handling, logging
- **Example:**
  ```python
  # Code:
  def delete_user(user_id):
      query = f"DELETE FROM users WHERE id = {user_id}"  # ❌ SQL injection
      db.execute(query)
  
  # Comment:
  "SQL injection vulnerability: user_id not validated. Use parameterized query:
  db.execute('DELETE FROM users WHERE id = %s', (user_id,))"
  ```

### 6. Classify feedback: blocking vs nice-to-have
- **What:** Separate must-fix from suggestions
- **Why:** Unblocks author while maintaining quality bar
- **How:** Use prefixes: **[blocking]** for bugs/security, **[nit]** for style, **[suggestion]** for improvements
- **Example:**
  - **[blocking]** "SQL injection: use parameterized query"
  - **[suggestion]** "Consider extracting this logic to helper function for reusability"
  - **[nit]** "Add blank line here for readability"

### 7. Give constructive feedback with reasons
- **What:** Explain why, not just what to change
- **Why:** Teaches thinking, not just fixes
- **How:** Format: "Issue → Why it matters → How to fix → Example"
- **Example:**
  - ❌ **Bad:** "This is wrong"
  - ✅ **Good:** "This will cause N+1 queries (1 query per user). At 1000 users, that's 1000 DB calls = slow. Suggest batch loading: `db.get_users(user_ids)`"

### 8. Approve or request changes with next steps
- **What:** Clear action for author
- **Why:** Avoids back-and-forth confusion
- **How:** State: "Approved" (ready to merge) OR "Request changes: fix [blocking items], then I'll re-review"
- **Example:**
  - **Approve:** "LGTM! Cache logic correct, edge cases handled. Merge when CI passes."
  - **Request changes:** "Two blocking issues: (1) SQL injection on line 45, (2) missing error handling for Redis failure. Please fix and ping me for re-review."

## Success Signals
**You know it's working when:**
- [ ] Bugs caught in review, not production
- [ ] Authors understand feedback (not just fix mechanically)
- [ ] Team members request your reviews
- [ ] Review turnaround <24h (doesn't block velocity)
- [ ] Authors improve over time (fewer blocking comments)

## Common Pitfalls (and how to avoid them)

### Pitfall 1: Nitpicking style instead of substance
- **Why it happens:** Easy to comment on formatting, hard to find logic bugs
- **Impact:** Wastes time, discourages author, misses real issues
- **Instead do:** Use linter for style; focus on correctness, performance, security

### Pitfall 2: Not explaining why
- **Why it happens:** Assumes author has same context
- **Impact:** Author doesn't learn, repeats mistake
- **Instead do:** Add one sentence explaining the reason

### Pitfall 3: Blocking on personal preference
- **Why it happens:** "I would have done it differently"
- **Impact:** Frustrates author, slows delivery
- **Instead do:** Use **[nit]** or **[suggestion]** for non-critical feedback; approve if functionally correct

### Pitfall 4: Reviewing too slowly
- **Why it happens:** Other priorities, waiting for "perfect time"
- **Impact:** Blocks author, slows team velocity
- **Instead do:** Set SLA (e.g., first pass within 4h); batch reviews 2-3x/day

## Real-world Example
**Context:** Junior dev submitted 400-line PR adding payment processing; needed thorough review.

**What happened:** PR added Stripe integration for subscription payments.

**Process applied:**
1. **Context:** Read ticket: "Users can subscribe to premium ($9.99/month). Use Stripe. Store subscription in DB."
2. **Correctness:**
   - **Bug found:** After successful Stripe charge, code didn't check if DB insert failed (user charged but subscription not saved)
   - **Comment:** **[blocking]** "If DB insert fails (line 67), user is charged but subscription not saved. Wrap in transaction OR use Stripe webhooks to reconcile."
3. **Performance:**
   - **Issue:** Stripe API call in HTTP request (blocking, timeout risk)
   - **Comment:** **[suggestion]** "Consider async: return 202 Accepted, process Stripe in background job. Avoids timeout if Stripe is slow."
4. **Maintainability:**
   - **Good:** Clear function names (`create_subscription`, `charge_customer`)
   - **Nit:** Magic number $9.99 hardcoded in 3 places
   - **Comment:** **[nit]** "Extract PREMIUM_PRICE = 9.99 constant"
5. **Security:**
   - **Critical:** Stripe secret key in code (not environment variable)
   - **Comment:** **[blocking]** "Stripe secret key hardcoded (line 12). Move to environment variable or secrets manager. Never commit secrets."
6. **Classified:**
   - **Blocking:** (1) DB insert failure handling, (2) Secret key in code
   - **Suggestion:** (1) Async processing
   - **Nit:** (1) Extract constant
7. **Feedback:**
   - **[blocking]** "Two critical issues: (1) If DB insert fails after Stripe charge, user loses money. Use transaction or webhook reconciliation. (2) Stripe key hardcoded—move to env var."
   - **[suggestion]** "Stripe call is synchronous (blocking HTTP request). If Stripe is slow, request times out. Consider: return 202, process in background, notify user via email when done."
   - **[praise]** "Great test coverage (85%)! Clear function names."
8. **Action:**
   - **Requested changes:** "Fix blocking issues (DB transaction + secret key), then ping me. Async suggestion can be follow-up PR if you prefer."

**Outcome:**
- Author fixed blocking issues in 2 hours
- Re-reviewed and approved
- Follow-up PR added async processing
- Author learned transaction patterns and secret management

## Interview Tips
### How to talk about this
- **Framework to use:** "Context → correctness → performance → maintainability → security → classify → constructive feedback → action"
- **Key points to emphasize:** Balanced speed and quality, teaching mindset, clear priorities (blocking vs nice-to-have)
- **Language that works:** "verified correctness," "checked performance," "classified feedback," "explained why," "unblocked author"

### Sample STAR response outline
- **Situation:** "Reviewing [team member's] PR for [feature], [LOC size], [complexity context]"
- **Task:** "Ensure correctness, performance, security while maintaining velocity and teaching"
- **Action:** "Read context, traced execution for correctness, checked [specific areas: perf/security], classified feedback [blocking/suggestion], explained reasoning, provided [action: approved/requested changes]"
- **Result:** "Caught [X bugs/issues] before production, author [learned Y], merged within [time], feature worked in production without issues"

### Red flags to avoid
- Don't say "I always check every line" (shows lack of prioritization)
- Don't claim "I approve everything" or "I block everything" (shows poor judgment)
- Don't omit teaching aspect (senior engineers teach through reviews)
- Don't forget turnaround time (slow reviews block team)

## Self-assessment
- [ ] I can describe my code review process (what I check and in what order)
- [ ] I can give examples of bugs I caught in review
- [ ] I can explain how I balance speed (unblocking author) vs quality (thorough review)
- [ ] I can show how I give constructive feedback (not just "this is wrong")
- [ ] I can describe how I prioritize (blocking vs nice-to-have)

## Related Playbooks
- [Applying Deep Technical Knowledge](applying-deep-knowledge.md)
- [Mentorship and Knowledge Transfer](../leadership/mentorship-knowledge-transfer.md)
- [Continuous Learning and Growth](../behavioral/continuous-learning-growth.md)
- [Balancing Quality and Technical Debt](quality-technical-debt.md)

## Related Topics (knowledge base)
- [Testing Pyramid](../../topics/quality/testing-pyramid.md)
- [Code Review Best Practices](../../topics/quality/code-review-best-practices.md)
- [Security Best Practices](../../topics/quality/security-best-practices.md)
- [SQL Injection](../../topics/quality/sql-injection.md)
- [Error Handling Patterns](../../topics/quality/error-handling-patterns.md)
- [Performance Basics](../../topics/operations/performance-basics.md)

## Notes / Refinements
- Default to approving unless blocking issue (bias toward unblocking)
- Use **[blocking]**, **[suggestion]**, **[nit]**, **[question]**, **[praise]** prefixes
- Not every comment needs response (nits can be ignored if author disagrees)
- Large PRs (>500 LOC): ask author to split OR schedule pairing session
- First review pass: focus on architecture/correctness; second pass: details
