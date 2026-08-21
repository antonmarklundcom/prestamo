# prestamo.com.py — Master Plan

This is the build plan for the whole system. It is written so that future
sessions (Opus for architecture-heavy PRs, Sonnet for content/CRUD PRs) can
execute one PR at a time without re-deriving context. Read
`docs/ARCHITECTURE.md` first, always; read `docs/LEGAL-PARAGUAY.md` before any
PR touching the form, consent, policies, advertising copy, or delivery.

**Business in one line:** capture consenting loan applicants on prestamo.com.py
and sell those leads to licensed Paraguayan lenders. We are a marketing
partner, never a lender.

## Ground rules for every PR

- Stack: Next.js 15 App Router + TypeScript + Tailwind + Drizzle + MySQL
  (Hostinger). Follow the `nodejs-mysql-hostinger-stack` conventions.
- Site language: Paraguayan Spanish (voseo). Code/comments/commits: English.
- Money: integer ₲, `es-PY` formatting. Phones normalized `+595…`.
- Never collect more personal data than the lead schema defines. Adding a form
  field is a legal decision, not a UX decision — it requires updating
  LEGAL-PARAGUAY.md and the consent text version in the same PR.
- Every PR: builds clean (`npm run build`), lints, seeds/scripts idempotent,
  `.env.example` updated when a new env var appears.
- One PR = one deliverable. Keep them small; the sequence below is ordered by
  dependency.

## Phase 0 — Legal & company setup (operator work, no code)

Blocking for *launch*, not for *build* — coding can start in parallel.
Checklist lives in `docs/OPERATOR-CHECKLIST.md`.

## Phase 1 — Launchable funnel (PR-1 … PR-6)

Goal: a live site that legally captures leads into our DB + VenderCRM, worked
manually. Revenue possible via manual forwarding under signed agreements.

- **PR-1 — Scaffold + schema.** create-next-app, Drizzle config, connection
  pool, full schema from ARCHITECTURE.md (leads, consents, consent_texts,
  buyers, deliveries, invoices, users, data_requests, activity_log), initial
  migration, seed script with demo data (valid RUC DVs, ₲ amounts), `.env.example`.
  *Model: Opus. Acceptance: `npm run build` + seed runs twice safely.*
- **PR-2 — Lead capture API.** `POST /api/leads`: zod validation, +595 phone
  normalization, idempotency dedup (sha256(phone|YYYY-MM-DD-HH)), honeypot
  check, consent row insert (versioned text), VenderCRM push per the
  `vendercrm-lead-capture` rules (server-side key, 10s timeout, never blocks
  visitor), basic rate limit. Unit tests for validation + dedup.
  *Model: Opus. Acceptance: duplicate submit → one lead; CRM down → lead still stored.*
- **PR-3 — Landing page + form.** `/`, `/solicitar` (multi-step form:
  amount/term → situation → contact+consent), `/gracias`. Consent checkbox
  unticked by default, links to policy, exact wording from LEGAL doc. Design
  per `web-design-system` skill (this is a trust-critical financial vertical:
  sober palette, real trust signals, no fake bank logos). Mobile-first,
  WhatsApp-first contact affordances.
  *Model: Opus for structure, Sonnet ok for polish. Acceptance: Lighthouse ≥90 mobile, form round-trip verified in VenderCRM.*
- **PR-4 — Legal pages.** `/politica-de-privacidad`, `/terminos-y-condiciones`,
  `/quienes-somos` (with "no somos una entidad financiera" disclosure),
  `/baja-de-datos` (data deletion/revocation request form → `data_requests`).
  Content drafted from LEGAL-PARAGUAY.md templates — **operator must have a PY
  lawyer review before ads run** (flagged in OPERATOR-CHECKLIST).
  *Model: Sonnet. Acceptance: every page reachable from footer; policy version constant matches `consent_texts` seed.*
- **PR-5 — Technical SEO + analytics.** `sitemap.ts`, `robots.ts`, per-page
  `generateMetadata`, canonical, JSON-LD (Organization, FAQPage on landing),
  og-images, `next/font`, GA4 + Meta Pixel with Consent Mode (fire marketing
  pixels only after cookie consent), conversion event on `/gracias`.
  *Model: Sonnet.*
