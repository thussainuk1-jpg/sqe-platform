# Inns & Quill — SQE1 Practice Platform

A real, deployable SQE1 (Functioning Legal Knowledge) practice platform.
Free for students, supported by reader donations. Admin-authored question bank.
No AI-generated content.

**Stack:** Next.js 14 (App Router) · TypeScript · Tailwind · Prisma · Postgres · NextAuth · Stripe

---

## What it does

**For students**
- Sign up with email + password
- **Quick Practice** — pick a subject, get one question at a time, instant feedback with model explanation
- **Timed Mocks** — full mock papers with countdown timer, question navigator, auto-submit
- **My Progress** — overall accuracy, weakest subjects, recent mock results
- **Donate** — optional Stripe-powered donations

**For you (admin)**
- Dashboard with live stats
- Create / edit / delete questions (subject, difficulty, 2–8 options, mark correct, model explanation, draft or publish)
- Build mock papers by selecting from your published question bank
- View aggregate donations

---

## 1. Run locally (15 minutes)

### a. Prerequisites
- Node.js 18+ and npm
- A Postgres database. Easiest free option: [Neon](https://neon.tech) — sign up, create a project, copy the connection string

### b. Install
```bash
npm install
```

### c. Configure
```bash
cp .env.example .env
```
Open `.env` and fill in:
- `DATABASE_URL` — your Neon (or other) Postgres connection string
- `NEXTAUTH_SECRET` — generate with: `openssl rand -base64 32`
- `NEXTAUTH_URL` — keep `http://localhost:3000` for local dev

Leave `STRIPE_*` blank for now — the site will run fine without donations.

### d. Push the database schema
```bash
npm run db:push
```

### e. Create your admin account
```bash
node scripts/create-admin.mjs your@email.com YourStrongPassword "Your Name"
```

### f. (Optional) Seed placeholder questions to test the flow
```bash
npm run db:seed
```
This inserts 3 clearly-marked placeholder questions so you can immediately test practice mode. **Delete them before launching to real users.**

### g. Run
```bash
npm run dev
```
Open http://localhost:3000

Sign in with your admin email → you'll see the "Admin" link in the nav.

---

## 2. Deploy to production (free tier, ~30 minutes)

### a. Database — Neon
1. Create a free Postgres database at https://neon.tech
2. Copy the connection string (with `?sslmode=require` at the end)

### b. Hosting — Vercel
1. Push this code to a GitHub repo
2. Go to https://vercel.com → New Project → import the repo
3. Add environment variables:
   - `DATABASE_URL` — your Neon string
   - `NEXTAUTH_SECRET` — `openssl rand -base64 32`
   - `NEXTAUTH_URL` — your Vercel URL (e.g. `https://inns-quill.vercel.app`)
4. Deploy. Vercel runs `npm run build` which includes `prisma generate`.

### c. Push the schema to production
From your local terminal, with `.env` pointing temporarily at your **production** `DATABASE_URL`:
```bash
npm run db:push
```
(Or use Vercel's CLI / a one-off Vercel function.)

### d. Create your admin in production
Same script, against the production database:
```bash
DATABASE_URL="<prod url>" node scripts/create-admin.mjs you@example.com Pass "Your Name"
```

### e. Donations (optional) — Stripe
1. Create a [Stripe](https://dashboard.stripe.com) account
2. Copy your **secret key** (start with `sk_test_…` for testing)
3. In Vercel, add `STRIPE_SECRET_KEY`
4. Create a webhook endpoint pointing to `https://YOUR-DOMAIN/api/donate/webhook`
   - Listen for: `checkout.session.completed`, `checkout.session.expired`, `checkout.session.async_payment_failed`
5. Copy the webhook signing secret into Vercel as `STRIPE_WEBHOOK_SECRET`
6. Redeploy

### f. Domain
In Vercel → Project → Settings → Domains, point your own domain (e.g. `innsandquill.co.uk`) at the deployment. Update `NEXTAUTH_URL` to match.

---

## 3. Operating the site

### Adding questions
1. Sign in as admin → click **Admin** → **+ New question**
2. Fill in stem, options (mark exactly one as correct), explanation
3. Tick **Publish** when ready

### Building a mock paper
1. Admin → Mocks → **+ New mock**
2. Set title, description, duration (default 170 mins — the SRA paper duration)
3. Add questions from your published bank
4. Tick **Published** in mock settings to make it live to students

### Managing donations
- Donations are recorded in the `Donation` table with status `pending`/`succeeded`/`failed`
- The webhook automatically marks them `succeeded` when Stripe confirms payment
- Aggregate totals show on the admin dashboard

---

## 4. Important notes (read these)

### Content quality is on you
The placeholder seed questions are **placeholders only** — they're not SQE-quality.
Writing real SQE1 questions takes months. Plan to author 200+ MCQs across all FLK subjects
before serious launch. Get them reviewed by another qualified solicitor before publishing.

### Regulatory considerations
- You are a UK Solicitor. Operating a paid (even donation-funded) legal-education platform
  may interact with **SRA Principles** and **Code of Conduct** rules — particularly around
  publishing legal information, advertising, and the boundary between "education" and "advice."
- The footer and disclaimer pages make it clear this is **educational content, not legal advice**,
  and that you are **not affiliated with the SRA**. Review those carefully and adapt them.
- **Don't** use the word "guarantee" or imply SRA endorsement anywhere.
- Consider consulting the SRA's guidance on solicitor-led businesses before commercial launch.

### Data & GDPR
- Passwords are bcrypt-hashed; you never see plaintext
- Stripe handles all card details — your database only stores the session reference
- The privacy page is a starting template — have it reviewed if you're processing significant
  user numbers, and register with the ICO if required (small operators often are required to)

### Bug reports and roadmap
Things you may want to add later (deliberately left out to keep the MVP focused):
- "Bookmark question" and "report this question" buttons
- CSV import for questions (bulk upload)
- Per-question difficulty calibration based on actual student accuracy
- A second admin role ("reviewer") who can edit drafts but not publish
- Email verification on signup
- Password reset flow

---

## File structure

```
src/
  app/
    page.tsx                   ← homepage
    login/, signup/            ← auth
    practice/, mock/           ← student-facing
    admin/                     ← your admin panel
    api/                       ← all backend routes
    about/, privacy/, terms/   ← static pages
  components/                  ← shared client components
  lib/                         ← db, auth, stripe, subjects, admin guard
  types/                       ← typescript augmentations
prisma/
  schema.prisma                ← the data model
scripts/
  create-admin.mjs             ← promote/create an ADMIN user
  seed.mjs                     ← optional placeholder questions
```

---

Built with care. If something breaks, the logs are in Vercel → Project → Deployments → Logs.
