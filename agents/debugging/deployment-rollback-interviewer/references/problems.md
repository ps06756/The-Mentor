# Problem Bank: Deployment Rollback Debugging

## Problem 1: Database Migration Incompatible with Rollback

### Problem Statement
Version 2.4.0 includes a database migration that adds a `NOT NULL` column `order_metadata` to the orders table. After deploying, error rates spike to 15%. The team wants to rollback to v2.3.0, but the database migration has already run. Determine if rollback is safe and execute the correct recovery strategy.

### Solution Walkthrough

**The Migration:**
```sql
-- v2.4.0 migration (already executed)
ALTER TABLE orders ADD COLUMN order_metadata JSONB NOT NULL;
```

**Why Rollback is Dangerous:**
```
v2.3.0 INSERT statement:
  INSERT INTO orders (customer_id, amount, status) VALUES (123, 99.99, 'pending');

After migration, this fails:
  ERROR: null value in column "order_metadata" violates not-null constraint

Result: Rolling back code from v2.4.0 to v2.3.0 breaks ALL order creation.
Error rate goes from 15% to 100%.
```

**Recovery Options:**

**Option A: Fix Forward (Preferred)**
```
1. Identify the bug in v2.4.0 (separate from the migration)
2. Deploy hotfix v2.4.1 that fixes the bug
3. Migration stays in place, new code works with it
Time: 30-60 minutes (depends on fix complexity)
Risk: Low if the fix is simple
```

**Option B: Make Migration Backward-Compatible, Then Rollback**
```sql
-- Make the column nullable with a default
ALTER TABLE orders ALTER COLUMN order_metadata SET DEFAULT '{}';
ALTER TABLE orders ALTER COLUMN order_metadata DROP NOT NULL;

-- Now v2.3.0 can insert without setting order_metadata
-- Rollback code to v2.3.0
```
```
Time: 10-15 minutes
Risk: Medium (altering production table under load)
```

**Option C: Roll Back Migration Too (Dangerous)**
```sql
ALTER TABLE orders DROP COLUMN order_metadata;
```
```
Time: 5 minutes
Risk: HIGH - Any data written to order_metadata since deploy is lost
Only use if: No important data was written to the new column
```

**The Right Answer: Prevention**
The migration should never have been deployed this way. The expand-contract pattern:
```
v2.3.5: ADD COLUMN order_metadata JSONB DEFAULT '{}' (nullable, with default)
v2.4.0: Code starts writing to order_metadata
v2.5.0: Backfill all NULL values, then ALTER COLUMN SET NOT NULL
```

### Follow-up Questions
- "What if the migration took 30 minutes to run? Can we afford to roll it back?"
- "What if the new column has a foreign key constraint?"
- "How do you test that a migration is backward-compatible?"

---

## Problem 2: Feature Flag with Incomplete Backend Gating

### Problem Statement
A new discount code feature is behind a feature flag enabled for 10% of users. After deploying, error rates spike to 15%. The feature flag is only 10%, so why is the error rate 15%? Diagnose and fix.

### Solution Walkthrough

**Investigation:**
```
Feature flag: enabled for 10% of users (frontend only)
Error rate: 15% (higher than the flag percentage)
Affected endpoint: POST /api/checkout
```

**The Bug:**
```java
// Frontend (correctly gated):
if (featureFlags.isEnabled("discount_codes", userId)) {
    showDiscountCodeInput();  // Only 10% see this
}

// Backend (NOT gated):
public CheckoutResponse checkout(CheckoutRequest request) {
    // This runs for ALL requests, not just flagged users
    DiscountCode code = discountService.validate(request.getDiscountCode());
    // request.getDiscountCode() is null for 90% of users
    // discountService.validate(null) throws NullPointerException

    // But wait - the error rate is 15%, not 90%...
    // Because the validation only runs when the checkout total > $50
    // and ~17% of checkouts are > $50 (15% error rate ≈ non-flagged users with $50+ carts)
}
```

**Diagnosis Steps:**
1. Check error rate by feature flag status:
   - Flagged users (10%): 0% error rate (they set discount codes correctly)
   - Non-flagged users (90%): ~17% error rate (those with carts > $50)
   - Blended: 0.9 * 17% = 15.3% -- matches observed error rate
2. Check error logs: `NullPointerException at discountService.validate()`
3. Check code: backend validation runs unconditionally

**Fix (Immediate):**
```java
public CheckoutResponse checkout(CheckoutRequest request) {
    if (featureFlags.isEnabled("discount_codes", request.getUserId())) {
        DiscountCode code = discountService.validate(request.getDiscountCode());
        applyDiscount(code);
    }
    // ... rest of checkout
}
```

