# Freedom Work Global — Project Master Rules

These rules are specific to Freedom Work Global and must be strictly followed alongside universal engineering rules.

---

## 1. Architectural Integrity & Scope
- **Core Stack**: React (v18/19), TypeScript (strict mode), Vite, Tailwind CSS, Supabase (PostgreSQL, Auth, RLS, Edge Functions).
- **YAGNI / Minimalist Scope**: Do not build complex CRM pipelines, ticketing platforms, or multi-tenant billing. Freedom Work Global is a high-converting B2B agency marketing site with a secure lead collection pipeline and a private inquiry management dashboard.
- **Routing**: Client-side routing with protected route guards for `/admin` and `/admin/dashboard`. Unauthorized users must be redirected to `/admin/login`.

---

## 2. Supabase & Database Security (Non-Negotiable)
- **Row Level Security (RLS)**:
  - RLS must be enabled on the `inquiries` table at all times.
  - `anon` role (public visitors): **`INSERT` ONLY** with validated columns. Strictly forbidden from `SELECT`, `UPDATE`, or `DELETE`.
  - `authenticated` role: Must NOT have automatic access to inquiries simply by logging in. Access must be verified against an explicit `admin_users` table or database authorization function (`is_admin()`).
- **Secrets Isolation**:
  - Only `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` may appear in frontend `.env` files.
  - `SUPABASE_SERVICE_ROLE_KEY` and third-party email provider keys (Resend/SendGrid) must **NEVER** be committed, imported in frontend code, or included in client bundles. They belong exclusively in Supabase Edge Function environment secrets.
- **Data Protection**:
  - Never return private inquiry details in public API responses or public form submission responses.

---

## 3. Inquiry Form & Data Validation
- **Schema Validation**: Validate all contact inputs using **Zod** both client-side and in Edge Functions:
  - `full_name`: string, min 2 characters.
  - `work_email`: valid email format (reject generic temp domains if possible).
  - `company_name`: string, min 2 characters.
  - `phone`: optional string, valid phone format if provided.
  - `service_interested_in`: enum matching the 4 agency offerings.
  - `message`: string, min 10 characters, max 2000 characters.
- **Race Condition & Spam Guard**:
  - Always lock the submit button during submission (`disabled={isSubmitting}`) to prevent duplicate lead creation.
  - Implement a hidden honeypot field (`website` / `company_fax`) to drop automated bot submissions before hitting the database.

---

## 4. UI, Spacing & Design System (from SRS & freedom_work.md)
Follow the exact spacing and typography scales established in `freedom_work.md`:
- **Spacing Scale**:
  - Icon & label: `8px` (`gap-2`)
  - Heading & paragraph: `16px` (`mb-4` / `space-y-4`)
  - Paragraph & CTA: `24px` (`mb-6` / `mt-6`)
  - Inside cards: `24px` (`p-6`)
  - Between service cards: `16–24px` (`gap-4` to `gap-6`)
  - Between major sections: `64–96px` (`py-16` to `py-24`)
  - Page horizontal padding: Desktop `32–48px` (`px-8` to `px-12`), Mobile `16–24px` (`px-4` to `px-6`)
- **Typography & Aesthetics**:
  - Font families: Inter or Manrope (clean, readable sans-serif).
  - Visual tone: Modern dark navy B2B corporate aesthetic with restrained accent colors and high text contrast.
  - Visual hierarchy: Prominent hero value proposition -> Clear primary CTA -> Social proof/credibility -> 4 core services -> Case studies with verified metrics -> Final CTA.
- **Mobile-First Touch & Layout**:
  - Minimum `44x44px` interactive touch targets for all buttons, links, and inputs.
  - `overflow-x-clip` on main containers to eliminate horizontal jitter.
  - Heading line-heights `leading-[1.15]` or greater to avoid overlapping title lines.
  - Zero opacity animation on elements using CSS `backdrop-filter: blur`.

---

## 5. Code Quality & State Management
- **No Loose Types**: Every database row, API response, and form payload must have an explicit TypeScript interface or type alias. No `any`.
- **Query & Mutation Invalidation**: When an admin updates an inquiry status (`New` -> `Read` -> `Replied`) or deletes a record, synchronize UI state immediately via React Query cache invalidation or optimistic update.
- **Destructive Actions**: Deleting an inquiry in the admin dashboard must require explicit confirmation.
