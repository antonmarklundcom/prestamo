# Legal framework — loan lead generation in Paraguay

Compliance basis for prestamo.com.py. Compiled from primary sources (BACN, BCP,
SEDECO, DNIT) + law-firm commentary, August 2026. **This is research, not legal
advice** — items marked ⚖️ must be confirmed by a Paraguayan lawyer before ads
run (see OPERATOR-CHECKLIST.md). Confidence flags are kept on purpose.

**The one-sentence position:** we are a marketing/lead-generation company. We
never lend, never take deposits, never decide credit, never handle loan funds,
and never present ourselves as a financial entity. Every rule below flows from
defending that position.

## 1. Do we need a financial license? (No — with a caveat)

- **Ley 861/96** (bancos y financieras, amended by Ley 5787/2016): licensing
  attaches to *intermediación financiera* — capturing public funds to lend
  them. We do neither. Not applicable.
- **Ley 6104/2018** + BCP Res. 7/Acta 78 (2019): entities lending **own funds**
  habitually must register with BCP as *casas de crédito / otorgantes de
  crédito dinerario* (400+ registered, categorized A/B/C, BCP Res. 9/Acta 72,
  2023). We lend nothing — not applicable, but **our buyers must be** banks,
  financieras, cooperativas (INCOOP) or **registered** casas de crédito.
  Verifying a buyer's registration/license is part of buyer onboarding.
- Pure lead brokerage/marketing is not a licensed activity in Paraguay;
  comparison sites (prestamos.com.py, ikiwi.com.py) operate openly with "no
  somos una entidad financiera" disclaimers. **Confidence LOW-MEDIUM** — this
  is inference from the absence of a regime, and BCP holds a discretionary
  inclusion power (Ley 6104, Art. 19.29 of the BCP charter). ⚖️ Get a local
  legal opinion confirming the lead-broker position.

**Behavioral guardrails that protect the position** (enforced in code + copy):
- Never state or imply we approve loans; the lender decides.
- No handling of money, no signing credit contracts, no fees charged to consumers.
- Site-wide disclosure: "prestamo.com.py es un servicio de marketing; no somos
  banco, financiera ni casa de crédito; no otorgamos créditos."
- Only sell leads to verified licensed/registered lenders.

## 2. Data protection (the core compliance load)

Two coexisting laws:

- **Ley 6534/2020** (datos personales crediticios) — **in force now**. Financial
  /patrimonial data may be processed only from public sources, from the titular,
  or **with the titular's consent**. Rights: access, rectification, suppression,
  opposition, portability. Credit bureaus (BICs) need BCP authorization —
  **we must never enrich or resell credit-history data**, or we drift into the
  BIC regime.
- **Ley 7593/2025** (comprehensive, GDPR-style) — promulgated 27 Nov 2025,
  **~2-year vacatio legis → fully applicable ~Nov 2027** (confidence
  MEDIUM-HIGH on the date ⚖️). Creates the **ANPDP** under MITIC.
  Extraterritorial scope. Consent + GDPR-like bases; disclosure of transfers to
  third parties; processor (encargado) contracts; breach handling; DPO "if
  applicable" (regulation pending); cross-border transfer safeguards (relevant:
  our hosting/CRM may sit outside PY). Fines 20–2,500 jornales mínimos
  (up to 5,000/10,000 for sensitive-data cases). Registration duties: still
  unregulated, confidence LOW — ⚖️ recheck once ANPDP issues regulation.
- Financial data is **not** on 7593's sensitive-data list (that list: health,
  ethnicity, sexual orientation, beliefs, union, biometric/genetic) — but it is
  specially regulated via 6534. We build to the stricter reading anyway.

**What we implement (already reflected in ARCHITECTURE.md):**
1. **Express, logged consent** at submission: checkbox unticked by default,
   versioned text covering (a) processing of the financial data in the form,
   (b) **transfer/sale to licensed lenders** (categories named, current buyer
   list linked in the privacy policy), (c) contact by phone/SMS/WhatsApp/email
   by us and by the receiving lenders. `consents` table stores timestamp, IP,
   UA, text version — append-only.
2. **Data minimization**: no cédula numbers, no exact salaries, no credit
   history. Income as bands, formality as booleans. The schema is the ceiling.
3. **ARCO+ rights**: `/baja-de-datos` self-service + `data_requests` admin
   queue; revocation propagates a notice to buyers who received the lead.
4. **Data-transfer agreements (DPA)** signed with every buyer before any
   delivery — role definition, purpose limitation, consent-evidence handover,
   security, deletion. Delivery engine hard-blocks buyers without `dpa_signed_at`.
5. **Retention**: undelivered leads purged/anonymized after 12 months;
   consent + delivery records kept 5 years (defensive evidence). ⚖️ Confirm
   both windows with counsel — the laws set no bright-line number.
