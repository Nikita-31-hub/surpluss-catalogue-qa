# Findings

---

## 1. Expired catalogues are still treated as live

**What happens**

The expiry check in `effectiveStatus` is inverted. A catalogue is marked as expired only when `expiresAt` is in the future, so any catalogue whose validity date has passed remains displayed as published.

**Steps to reproduce**

1. Create or edit a catalogue and set `expiresAt` to a time in the past.
2. Open the admin catalogue list or the public catalogue page.
3. Observe that the catalogue still appears as active instead of expired.

**What should happen instead**

Once the validity date has passed, the catalogue should be shown as expired and not presented as a live offer.

**Impact — how bad is this, and why?**

This is mostly a pricing and trust issue, but it can affect buying decisions: buyers can still see stale offers after the validity window closes, and internal sales teams may make decisions based on the wrong live state.

**Failing test**

`src/lib/catalogue-status.test.ts` — `marks a published catalogue as expired once its validity date has passed`

---

## 2. Any signed-in user can delete a catalogue

**What happens**

The server action `deleteCatalogue` validates that the user is signed in, but never checks whether the actor is an admin. That means a staff account can delete a catalogue and the related lead history from the database.

**Steps to reproduce**

1. Sign in as a non-admin user with the `staff` role.
2. Call `deleteCatalogue` with a valid catalogue UUID.
3. The action deletes the record instead of rejecting the request.

**What should happen instead**

The server should reject the request unless the actor is an admin, and return an explicit authorization error instead of deleting anything.

**Impact — how bad is this, and why?**

This is a serious access-control bug. It can destroy catalogues, remove lead history, and cause revenue loss or operational disruption if a staff user deletes a live catalogue by mistake or maliciously.

**Failing test**

`src/app/admin/actions.security.test.ts` — `blocks non-admin users from deleting a catalogue`
