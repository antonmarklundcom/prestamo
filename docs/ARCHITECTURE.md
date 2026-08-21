# prestamo.com.py — Architecture

## What this system is

A loan **lead generation** platform for Paraguay. We are a **marketing partner**: we
attract people looking for a personal loan, capture a qualified application (a
"lead") with explicit consent, and sell/deliver that lead to licensed lenders
(bancos, financieras, casas de crédito, fintechs). **We never lend money, never
decide on credit, and never touch loan funds.** That boundary is what keeps us
out of financial-intermediation licensing — see `docs/LEGAL-PARAGUAY.md`.

## The build decision (html/php vs Node.js vs VenderCRM-only)

Three options were on the table:

| Option | Verdict |
|---|---|
| Static HTML/PHP landing only, leads straight into VenderCRM | Fastest to launch, but VenderCRM is a sales CRM — it cannot be the legal system of record for consent logs, lead dedup, per-buyer delivery tracking, or buyer invoicing. Fine for week 1, dead end by month 2. |
| HTML/PHP landing + separate Node.js backend | Two deployables, two codebases, glue code between them. No benefit over one app. |
| **One Next.js app: landing + API + admin + MySQL** | **Chosen.** Same proven stack as propia (Next.js 15 App Router + Drizzle + MySQL on Hostinger Node.js hosting). Landing pages are statically generated (fast, SEO), the form posts to our own API route, leads live in our MySQL, and the admin panel grows in the same repo. |

**VenderCRM is still used** — but as the *operations* tool, not the system of
record. Every lead is stored in our DB first (with consent + attribution), then
pushed to VenderCRM via `POST /api/v1/leads` so leads can be worked manually
(WhatsApp follow-up, manual forwarding to lenders) from day one, before the
automated delivery engine exists. VenderCRM also holds the *B2B* pipeline:
lender prospects we are selling lead-buying agreements to.

## Stack

- **Next.js 15** (App Router, TypeScript, Tailwind) — one app: public site + API + `/admin`
- **Drizzle ORM + MySQL** on Hostinger (single DB, `connectionLimit: 8`)
- **tsx scripts** in `scripts/` for seeds, exports, one-off jobs (idempotent upserts)
- **Hostinger Node.js slot** — deploy per `nextjs-deploy-hostinger` playbook; check slot budget before creating the app
- **VenderCRM** — lead mirror + ops pipeline (server-to-server only, key in env)
- Spanish (Paraguayan, voseo) UI; `es-PY` formatting; ₲ as integers; timezone `America/Asuncion`

## Domain model (core tables)

All money in integer guaraníes. All tables get `created_at`/`updated_at`.

- **`leads`** — the product we sell.
  `id`, `full_name`, `phone` (+595-normalized), `email?`, `city`, `department`,
  `amount_requested` (₲ int), `term_months`, `monthly_income_band` (enum, banded
  — never exact salary), `employment_type` (asalariado/independiente/jubilado/…),
  `has_iva_or_ips` (bool — proxy for formality, what lenders actually ask),
  `loan_purpose` (enum), `status` (nuevo → validado → entregado → facturado /
  rechazado / duplicado), `source`, `utm_*`, `gclid/fbclid`, `page_url`,
  `ip_address`, `user_agent`, `idempotency_key` (unique).
  **We do NOT collect cédula numbers, exact salaries, or any credit-history data.**
  Data minimization is a legal strategy, not just hygiene (see LEGAL doc §data).
- **`consents`** — append-only. `lead_id`, `consent_text_version`, `purposes`
  (JSON: share_with_lenders, marketing), `granted_at`, `ip`, `user_agent`.
  Never updated, never deleted while retention applies. This table is what we
  show a regulator or a lender's compliance team.
- **`consent_texts`** — versioned copies of the exact checkbox wording + privacy
  policy version live at the time. A consent row is meaningless without the text.
- **`buyers`** — lenders who buy leads. `name`, `ruc`, `contact`, `delivery_method`
  (email/webhook/csv/manual), `delivery_config` (JSON), `price_per_lead` (₲),
  `filters` (JSON: min amount, departments, employment types), `active`,
  `dpa_signed_at` (data-processing/transfer agreement — delivery is blocked
  until set).
- **`deliveries`** — one row per lead-per-buyer. `lead_id`, `buyer_id`, `status`
  (pending/sent/accepted/rejected/returned), `sent_at`, `response` (JSON),
  `billable` (bool). Exclusive vs shared selling is a per-buyer commercial term;
  the table supports both.
- **`invoices`** + `invoice_lines` — monthly invoicing of buyers per billable
  delivery. Modeled per the PY invoicing rules (timbrado, `001-001-XXXXXXX`,
  IVA 10% on services) so a SIFEN e-invoicing provider can be bolted on later.
- **`users`** — admin auth, `role` enum (`admin` | `ops`) from day one.
- **`data_requests`** — ARCO/data-subject requests (access, delete, revoke
  consent): `phone`, `type`, `status`, `resolved_at`. Revocation must also
  propagate a notice to buyers the lead was delivered to.
- **`activity_log`** — audit trail on anything money- or data-adjacent.

## Lead flow

```
visitor (ads/SEO/WhatsApp)
  → landing page (statically generated, es-PY)
  → multi-step form  [honeypot + required consent checkbox, unticked by default]
  → POST /api/leads  (server-side: validate, normalize phone, idempotency-dedup)
      1. INSERT lead + consent row (own MySQL — system of record)
      2. push to VenderCRM /api/v1/leads (try/catch, 10s timeout, never blocks visitor)
      3. queue delivery matching (phase 3+; manual via admin before that)
  → gracias page (+ conversion pixel)
```

Delivery (phase 3): a matcher selects active buyers whose `filters` accept the
lead and whose DPA is signed, creates `deliveries`, and sends via the buyer's
configured channel. Duplicates (same phone within N days) are flagged and
non-billable. Everything visible and overridable in `/admin`.

## Public site structure (SEO)

Media/lead-funnel archetype — content hub feeding one conversion action
(the loan form):

```
/                                  → hero + form CTA + how-it-works + trust
/solicitar                         → the multi-step form (THE conversion page)
/gracias                           → confirmation + what happens next
/prestamos-personales              → product hub page
/prestamos-en-[ciudad]             → Asunción, Ciudad del Este, Encarnación, Luque, San Lorenzo…
/prestamos-para-[segmento]         → asalariados, independientes, jubilados, funcionarios públicos
/guias/[slug]                      → guides (requisitos, cómo comparar tasas, IPS, veraz/Informconf…)
/quienes-somos                     → who we are + the "we are not a lender" disclosure
/politica-de-privacidad            → privacy policy (versioned in repo)
/terminos-y-condiciones            → T&C incl. lead-sharing disclosure
/baja-de-datos                     → self-service data deletion / consent revocation request
```

One primary intent per page. Interest rates are never promised — advertising
credit terms triggers consumer-credit advertising rules (LEGAL doc §advertising);
we advertise the *service* ("compará opciones", "recibí ofertas"), not a rate.

## Security & data protection posture

- Personal data only over TLS; DB access only from the app (Hostinger Remote MySQL allowlist).
- API keys/DB creds in env vars only; `.env.example` committed, `.env` never.
- Admin behind auth + role check on every server action; no public lead reads.
- Lead exports logged in `activity_log` (who exported what, when).
- Retention job: purge/anonymize undelivered leads after a defined window
  (set in LEGAL doc), keep consent + delivery records for the legal retention period.
- Rate limiting + honeypot on the public form.