- **PR-6 — Deploy.** Hostinger Node.js setup per `nextjs-deploy-hostinger`
  (env vars, Remote MySQL allowlist, domain mapping prestamo.com.py, SSL).
  Smoke-test script. Document the exact slot/account used in this file.
  *Model: Sonnet with the deploy skill. Acceptance: production form submission lands in DB + VenderCRM.*

## Phase 2 — Admin & ops (PR-7 … PR-10)

Goal: run the business without touching the DB by hand.

- **PR-7 — Auth + admin shell.** iron-session + bcrypt users, role enum,
  `requireRole` helper on every server action, `/admin` layout + dashboard
  (today's leads, sources, statuses). *Model: Opus.*
- **PR-8 — Lead management.** `/admin/leads`: list/filter/search, detail view
  with consent + attribution history, status transitions, wa.me deeplink,
  manual "mark delivered to buyer X" (creates a `deliveries` row), CSV export
  (logged in activity_log). *Model: Sonnet.*
- **PR-9 — Buyer management.** `/admin/buyers` CRUD: filters JSON editor,
  price per lead, delivery config, DPA-signed flag (blocks delivery when unset).
  *Model: Sonnet.*
- **PR-10 — Data-subject requests.** `/admin/solicitudes`: work the
  `data_requests` queue — anonymize lead, revoke consent, record buyer
  notification; hard-coded SLA countdown per LEGAL doc. Retention cron script
  (`scripts/cron-retention.ts`) purging expired undelivered leads.
  *Model: Opus (data-destruction logic — needs care).*

## Phase 3 — Automated delivery & billing (PR-11 … PR-14)

Goal: leads flow to buyers automatically; buyers get invoiced.

- **PR-11 — Delivery engine.** Matcher (buyer filters + dedup window + DPA
  check) → `deliveries` rows → dispatch by channel: email (formatted lead
  sheet) and webhook (signed JSON POST, retry w/ backoff). Admin override UI.
  *Model: Opus. Acceptance: rejected/duplicate leads never billable.*
- **PR-12 — Buyer feedback loop.** Signed status-callback endpoint + manual
  admin controls so buyers can mark accepted/rejected/returned; returns window
  per contract; delivery quality report per buyer. *Model: Sonnet.*
- **PR-13 — Invoicing.** Monthly invoice generation per buyer from billable
  deliveries: PY format (timbrado fields, `001-001-XXXXXXX`, IVA 10% included
  breakdown), PDF, wa.me/email send. Data kept SIFEN-ready; actual e-invoicing
  via a certified provider is an operator decision later. *Model: Opus, using
  `paraguay-business-apps` rules.*
- **PR-14 — Reporting.** `/admin/reportes`: CPL by source/campaign (utm),
  conversion funnel, revenue per buyer, lead quality (acceptance rate) per
  source. *Model: Sonnet.*

## Phase 4 — SEO growth (PR-15 … PR-17, repeatable)

- **PR-15 — City pages.** `/prestamos-en-[ciudad]` template + first 6 cities
  (Asunción, Ciudad del Este, Encarnación, Luque, San Lorenzo, Lambaré) with
  genuinely local content, not spun text. *Model: Sonnet.*
- **PR-16 — Segment pages.** `/prestamos-para-[segmento]`: asalariados,
  independientes, jubilados, funcionarios públicos — each with segment-specific
  requisitos and FAQ schema. *Model: Sonnet.*
- **PR-17 — Guides hub.** `/guias`: 8–10 pillar articles (requisitos para un
  préstamo, cómo funciona Informconf, tasa de interés máxima BCP explained,
  IPS y préstamos, refinanciación…). Educational tone; rate figures only with
  BCP citation + date, per LEGAL advertising rules. *Model: Sonnet, repeat as
  content batches.*

## Phase 5 — Scale (backlog, spec before building)

- Ping-post / real-time bidding between buyers (price competition per lead)
- Buyer self-service portal (their leads, quality stats, invoices)
- WhatsApp Cloud API automation (reminder to complete form, delivery to buyers)
- Additional verticals on subfolders (tarjetas, seguros) — each is a new legal
  review, not just a new template
- A/B testing on the form (only after volume justifies it)

## Session hygiene for future builders

- After PR-6 is live, create a `prestamo-dev` project skill capturing the real
  schema, routes, env vars, deploy slot, and a known-issues log.
- Update THIS file's PR list when scope changes: it is the roadmap of record.
- Never start a PR without reading its acceptance criteria here; never merge
  one that expands data collection without a LEGAL-PARAGUAY.md change.
