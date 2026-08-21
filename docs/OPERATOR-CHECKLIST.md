# Operator checklist — the non-code work

Everything the owner does outside the repo. Ordered; items marked **[BLOCKS
LAUNCH]** must be done before paid traffic runs. Legal background for each item
is in `docs/LEGAL-PARAGUAY.md`.

## A. Company & tax

- [ ] **[BLOCKS LAUNCH]** Incorporate an **EAS** via SUACE (Ley 6480/2020) —
      resolves RUC at the same time. As a foreigner, confirm first with a local
      gestor whether you need residency/cédula or will act via apostilled POA
      or a foreign entity.
- [ ] Open a local bank account for the EAS (buyers pay in ₲ by transferencia).
- [ ] Contract an accountant (contador) — monthly IVA declarations, IRE.
- [ ] Get timbrado from DNIT; ask the accountant whether the EAS falls under a
      SIFEN e-invoicing phase now, and if so pick a certified e-invoicing
      provider (lenders will expect electronic facturas).

## B. Legal protection (lawyer engagement — one package)

Engage a Paraguayan lawyer (data-protection + consumer law) once, with this
scope:

- [ ] **[BLOCKS LAUNCH]** Written opinion confirming the **lead-broker
      position** (no BCP license; boundaries to respect) — LEGAL doc §1.
- [ ] **[BLOCKS LAUNCH]** Review the site's privacy policy, T&C, and the
      **consent checkbox wording** (Ley 6534 now, Ley 7593 readiness).
- [ ] Draft two reusable templates: **lead-purchase agreement** and
      **data-transfer/processing agreement (DPA)** for buyers.
- [ ] Confirm retention windows (proposed: 12 months undelivered leads,
      5 years consent/delivery records).
- [ ] Calendar item: **Ley 7593 full applicability ~Nov 2027** — re-audit
      consent flow, ANPDP registration duties, DPO need, cross-border transfer
      safeguards (hosting/CRM location) once ANPDP regulation exists.

## C. Domain, hosting, accounts

- [ ] Confirm ownership/renewal of **prestamo.com.py** (NIC.py); consider also
      defensive variants if cheap (prestamo**s** plural is a competitor —
      don't buy, don't imitate).
- [ ] Choose the Hostinger account/Node.js slot (record it in MASTER_PLAN
      PR-6); create the MySQL DB.
- [ ] VenderCRM: create the site under **Sitios**, get the API key (goes in
      server env only); set the default pipeline/stage for prestamo leads.
- [ ] Corporate email on the domain (contacto@, datos@ for ARCO requests).
- [ ] Google Analytics 4 + Google Ads + Meta Business accounts under the EAS.
      Note: Google/Meta both have restricted-vertical policies for **personal
      loans** — expect advertiser verification, no "guaranteed approval"
      language, and destination requirements (visible APR/CTC disclosures if
      rates are ever shown). Budget time for ad-account review.

## D. Buyer development (the actual business)

- [ ] Build the prospect list from LEGAL doc §7 (start with casas de crédito
      groups A/B and digital-first lenders: Ueno, Banco Familiar/Credicédula,
      Finexpar, Tu Financiera).
- [ ] Track prospects as a **B2B pipeline in VenderCRM** (separate from
      consumer leads).
- [ ] Per closed buyer: verify license/registration → sign lead-purchase
      agreement + DPA → only then activate in `/admin/buyers`.
- [ ] Agree per buyer: price per lead (₲), exclusive vs shared, filters,
      returns window, payment terms (monthly invoice).
- [ ] First 2–4 weeks: deliver manually (WhatsApp/email from admin) to prove
      quality before automating (Phase 3).

## E. Ongoing operations

- [ ] Answer **data-subject requests** within the SLA in the admin queue
      (weekly check minimum).
- [ ] Monthly: skim BCP's published usury limits if any content cites rates;
      refresh dated figures in guides.
- [ ] Keep the buyer list in the privacy policy current (it names lender
      categories/recipients).
- [ ] Breach response: if lead data leaks, call the lawyer same-day; document
      timeline; buyer notification per DPA; ANPDP notification duties apply
      once 7593 regulation is live.
- [ ] Never launch a new form field, rate display, or marketing channel
      without the LEGAL doc change-control rule (it's enforced in MASTER_PLAN
      ground rules too).
