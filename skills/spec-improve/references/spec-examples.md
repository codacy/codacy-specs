# Spec Transformation Examples

Before/after examples for each spec type. Use these during Phase 5 to calibrate rewriting quality — this is what a good rewrite looks like.

## Contents

- [Example 1: API Endpoint](#example-1-api-endpoint)
- [Example 2: UI Component](#example-2-ui-component)
- [Example 3: Bug Fix](#example-3-bug-fix)
- [Example 4: Backend Feature](#example-4-backend-feature)
- [Calibration Notes](#calibration-notes)

---

## Example 1: API Endpoint

### Before (D-grade, score: 38/100)

```
Title: Add endpoint to get user orders

We need an API endpoint that returns the orders for a user. Should support filtering by status.
```

**Why it fails:**
- No HTTP method or path
- No authentication requirements
- No request/response schema
- No error handling
- "Should support filtering" — vague, no filter values defined
- Zero acceptance criteria
- No edge cases

---

### After (A-grade, score: 91/100)

```markdown
## Overview

**What**: Create a `GET /users/{userId}/orders` endpoint that returns paginated orders for a given user, with optional filtering by order status.
**Why**: The mobile client needs to display a user's order history. Currently requires two API calls and client-side filtering, causing performance issues on large accounts.
**Who**: Authenticated mobile and web clients.

## Scope

**In scope:** Listing orders for a single user; filtering by status; pagination.
**Out of scope:** Creating, updating, or deleting orders; cross-user order queries; bulk exports.

---

## Endpoint Definition

| Field | Value |
|-------|-------|
| Method | `GET` |
| Path | `/users/{userId}/orders` |
| Auth | Bearer token (JWT) |
| Required roles | `user` (own orders only), `admin` (any userId) |
| Rate limit | 60 requests/minute per user |

---

## Parameters

### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `userId` | `string (UUID)` | Yes | The user whose orders to retrieve |

### Query Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `status` | `enum` | No | (all) | Filter by status. Values: `pending`, `processing`, `shipped`, `delivered`, `cancelled` |
| `page` | `integer` | No | `1` | Page number (1-indexed) |
| `pageSize` | `integer` | No | `20` | Results per page. Max: `100` |

---

## Response

### Success — `200 OK`

```json
{
  "data": [
    {
      "id": "ord_abc123",
      "status": "shipped",
      "total": 4999,
      "currency": "USD",
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-16T08:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 47,
    "totalPages": 3
  }
}
```

### Errors

| Status | Code | When |
|--------|------|------|
| `401` | `UNAUTHORIZED` | Missing or expired token |
| `403` | `FORBIDDEN` | Non-admin user requesting another user's orders |
| `404` | `USER_NOT_FOUND` | `userId` does not exist |
| `422` | `INVALID_PARAMETER` | `status` is not a valid enum value, or `pageSize` > 100 |

---

## Acceptance Criteria

- [ ] Given a valid token for user A, when `GET /users/{userA.id}/orders`, then return 200 with only user A's orders
- [ ] Given a valid admin token, when `GET /users/{userA.id}/orders`, then return 200 with user A's orders
- [ ] Given a valid token for user A, when `GET /users/{userB.id}/orders`, then return 403
- [ ] Given `status=shipped`, then only orders with status `shipped` are returned
- [ ] Given `pageSize=5` and the user has 12 orders, then return 5 orders and `totalPages: 3`
- [ ] Given `status=invalid_value`, then return 422 with `INVALID_PARAMETER`
- [ ] Given a userId that does not exist, then return 404 with `USER_NOT_FOUND`

## Edge Cases

- **User with no orders**: Return 200 with `{"data": [], "pagination": {"total": 0, "totalPages": 0}}`
- **Deleted user**: Return 404 (same as non-existent)
- **pageSize=0**: Return 422 with `INVALID_PARAMETER`
- **Requesting page beyond totalPages**: Return 200 with empty `data` array
```

**What changed:** Added HTTP method + path, auth + roles, full query parameter definitions, typed response schema with pagination, complete error table with status codes, 7 testable ACs, and 4 specific edge cases.

---

## Example 2: UI Component

### Before (D-grade, score: 41/100)

```
Title: Date picker component

Create a reusable date picker that can be used across the application. Should support date ranges and be accessible.
```

**Why it fails:**
- No props defined
- "Should support date ranges" — no specification of how range selection works
- "Be accessible" — no specific requirements
- No states, no interactions, no variants
- No acceptance criteria

---

### After (A-grade, score: 88/100)

```markdown
## Overview

**What**: A reusable `DatePicker` component for selecting a single date or a date range from a calendar popover.
**Why**: We currently use three different date input implementations across the product, causing UX inconsistency and accessibility failures (2 open a11y bug reports).
**Where used**: Booking flow, report filters, task due dates.

## Scope

**In scope:** Single date selection; date range selection; minimum/maximum date constraints; keyboard navigation.
**Out of scope:** Time selection (separate TimeInput component); recurring date patterns; multi-calendar view.

---

## Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `mode` | `'single' \| 'range'` | No | `'single'` | Enables single or range selection |
| `value` | `Date \| [Date, Date] \| null` | No | `null` | Controlled value |
| `onChange` | `(value: Date \| [Date, Date] \| null) => void` | Yes | — | Called when selection changes |
| `minDate` | `Date` | No | — | Earliest selectable date |
| `maxDate` | `Date` | No | — | Latest selectable date |
| `placeholder` | `string` | No | `'Select date'` | Input placeholder when no value |
| `disabled` | `boolean` | No | `false` | Disables the input and popover |
| `error` | `string` | No | — | Error message to display below input |

---

## States

| State | Description |
|-------|-------------|
| Default | Input field with placeholder; no popover |
| Focused | Input has focus ring; popover opens |
| Popover open | Calendar grid visible below input |
| Date selected | Input shows formatted date; calendar highlights selection |
| Range in-progress | First date selected; hovering shows range preview highlight |
| Range selected | Both dates selected; range highlighted in calendar |
| Disabled | Input and trigger grayed out; popover cannot open; `cursor: not-allowed` |
| Error | Red border on input; error message below |
| Loading | [TO BE DEFINED] |

---

## Interactions

| Trigger | Element | Action |
|---------|---------|--------|
| Click | Input / trigger button | Open popover |
| Click | Date cell (enabled) | Select date; close popover (single mode); or set range start (range mode) |
| Hover | Date cell | Highlight date; in range mode with start selected, preview the would-be range |
| Click | Date cell (disabled) | No action; cell is visually muted |
| Click | Outside popover | Close popover without changing selection |
| Press Enter/Space | Focused date cell | Select date (same as click) |
| Press ArrowKeys | Popover open | Move focus between date cells |
| Press Escape | Popover open | Close popover without changing selection |
| Press Tab | Popover open | Move to next interactive element; close popover |

---

## Accessibility

- **ARIA role**: `combobox` on the trigger input with `aria-haspopup="dialog"` and `aria-expanded`
- **Calendar role**: `dialog` on popover with `aria-label="Calendar"`
- **Date cells**: `role="button"` with `aria-label="[Day] [Month] [Year]"` (e.g. "15 January 2024")
- **Selected state**: `aria-selected="true"` on selected date cells
- **Disabled dates**: `aria-disabled="true"` on out-of-range cells
- **Focus management**: On popover open, focus moves to the selected date or today's date. On close, focus returns to the trigger input.
- **Color contrast**: Date cells must meet 4.5:1 contrast ratio in all states
- **Motion**: Calendar open/close animation respects `prefers-reduced-motion`

---

## Acceptance Criteria

- [ ] In single mode, clicking a date sets `value` and closes the popover
- [ ] In range mode, clicking two dates sets `value` as `[startDate, endDate]`; clicking a third date resets and starts a new range
- [ ] Dates before `minDate` or after `maxDate` are rendered disabled and cannot be selected
- [ ] Pressing Escape closes the popover without changing the current value
- [ ] Arrow key navigation moves focus between all enabled date cells
- [ ] Screen reader announces the selected date when focus moves to a selected cell
- [ ] Component passes axe accessibility audit with zero violations

## Edge Cases

- **`minDate` === `maxDate`**: Only that single date is selectable
- **Range where start > end** (programmatic): Swap dates silently to maintain start < end invariant
- **`value` outside min/maxDate bounds**: Log a console warning; render value as-is without clamping
- **Month with 28 days (February)**: Calendar grid renders correctly without empty row artifacts
```

**What changed:** Added all props with types and defaults, 8 component states, full interaction table with keyboard events, specific ARIA requirements, 7 testable ACs, and 4 edge cases.

---

## Example 3: Bug Fix

### Before (F-grade, score: 22/100)

```
Title: Fix login bug

Users are getting an error when they try to log in. It started happening after the deployment last week.
```

**Why it fails:**
- No reproduction steps
- No expected vs actual behavior
- No error message captured
- No environment information
- No acceptance criteria

---

### After (A-grade, score: 90/100)

```markdown
## Overview

**Summary**: Users on the Starter plan receive a 500 error when attempting to log in with Google OAuth after the v2.3.1 deployment on 2024-01-15.
**Severity**: Critical — affects ~30% of login attempts for Starter plan users.
**Reported by**: Customer support (12 tickets in 2 days)
**Affects**: v2.3.1+; Google OAuth login; Starter plan users only

---

## Reproduction Steps

1. Navigate to `https://app.example.com/login`
2. Click "Continue with Google"
3. Complete Google OAuth consent (any Google account)
4. Observe the application callback at `/auth/callback?code=...`
5. **Actual**: Page shows "Something went wrong. Error ID: [random UUID]" and a 500 status in the network tab
6. **Expected**: User is redirected to `/dashboard` and logged in

**Reproduction rate**: ~100% reproducible for Starter plan users; Pro plan users are not affected.

---

## Expected vs Actual Behavior

**Expected**: After completing Google OAuth, user is redirected to `/dashboard` with an active session cookie set.

**Actual**: The server returns HTTP 500. Application logs show:
```
TypeError: Cannot read properties of undefined (reading 'planId')
  at UserService.getPlanFeatures (user-service.js:142)
  at OAuthController.handleCallback (oauth-controller.js:67)
```

---

## Environment

| Field | Value |
|-------|-------|
| Affected versions | v2.3.1 (deployed 2024-01-15), not present in v2.3.0 |
| Auth method | Google OAuth only (email/password login works) |
| Affected plans | Starter (Free tier); Pro plan users are not affected |
| Browser | Reproducible in Chrome 121, Firefox 122, Safari 17 |
| OS | macOS, Windows, iOS — all affected |

---

## Root Cause Hypothesis

The v2.3.1 release added a `planId` field requirement to `UserService.getPlanFeatures()`. Starter plan users were created before `planId` was introduced and have `planId: undefined` in their user records. The OAuth callback calls this method synchronously without a null check.

Suspected files: `src/services/user-service.js:142`, `src/controllers/oauth-controller.js:67`

---

## Fix Acceptance Criteria

- [ ] Starter plan users with `planId: undefined` can successfully log in via Google OAuth
- [ ] After login, user is redirected to `/dashboard` with an active session
- [ ] Pro plan users are unaffected (regression check)
- [ ] Email/password login continues to work for all plan types
- [ ] A new automated test covers the Starter-plan-user OAuth callback scenario

## Regression Risk

**Areas at risk**: Any code path that calls `UserService.getPlanFeatures()` — check `oauth-controller.js`, `billing-controller.js`, and `subscription-middleware.js`

**Tests to verify**: Run `npm test -- --grep "OAuth"` and `npm test -- --grep "UserService"`
```

**What changed:** Added specific error message with stack trace, version/environment table, precise reproduction steps, a root cause hypothesis pointing to specific files, 5 concrete ACs, and explicit regression risk with test commands.

---

## Example 4: Backend Feature

### Before (C-grade, score: 58/100)

```
Title: Add email notifications for order status changes

When an order status changes, send an email to the customer. Emails should be sent for: order placed, order shipped, order delivered.
```

**Why it's C-grade:** Basic scope is clear, but there's no error handling, no data model, no performance or security details, and AC are implied rather than stated.

---

### After (A-grade, score: 89/100)

```markdown
## Overview

**What**: Implement a transactional email notification system that sends emails to customers when their order status changes.
**Why**: Customers are contacting support at a rate of 40 tickets/day to check order status. Automated notifications will reduce support load and improve satisfaction.
**Who**: All customers who place orders; email sent to the address on file at order creation.

## Scope

**In scope:** Order status change events: `placed`, `shipped`, `delivered`, `cancelled`. Email delivery via SendGrid. Retry on failure. Customer opt-out respected.
**Out of scope:** SMS notifications; push notifications; vendor/admin notifications; marketing emails; email template designer.

---

## Events and Triggers

| Order status | Email trigger | Delay |
|-------------|---------------|-------|
| `placed` | Immediately on order creation | 0 min |
| `shipped` | On `status` field update to `shipped` | 0 min |
| `delivered` | On `status` field update to `delivered` | 0 min |
| `cancelled` | On `status` field update to `cancelled` | 0 min |

---

## Data Model: `notification_log`

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | `uuid` | PK | |
| `orderId` | `uuid` | FK → orders.id, indexed | |
| `userId` | `uuid` | FK → users.id, indexed | |
| `event` | `enum` | `placed/shipped/delivered/cancelled`, not null | |
| `status` | `enum` | `pending/sent/failed`, default `pending` | |
| `attempts` | `integer` | default 0 | Retry count |
| `sentAt` | `timestamp` | nullable | |
| `failedAt` | `timestamp` | nullable | |
| `errorMessage` | `text` | nullable | SendGrid error or system error |
| `createdAt` | `timestamp` | not null | |

---

## Business Rules

1. **Opt-out respected**: If `users.emailNotifications = false`, skip the notification and log with `status: skipped`.
2. **No duplicate sends**: If a `notification_log` entry already exists for `(orderId, event)` with `status: sent`, do not resend.
3. **Retry on failure**: Retry up to 3 times with exponential backoff (1 min, 5 min, 25 min). After 3 failures, set `status: failed` and alert on-call.
4. **Email address snapshot**: Use the email address from `orders.customerEmail` at send time, not the current `users.email`, in case the user updated their email after ordering.

---

## External Dependencies

| Dependency | Contract | Failure behavior |
|------------|----------|-----------------|
| SendGrid API | `POST /v3/mail/send` — expect 202 on success | Retry up to 3 times; alert after 3 failures |
| Order service | Publishes `order.status_changed` events to the `orders` SNS topic | Worker polls queue; if queue unavailable, emails are delayed (not lost) |

---

## Acceptance Criteria

- [ ] When order status changes to `placed`, an email is sent to `orders.customerEmail` within 60 seconds
- [ ] When order status changes to `shipped`, `delivered`, or `cancelled`, an email is sent within 60 seconds
- [ ] If `users.emailNotifications = false`, no email is sent; `notification_log` entry is created with `status: skipped`
- [ ] If SendGrid returns a 5xx error, the notification is retried up to 3 times before marking `status: failed`
- [ ] If an email for `(orderId, event)` was already sent successfully, it is not sent again even if the event fires twice
- [ ] `notification_log` entries are created for every event attempt, including skipped and failed ones

## Edge Cases

- **Order cancelled immediately after being placed**: Both `placed` and `cancelled` notifications are sent in order.
- **Customer email is invalid format**: Log `status: failed` with `errorMessage: "Invalid email address"`. Do not retry.
- **SendGrid outage > 25 min**: After 3 retries, mark `status: failed`. Create PagerDuty alert. Do not block order service.
- **User deleted account before email sends**: Send to `orders.customerEmail` regardless (email snapshot rule applies).

## Performance

- Expected volume: ~500 order events/day at current scale, up to 5,000 at 10× growth
- Worker must process events within 60 seconds of the SNS message being published
- `notification_log` table must be indexed on `(orderId, event)` for duplicate-check query performance
```

**What changed:** Added event trigger table, full data model, 4 explicit business rules, external dependency failure behavior, 6 testable ACs with specific conditions, and 4 edge cases including SendGrid outage handling.

---

## Calibration Notes

When deciding how much to rewrite, use these guidelines:

- **Preserve original prose** if it accurately describes the problem or context. Restructure it, don't replace it.
- **Grade A does not mean verbose.** A 200-word spec with all required sections can score higher than a 1,000-word spec with vague language.
- **The goal is agent autonomy.** Ask yourself: "Could an AI agent implement this spec without asking a single question?" If yes, the rewrite is done.
- **`[TO BE DEFINED]` is always acceptable.** An honest placeholder is better than a fabricated value.