**Fix (Alternative - Even Faster):**
```java
// Just null-check the discount code
if (request.getDiscountCode() != null) {
    DiscountCode code = discountService.validate(request.getDiscountCode());
    applyDiscount(code);
}
```

**Prevention:**
- Feature flag policy: flags must gate BOTH frontend AND backend code paths
- PR review checklist: "If this feature is behind a flag, is the flag checked in all relevant code paths?"
- Integration tests that simulate flagged and non-flagged users hitting the same endpoint
- Shadow mode: run new code in shadow for non-flagged users (log but don't act), verify no errors

### Follow-up Questions
- "How would you write a test that catches this?"
- "What's the difference between a release flag and an experiment flag?"
- "Should you fix forward or rollback in this case?"

---

## Problem 3: Breaking Change in Dependency Upgrade

### Problem Statement
PR #901 upgraded `payment-sdk` from v3.1 to v4.0. Error logs show `NoSuchMethodError: PaymentClient.charge(Amount)`. The method signature changed in v4.0. Find all broken call sites and fix them.

### Solution Walkthrough

**The Breaking Change:**
```
payment-sdk v3.1:
  PaymentClient.charge(Amount amount) -> ChargeResult

payment-sdk v4.0:
  PaymentClient.charge(Amount amount, ChargeOptions options) -> ChargeResult
  // charge(Amount) was removed -- it's a major version bump
```

**Call Site Audit:**
```
PR #901 updated:
  [x] CheckoutController.processPayment()     -> Updated to new API

PR #901 missed:
  [ ] OrderRetryJob.retryFailedPayments()      -> Still uses old API
  [ ] RefundProcessor.issueRefund()            -> Still uses old API
  [ ] SubscriptionRenewalJob.renewPayment()    -> Still uses old API
```

**Why Tests Didn't Catch It:**
- `CheckoutController` has unit tests -- they pass with the new API
- `OrderRetryJob` is a background cron job -- no integration tests run it during deploy
- `RefundProcessor` only runs when a refund is requested -- not covered in smoke tests
- `SubscriptionRenewalJob` runs at midnight -- error won't surface until then

**Diagnosis Steps:**
1. Error logs: `NoSuchMethodError: PaymentClient.charge(Amount)` in `OrderRetryJob`
2. Grep codebase for old API usage:
   ```
   grep -r "charge(amount)" --include="*.java" src/
   ```
3. Find 3 missed call sites
4. Check test coverage: none of the background jobs have integration tests

**Fix:**
```java
// Update all call sites to new API
// OrderRetryJob.java
ChargeResult result = paymentClient.charge(amount, ChargeOptions.withRetry());

// RefundProcessor.java
ChargeResult result = paymentClient.charge(refundAmount, ChargeOptions.forRefund());

// SubscriptionRenewalJob.java
ChargeResult result = paymentClient.charge(renewalAmount, ChargeOptions.forSubscription());
```

**Prevention:**
- Major dependency upgrades must include a full-codebase audit of changed APIs
- Add to PR template: "For dependency upgrades: have all usages of changed APIs been updated?"
- Use compiler warnings / deprecation notices: pin to v3.1 first, fix all deprecation warnings, then upgrade to v4.0
- Integration test suite must exercise ALL code paths, including background jobs
- Canary deploys: roll out to 5% of traffic first, catch errors before full rollout
- Consider using an API compatibility check tool (e.g., `japicmp` for Java)

### Follow-up Questions
- "Should major dependency upgrades be their own release, separate from feature work?"
- "What if the old API is deprecated but not removed? How would you handle migration gradually?"
- "How do you test background jobs in a CI pipeline?"

---

## Sample Session Flow

### Opening
**Interviewer**: "We deployed v2.4.0 at 2pm. Fifteen minutes later, error rates went from 0.1% to 15% and p99 latency doubled. I'm the release engineer and I need to make a call: rollback, fix forward, or something else. Help me think through this."

### Phase 1: Triage (5 minutes)
**Interviewer**: "Here's the error rate graph and the v2.4.0 changelog. Three PRs shipped. What's your first move?"

*[Candidate examines changelog]*

### Phase 2: Rollback Decision (15 minutes)
*[Candidate suggests rollback]*
**Interviewer**: "Hold on. PR #905 included a database migration. Can we rollback safely?"

*[Walk through the migration compatibility analysis]*

### Phase 3: Root Cause (15 minutes)
**Interviewer**: "Good, we decided to fix forward. Now we need to find which PR caused the errors. How do you narrow it down?"

*[Candidate investigates each PR]*

### Phase 4: Process Improvement (10 minutes)
**Interviewer**: "Incident resolved. Three things shipped in this release and any one of them could have been caught earlier. What changes to our process?"

*[Generate scorecard]*
