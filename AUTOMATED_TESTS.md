# Automated Tests Created

This document lists the automated tests currently added to the project and what each one verifies.

## 1. Catalogue expiry status test

File: `src/lib/catalogue-status.test.ts`

### Test: `marks a published catalogue as expired once its validity date has passed`

What it checks:
- A catalogue with status `published` and an `expiresAt` value in the past should resolve to the effective status `expired`.

Why it matters:
- Prevents stale catalogues from being treated as still live after their validity period has ended.
- Protects buyers from seeing outdated offers and reduces the risk of incorrect sales decisions.

### Test: `keeps a published catalogue active while its validity window is still open`

What it checks:
- A catalogue with status `published` and a future `expiresAt` value should remain `published`.

Why it matters:
- Ensures valid live catalogues are not incorrectly marked expired before their sales window ends.

---

## 2. Admin delete authorization test

File: `src/app/admin/actions.security.test.ts`

### Test: `blocks non-admin users from deleting a catalogue`

What it checks:
- A signed-in user with the `staff` role cannot delete a catalogue.
- The action returns an authorization error and does not call the Prisma delete operation.

Why it matters:
- Prevents staff users from deleting catalogue data and lead history without admin permission.
- Enforces server-side access control, which is crucial because UI hiding alone is not a security boundary.

---

## Summary of coverage

The project’s automated tests currently cover the most sensitive and risky business rules:

- catalogue lifecycle status transitions
- access control for destructive admin actions

These are the parts most likely to cause customer-facing issues or data loss if logic regresses.
