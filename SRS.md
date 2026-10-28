Your requirements define a clear, production-oriented B2B lead generation agency website. The solution should combine a professional marketing website with a secure inquiry collection system and a private admin dashboard, without adding unnecessary CRM or sales pipeline features.

## 1. Recommended architecture

React + TypeScript + Vite

Responsive public website and protected admin interface

Supabase

Authentication, PostgreSQL database, Row Level Security and Edge Functions

Email provider

Owner notification when an inquiry is submitted

Vercel / Netlify

Frontend deployment and hosting

## 2. Website pages and features

| Page | Main functionality |
| --- | --- |
| Home | Hero section, value proposition, services overview, trust signals and CTA |
| Services | Four service offerings with benefits and inquiry links |
| About Us | Company story, approach and differentiators |
| Results / Case Studies | Measurable outcomes and case studies based on verified company data |
| Contact | Validated inquiry form with success and error states |
| Admin Login | Secure Supabase authentication |
| Admin Dashboard | Search, view, update status and delete inquiries |

The visual design should use a modern B2B aesthetic: strong typography, restrained colors, clear calls to action, mobile-friendly layouts and consistent navigation.

## 3. Contact form and inquiry workflow

The contact form will collect:

- Full name
- Work email
- Company name
- Phone number (optional)
- Service interested in
- Message

## Inquiry lifecycle

1. Visitor submits the form
   Validate required fields, email format and message length.
2. Supabase stores the inquiry
   A database insert policy allows public submissions without granting public read access.
3. Email notification is triggered
   A Supabase Edge Function sends an email to the configured business owner through an email provider.
4. Admin signs in
   Supabase Auth verifies the administrator's identity and the database checks authorization.
5. Admin manages inquiries
   Search submissions, inspect details, change status to New, Read or Replied, and delete records.

## 4. Admin dashboard design

The dashboard should be simple and focused on managing incoming leads.

## Inquiry management

Illustrative preview

New

# 12

Read

# 8

Replied

# 5

Search by name, company or email

Example Company Ltd.

Contact: Alex Morgan · Appointment Setting

New

Northstar Solutions

Contact: Jordan Lee · B2B Lead Generation

Read

Illustrative interface only; the counts and sample inquiries are not real database records.

Each inquiry should have a detail view showing all submitted fields, submission timestamp, current status and available actions. Deletion should require confirmation.

## 5. Database and security requirements

Recommended database table: `inquiries`.

| Column | Type | Purpose |
| --- | --- | --- |
| `id` | UUID | Unique inquiry identifier |
| `full_name` | Text | Contact's name |
| `work_email` | Text | Contact email |
| `company_name` | Text | Business name |
| `phone` | Text, nullable | Optional phone |
| `service_interested_in` | Text | Selected service |
| `message` | Text | Inquiry details |
| `status` | Text or enum | New, Read, Replied |
| `created_at` | Timestamp | Submission time |

Security is a core acceptance requirement, not an optional enhancement.

- Enable Row Level Security (RLS) on the `inquiries` table.
- Allow anonymous visitors to insert validated inquiries only.
- Do not grant anonymous users `SELECT`, `UPDATE` or `DELETE` access.
- Require authenticated and explicitly authorized administrators for reading, updating and deleting inquiries.
- Protect the admin route in the frontend, while enforcing the actual permissions in Supabase.
- Use a server-verified admin role or authorization table; do not trust a user-editable profile field or frontend-only role check.
- Keep the Supabase service-role key and email-provider API key exclusively in secure server-side secrets.
- Add rate limiting or abuse protection, server-side validation and safe error handling.
- Do not return private inquiry information in public API responses.

Important: A login screen alone does not secure the database. Authorization must be enforced by Supabase policies for every operation.

## 6. Implementation and deployment plan

1. Build the frontend
   Create reusable React components, page routing, responsive navigation, accessible forms and SEO metadata.
2. Configure Supabase
   Create the database schema, migrations, RLS policies, authentication settings and administrator authorization.
3. Implement inquiry submission
   Connect the form to Supabase, validate inputs and display appropriate submission states.
4. Implement notifications
   Create an Edge Function to send owner notifications, with secrets configured in the Supabase project.
5. Build the private dashboard
   Implement login, session handling, inquiry search, detail views, status updates and deletion.
6. Test and deploy
   Verify access controls, mobile responsiveness, SEO, email delivery, error handling and production environment variables before deployment.

Suggested project structure:

text

```
src/
  components/
  pages/
    Home.tsx
    Services.tsx
    About.tsx
    CaseStudies.tsx
    Contact.tsx
    AdminLogin.tsx
    AdminDashboard.tsx
  lib/
    supabase.ts
  types/
  App.tsx
  main.tsx

supabase/
  migrations/
  functions/
    notify-inquiry/

.env.example
package.json
vite.config.ts
```

Use environment variables for the frontend Supabase URL and publishable key, and configure privileged secrets separately in Supabase. Deploy the frontend to Vercel or Netlify with appropriate SPA routing and production configuration.

## 7. Final acceptance checklist

- [ ]All five public pages are responsive and navigable.
- [ ]All four services are presented clearly.
- [ ]The contact form validates inputs and stores inquiries.
- [ ]Successful submissions trigger an owner email notification.
- [ ]Public visitors cannot read or modify stored inquiries.
- [ ]Only authorized administrators can manage submissions.
- [ ]Search, detail views, status updates and deletion work correctly.
- [ ]No privileged credentials are exposed in frontend code.
- [ ]SEO metadata, page titles, accessibility and loading/error states are implemented.
- [ ]Database permissions and authentication are tested before launch.

## 8. Information needed to finalize the website

The technical scope is sufficiently defined. The remaining choices are mainly branding and deployment configuration.

Company or agency name

Visual directionModern dark navy Modern dark navy — premium B2B

Clean white and blue — corporate

Bold black and orange — sales-focused

Frontend hostingVercel Vercel

Netlify

Email notification providerResend Resend

SendGrid

Use an existing email provider

 Finalize implementation plan

The final deliverable should be a deployable, fully functional website—not just a visual mockup—with real database integration, server-side notifications, and verified access controls.