# prestamo.com.py

Loan lead-generation platform for Paraguay. We market to people looking for a
personal loan, capture their application with explicit consent, and sell those
leads to licensed lenders (bancos, financieras, casas de crédito, fintechs).
**We are a marketing partner — we never lend, never decide credit, never touch
funds.**

## Status

Planning phase. No application code yet — the repo currently holds the build
plan that future Claude sessions (Opus/Sonnet) execute PR by PR.

## Read in this order

1. **`docs/ARCHITECTURE.md`** — what we're building and why (stack, domain
   model, lead flow, site structure). Read before every PR.
2. **`docs/MASTER_PLAN.md`** — the PR-by-PR roadmap with acceptance criteria
   and which model builds what. The roadmap of record.
3. **`docs/LEGAL-PARAGUAY.md`** — the legal framework we operate under (data
   protection, financial regulation boundary, consumer/advertising rules).
   Mandatory reading before touching the form, consent flow, policies, ad copy,
   or lead delivery.
4. **`docs/OPERATOR-CHECKLIST.md`** — the non-code work the owner does
   (company setup, contracts, lawyer review, accounts).

## Stack (decided)

Next.js 15 (App Router) + TypeScript + Tailwind + Drizzle ORM + MySQL, deployed
on Hostinger Node.js hosting. VenderCRM as the ops mirror for leads and the B2B
sales pipeline. See ARCHITECTURE.md for the reasoning.

## Hard rules

- Data minimization: the lead schema in ARCHITECTURE.md is the ceiling. New
  form fields require a legal-doc update in the same PR.
- Consent is versioned and append-only. No consent row, no lead.
- No interest rates in ads or on pages without a BCP citation and date.
- Site content in Paraguayan Spanish (voseo); code and commits in English.
