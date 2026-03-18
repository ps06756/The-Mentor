# Example: Debugging Interview (Cascading Failure)

A realistic transcript showing the AI interviewer presenting symptoms and testing diagnostic skills.

---

**Interviewer:** We have a P0 incident. The Payment Service response times went from 100ms to 30 seconds. Now the Order Service is down. The Cart Service is down. The web frontend is returning blank pages. 100% of users are affected. You're the incident commander. What do you do first?

**You:** I'd check the Payment Service logs to see what's wrong.

**Interviewer:** You're jumping to root cause before triage. Right now, everything is on fire. What's your first priority — understanding *why* it happened, or stopping the bleeding?

**You:** Oh, right. Stopping the bleeding first. I'd... restart the Payment Service?

**Interviewer:** That might work temporarily, but you said Order Service, Cart Service, and the frontend are all down too. Restarting Payment won't fix those. Why are they failing? They're separate services.

**You:** Because they depend on Payment?

**Interviewer:** Do they *all* directly call the Payment Service? Does the Cart Service need to charge a credit card? Think about the failure pattern — Payment slows down, then everything else cascades. What mechanism causes that?

**You:** ... thread pool exhaustion? The Order Service is waiting on Payment, its threads fill up, then it can't respond to the Gateway, and the Gateway fills up too?

**Interviewer:** Exactly. Now you've identified the mechanism. What should have been in place to prevent this?

**You:** A timeout on the Payment call?

**Interviewer:** That helps, but 30 seconds * 1000 concurrent requests still fills the thread pool before timeouts kick in. What pattern catches a failing dependency *before* every request tries it?

**You:** A circuit breaker.

**Interviewer:** Yes. Now walk me through how you'd implement it. What triggers the circuit to open? When does it close again? What does the user see while it's open?

---

**Scorecard:**

| Area | Rating | Notes |
|------|--------|-------|
| Incident Response | Novice | Jumped to debugging before triage |
| System Thinking | Intermediate | Identified cascading mechanism with prompting |
| Mitigation | Novice | Proposed restart without understanding propagation |
| Prevention | Intermediate | Named circuit breaker but needed guidance on implementation |

**Work on:** Practice the triage-first mindset: mitigate before debugging. Study the circuit breaker pattern (Closed → Open → Half-Open state machine). Read about bulkhead isolation and backpressure.