6. **Security**: TLS everywhere, server-side keys, role-checked admin, logged
   exports, breach-response note in OPERATOR-CHECKLIST.

## 3. Consumer protection & credit advertising

- **Ley 1334/98** (Defensa del Consumidor) enforced by **SEDECO** (Ley
  4974/2013). We are not BCP-supervised, so SEDECO is our regulator.
  Publicidad engañosa prohibited (Arts. 35–37); comparative advertising
  allowed if objective and verifiable.
- **Ley 6366/2019 (CTC rule)**: any ad or page that mentions an interest rate
  or installment **must show the Costo Total del Crédito as an annual effective
  percentage with equal prominence** (size, duration, audibility). Site rule:
  **we do not display rates or cuotas at all** in phase 1 (we advertise the
  service: "compará opciones y recibí ofertas"). If a later phase shows lender
  rates (comparison tables), every figure carries CTC + lender name + "sujeto a
  aprobación" + date, and the CTC rendering becomes a shared component so it
  cannot be omitted.
- **Anti-usury, Ley 2339/2003**: usury = effective rate >30% above the BCP-
  published monthly average for the product/currency (also a crime, CP Art.
  193). BCP publishes limits monthly (≈27–30% TEA consumer PYG recently). We
  never publish or promote an offer above the current cap; guide articles
  citing rates must cite BCP + date.

## 4. E-commerce & electronic marketing

- **Ley 4868/2013 + Decreto 1165/2014**: no registration needed to operate
  online, but mandatory site disclosures — razón social, **RUC**, domicile,
  contact, clear pre-contractual info, dispute resolution. Goes in the footer
  + `/quienes-somos` + T&C.
- **Spam (Ley 4868 Arts. 20–23)**: commercial emails identified as advertising
  with a working opt-out.
- **Ley 5830/2017 + Registro "No Molestar" (SEDECO)**: before marketing
  calls/SMS/WhatsApp to mobiles, numbers must be scrubbed against
  nomolestar.sedeco.gov.py — **unless** there is an existing relationship or
  express authorization. Our consent text grants that express authorization for
  us + receiving lenders; still, any cold outreach lists (never planned) would
  require scrubbing. Enforcement is real (Claro fined ~₲17M, upheld 2024).
- WhatsApp: no specific statute; 5830 + consent rules + **Meta's commerce
  policy** (restricts loan-offer messaging — relevant when we add WhatsApp
  Cloud API automation in phase 5).

## 5. Company, tax, invoicing

- Entity: **EAS (Ley 6480/2020, Decreto 3998/2020)** via SUACE (online,
  ~3–15 business days), RUC bundled. ⚖️ Foreign shareholder in practice needs
  PY cédula/residency, or acts via apostilled POA / foreign entity — confirm
  current SUACE practice.
- Tax (Ley 6380/2019): **IVA 10%** on lead-sale services invoiced to lenders;
  **IRE 10%**; IDU 8%/15% on dividends. Invoicing with timbrado
  (`001-001-XXXXXXX`); lenders are large taxpayers and will expect **SIFEN
  electronic facturas** — ⚖️ check DNIT's current e-invoicing obligation for a
  new small EAS; integrate a certified provider rather than direct SIFEN
  certification when required.

## 6. Buyer onboarding checklist (per lender, enforced by admin)

1. Verify license/registration: bank/financiera (BCP Superintendencia list),
   cooperativa (INCOOP), or casa de crédito (BCP registro de otorgantes).
2. Sign lead-purchase agreement (price, exclusivity, returns window, quality
   criteria) — template task in OPERATOR-CHECKLIST.
3. Sign data-transfer/processing agreement → set `dpa_signed_at`.
4. Confirm the buyer's own consent-handling: they contact leads under OUR
   consent; their onward marketing is on their compliance.

## 7. Market context (from research, for the operator)

- Competitors: prestamos.com.py (direct, plural domain), ikiwi.com.py,
  Tu Financiera, inversionparaguay.com.
- Likely buyers: Ueno Bank, Banco Familiar (Credicédula), Banco Interfisa,
  Solar, Basa, Continental, Itaú; financieras Finexpar, FIC, Fielco, Tu
  Financiera; and the 400+ registered casas de crédito — the most realistic
  **first** buyers. Cooperativas lend heavily but buy slowly.
- No Paraguay-specific loan CPA network found — **direct lender agreements are
  the monetization path** (confidence LOW that none exists; recheck Leadgid /
  LatAm finance networks periodically).

## Change control

Any PR that adds a form field, changes consent wording, displays a rate, or
adds a marketing channel must update this file in the same PR and bump the
consent text version where applicable.
