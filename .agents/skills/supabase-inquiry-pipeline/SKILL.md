---
name: supabase-inquiry-pipeline
description: >-
  Procedures, schema migrations, and security runbooks for Supabase database setup,
  Row Level Security (RLS) enforcement, authentication, and Edge Function email notifications
  for Freedom Work Global. Use when designing, migrating, or testing backend inquiry handling.
---

# Supabase Inquiry Pipeline & Security Runbook

This skill guides the implementation, verification, and maintenance of the Supabase backend for Freedom Work Global.

---

## 1. Database Schema Specification

### `inquiries` Table
```sql
CREATE TYPE inquiry_status AS ENUM ('New', 'Read', 'Replied');

CREATE TABLE public.inquiries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    full_name TEXT NOT NULL,
    work_email TEXT NOT NULL,
    company_name TEXT NOT NULL,
    phone TEXT,
    service_interested_in TEXT NOT NULL,
    message TEXT NOT NULL,
    status inquiry_status NOT NULL DEFAULT 'New',
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())
);

-- Performance Indexes
CREATE INDEX idx_inquiries_status ON public.inquiries(status);
CREATE INDEX idx_inquiries_created_at ON public.inquiries(created_at DESC);
```

### `admin_users` Authorization Table
```sql
CREATE TABLE public.admin_users (
    user_id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())
);

-- Authorization Helper Function
CREATE OR REPLACE FUNCTION public.is_admin()
RETURNS BOOLEAN SECURITY DEFINER AS $$
BEGIN
    RETURN EXISTS (
        SELECT 1 FROM public.admin_users
        WHERE user_id = auth.uid()
    );
END;
$$ LANGUAGE plpgsql;
```

---

## 2. Row Level Security (RLS) Policies

Enable RLS on tables:
```sql
ALTER TABLE public.inquiries ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.admin_users ENABLE ROW LEVEL SECURITY;
```

### Inquiries Policies
```sql
-- 1. Anonymous & Public Visitors: INSERT ONLY
CREATE POLICY "Public insert only"
ON public.inquiries
FOR INSERT
TO anon, authenticated
WITH CHECK (
    char_length(full_name) >= 2 AND
    char_length(work_email) >= 5 AND
    char_length(company_name) >= 2 AND
    char_length(message) >= 10
);

-- 2. Verified Admins: SELECT, UPDATE, DELETE
CREATE POLICY "Admins full access"
ON public.inquiries
FOR ALL
TO authenticated
USING (public.is_admin())
WITH CHECK (public.is_admin());
```

### RLS Verification Checklist
Always test access using SQL or Supabase client queries before marking backend tasks complete:
1. **Anon Insert Test**: `anon` client can insert a valid inquiry.
2. **Anon Read Leak Test**: `SELECT * FROM inquiries;` as `anon` MUST return an empty array `[]`.
3. **Anon Update/Delete Block**: Attempting update or delete as `anon` must be rejected with 0 rows affected or permission denied.
4. **Non-Admin Authenticated Test**: A registered user NOT present in `admin_users` must NOT be able to view, edit, or delete inquiries.
5. **Admin Access Test**: A user whose `id` exists in `admin_users` can query, update status, and delete records.

---

## 3. Edge Function: `notify-inquiry`

Triggered upon new inquiry insertion to send transactional email notification to the agency owner.

### Environment Secrets Needed:
- `RESEND_API_KEY` (or `SENDGRID_API_KEY`)
- `NOTIFICATION_RECIPIENT_EMAIL` (agency owner email)
- `NOTIFICATION_SENDER_EMAIL` (e.g. `notifications@freedomworkglobal.com`)

### Function Workflow (`supabase/functions/notify-inquiry/index.ts`):
1. Receive database webhook or HTTP invocation with inquiry payload.
2. Verify authorization header / webhook signature.
3. Validate payload schema.
4. Send styled HTML email summary (Full Name, Company, Work Email, Service, Message, Submission Time).
5. Return clean JSON response `{ success: true }`.

---

## 4. Frontend Client Safety
- Never import `@supabase/supabase-js` with `SUPABASE_SERVICE_ROLE_KEY` in `src/`.
- Initialize client in `src/lib/supabase.ts` using only `import.meta.env.VITE_SUPABASE_URL` and `import.meta.env.VITE_SUPABASE_ANON_KEY`.
