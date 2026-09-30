# Write-up

---

## 1. Strategy

I started with the highest-risk business logic and the most dangerous access-control path, because those are the places a small bug can create real customer or revenue damage.

I focused on:

- catalogue lifecycle logic, especially expired vs. live states
- admin authorization checks, because deleting a catalogue is destructive
- regression tests around those exact failure modes so the bug cannot silently return

I deliberately did not try to cover every UI or import flow in depth. The goal was to protect the parts most likely to cause incorrect pricing, stale public offers, or data loss.

## 2. The riskiest part of this product

The riskiest area is the public catalogue flow: pricing, validity windows, and access to admin actions all interact. If a published catalogue stays live after it should expire, buyers can still see stale offers; if the delete path is not guarded, a staff user can remove catalogue data and lead history. That is the most direct path to business disruption.

## 3. What I left out, and why

I did not spend time building a broad Playwright matrix or exhaustive spreadsheet-import suite because the project did not yet have a stable shared test harness for those flows, and the assessment is best served by sharp, high-signal checks rather than a large but shallow suite.

I also did not try to cover every edge-case in the importer or every route in the admin app. The value is in proving the real breakpoints: expiry logic and authorization, which are the places where correctness matters most.

## 4. AI tool usage

I used AI assistance mostly to scan the codebase, identify likely hotspots, and draft the regression tests. I then reviewed each generated assertion and fixed the exact source logic instead of trusting the first pass.

In practice, I used the tools to accelerate discovery, then I tightened the failing checks around the actual root cause: the inverted expiry comparison and the missing admin guard in `deleteCatalogue`.

## 5. One thing this codebase gets wrong

The codebase has a strong product surface but a weak test safety net. The biggest quality problem is that business-critical logic is spread across small helper functions and server actions without a clear regression layer around expiry and access control.

I would fix that by enforcing a small set of explicit business-rule tests for every catalogue status transition and every admin-only action, so accidental logic inversion and role bypasses are caught immediately before they hit production.

---

## Notes

The app initially had no matching Vitest tests to run, so I created a focused regression suite around the real bugs I found and then corrected the server behavior to match the intended rules.
