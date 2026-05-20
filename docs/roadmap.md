# Roadmap

🌐 [한국어](roadmap.ko.md)

How FlowLinq evolves from an empty repo to a paid-customer product.

This roadmap is opinionated about **what to skip**, not just what to build. Skipped
items are intentional — see [why.md](why.md) for the reasoning.

## Phase 0 — Foundation

**Goal.** Repo is set up to consume `linq-kit` and ready for first feature work.

- [ ] `package.json`, `pnpm-workspace.yaml` (if needed), `tsconfig.json`
- [ ] `.npmrc` with GitHub Packages auth
- [ ] CI: `pnpm install` + `pnpm test` + `pnpm build` on every PR
- [ ] First `@devslab-kr/linq-core` dependency added (proves the auth + install path)
- [ ] Domain `getflowlinq.app` purchased (Cloudflare Registrar)
- [ ] Landing page placeholder (just a "coming soon" + email capture)
- [ ] `filebrowser` vendoring strategy decided (sidecar vs embed)

**Done when.** Empty FlowLinq app starts, imports a `linq-core` symbol, lands on
`getflowlinq.app`.

## Phase 1 — Document intake

**Goal.** Document arrives via WhatsApp, lands in R2, is visible in filebrowser UI.

- [ ] WhatsApp adapter consumed via `linq-channel` (extracted from BookLinq in `linq-kit` Phase 2)
- [ ] R2 bucket + signed upload via `linq-vault`
- [ ] `filebrowser` running (chosen vendoring strategy)
- [ ] WhatsApp → R2 → filebrowser flow working end-to-end
- [ ] `documents` table created (id, businessId, storageRef, ingestChannel, createdAt)
- [ ] Multi-tenant scoping (single test business for now)
- [ ] Basic audit log of "document arrived" events via `linq-ops`

**Done when.** A real WhatsApp photo from a test phone appears in filebrowser within
10 seconds of sending.

## Phase 2 — Classification + extraction

**Goal.** A document is automatically classified and its fields extracted.

- [ ] OCR pipeline (`linq-lens`)
- [ ] LLM classification: invoice / PO / expense / contract / receipt / unknown
- [ ] Field extraction with confidence scoring
- [ ] Indian invoice variants: GST number, HSN code, vendor formats
- [ ] Confidence threshold logic — low confidence → "needs human review"
- [ ] Manual correction UI (in filebrowser overlay) for low-confidence cases
- [ ] Test corpus of 50+ real Indian invoices for accuracy tracking

**Done when.** A test corpus of 50 Indian invoices reaches >90% classification accuracy
and >80% field extraction accuracy.

## Phase 3 — Workflow engine

**Goal.** Documents route to approvers and get approved over WhatsApp.

- [ ] Workflow state machine (`linq-flow`): `PENDING → ROUTED → APPROVED/REJECTED → POSTED`
- [ ] Approval chain configuration (amount × category × department)
- [ ] WhatsApp notification to approver: "Vendor X sent invoice for ₹Y. Approve?"
- [ ] Approver reply parsing ("approve", "승인", "reject — quantity wrong")
- [ ] Audit trail of every transition via `linq-ops`
- [ ] Manager dashboard (web) showing pending approvals
- [ ] Reminder logic for unresponded approvals (24h, 48h)

**Done when.** End-to-end: invoice arrives → classified → routed → approved over
WhatsApp → state shows APPROVED with full audit log.

## Phase 4 — Tally integration

**Goal.** Approved invoices post to a real Tally instance. First paying customer.

- [ ] `linq-bridge` Tally connector — TDL or ODBC, decide based on field testing
- [ ] Idempotency: re-posting an approved invoice does not duplicate in Tally
- [ ] Reconciliation report — what's in FlowLinq vs what's in Tally
- [ ] Self-serve onboarding: SMB connects their Tally via setup wizard
- [ ] First Indian SMB pays for the product
- [ ] Pricing decided (tentatively: $20–60 / month tiered by document volume)

**Done when.** One paying Indian SMB has at least 30 days of approved invoices
posted from FlowLinq to their Tally.

## Phase 5 — Conversational search

**Goal.** Users find documents and source evidence by chatting with LinqAI.

- [ ] LinqAI chat overlay embedded in filebrowser UI
- [ ] Semantic search over documents (embeddings + vector store)
- [ ] Tool gateway: `searchDocuments`, `findEvidence`, `summarizeApprovals`
- [ ] Multi-language UI: English, Hindi, Korean
- [ ] "지난달 vendor X invoice 다 보여줘" / "show me last month's vendor X invoices"
      both work
- [ ] Cite sources — answer points to specific document IDs

**Done when.** Three different SMB users say "I just ask FlowLinq instead of searching
manually now."

## Phase 6 — Scale out

**Goal.** More bridges, more channels, more tenants.

- [ ] Zoho Books connector
- [ ] ERPNext connector
- [ ] Email forward intake (`documents@<tenant>.flowlinq.app`)
- [ ] Voice channel (describe an invoice over the phone)
- [ ] Multi-tenant hardening (tenant isolation, per-tenant rate limits)
- [ ] Self-host bundle (Docker Compose) for SMBs that need on-prem
- [ ] 10+ paying SMBs

**Done when.** Revenue is recurring and the product can absorb a tenant without
manual onboarding.

## Phase 7+ — Adjacent expansion

Only consider after Phase 6:

- HR documents (leave, reimbursement, onboarding)
- Vendor / contract approval workflows
- eSign integration (eMudhra / Sify partnership)
- UPI collection on approved invoices (Razorpay / Cashfree)

Each of these is a separate product decision with its own go-to-market risk. Don't
treat them as "phase 8 features" — treat them as separate ventures.

## What this roadmap intentionally skips

- **General workflow builder.** Loses to Zoho. We only build the workflows our wedge needs.
- **Web-form-first UX.** WhatsApp is the channel. Web is for managers, not uploaders.
- **Mobile app.** WhatsApp IS the mobile app.
- **Multiple LLM providers in V1.** Use whatever BookLinq settles on; revisit later.
- **Custom OCR.** Use what works; switch only when accuracy floor is hit.
- **Enterprise features (SSO, SAML, custom roles).** Not our buyer in V1.

## Cadence

- Phase target: ~6–8 weeks each, but length-of-Phase is not the metric. **Definition of
  Done** for each phase is the metric.
- Move to the next phase only when current phase Done criteria are met. Don't half-finish.
