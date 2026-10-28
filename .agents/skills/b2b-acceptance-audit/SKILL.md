---
name: b2b-acceptance-audit
description: >-
  Comprehensive acceptance checklist and automated audit runbook for Freedom Work Global.
  Use to audit public marketing pages, responsive layouts, inquiry submission flow, admin dashboard,
  and security criteria against SRS.md before committing or deploying.
---

# Freedom Work Global — Acceptance & Audit Runbook

This skill outlines the step-by-step verification process to ensure the application satisfies all requirements in `SRS.md`, `UI.md`, and `freedom_work.md`.

---

## 1. Public Pages Audit (5 Core Pages)

### Home Page
- [ ] **Exact Narrative Flow**: Value Proposition → Primary CTA → Social Proof/Credibility → 4 Core Services → Case Studies → Final CTA.
- [ ] **Hero Section**: Clear B2B value proposition headline, subtitle, and instant lead action button.
- [ ] **Visual Hierarchy**: Prominent headings, controlled body line lengths, and consistent spacing scale (64–96px section padding).

### Services Page
- [ ] Displays all 4 defined service offerings.
- [ ] Each service card includes tangible client benefits and direct link/CTA to the inquiry form.
- [ ] Card spacing respects the 16–24px gap standard.

### About Us Page
- [ ] Company narrative, agency philosophy, and distinct differentiators.
- [ ] Professional team or methodology breakdown with high-contrast text.

### Results & Case Studies Page
- [ ] Measurable, metrics-driven outcomes (e.g., qualified appointments booked, pipeline value generated).
- [ ] High-credibility layout with client problem, solution, and numerical results.

### Contact Page
- [ ] Inquiry form collecting: Full name, work email, company name, phone (optional), service interested in, and message.
- [ ] Client-side Zod validation with friendly inline error states.
- [ ] Anti-spam honeypot field included and hidden from human users.
- [ ] Loading spinner on submit button + double-click lock (`disabled={isSubmitting}`).
- [ ] Immediate positive feedback screen or message upon successful submission.

---

## 2. Admin Dashboard & Authentication Audit

### Admin Authentication
- [ ] Protected route guard on `/admin/dashboard` redirecting unauthenticated visitors to `/admin/login`.
- [ ] Secure Supabase Auth sign-in with clear error messages for invalid credentials.
- [ ] Sign-out functionality terminates session and clears local auth tokens.

### Dashboard Operations
- [ ] **Summary Metric Badges**: Real-time counts for `New`, `Read`, and `Replied` inquiries.
- [ ] **Search & Filter**: Fast client/server search filtering by contact name, company name, or email.
- [ ] **Detail Modal / Drawer**: View full inquiry text, contact metadata, and timestamp.
- [ ] **Status Toggles**: Seamless mutation updating inquiry status with immediate UI sync.
- [ ] **Deletion Guard**: Deleting an inquiry displays a confirmation modal before executing the database delete.

---

## 3. Production Security & RLS Verification

Run these explicit tests before release:
- [ ] **Zero Secret Leakage**: Search build bundle (`dist/`) for `service_role` or email API keys.
- [ ] **Public DB Access Probe**: Attempt a direct `SELECT * FROM inquiries` with the `anon` key. Must return empty results.
- [ ] **Non-Admin Auth Probe**: Create a regular authenticated user not listed in `admin_users`. Attempt to read inquiries—must return 0 records.
- [ ] **Input Sanitization**: Test inquiry text fields with HTML/script characters to ensure zero XSS rendering in the admin dashboard.

---

## 4. Mobile & Performance Checklist

- [ ] Minimum `44x44px` interactive touch targets for all buttons and nav links.
- [ ] Mobile viewport tested down to 360px width with zero horizontal scroll (`overflow-x-clip`).
- [ ] Clean font rendering using Inter or Manrope.
- [ ] No choppy animations over CSS `backdrop-filter: blur`.
- [ ] Production build succeeds cleanly (`npm run build`) with zero TypeScript errors.
