# Decision Log

🌐 [한국어](decisions.ko.md)

Every non-obvious product / technical choice — what we picked, what we rejected,
and when to revisit. Read this before pushing back on a decision; the alternatives
that were already considered live here.

Each entry has the same shape:

- **Status** — Decided / Deferred / Reconsider when X
- **Date** — when the decision (or last revision) was made
- **Context** — what question we were answering
- **Options considered** — every alternative on the table
- **Decision** — what we picked
- **Rationale** — why
- **Trade-offs accepted** — costs we knowingly took on
- **Revisit when** — the trigger to reopen the question

Cross-cutting kit-level decisions live in
[`devslab-kr/linq-kit/docs/decisions.md`](https://github.com/devslab-kr/linq-kit/blob/main/docs/decisions.md).

---

## D-001 — Market direction: AI Document Operations Platform

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Where to take the Linq stack next, beyond BookLinq's booking SaaS.

**Options considered.**

| Option                                       | Verdict   | Why                                                                                                |
| -------------------------------------------- | --------- | -------------------------------------------------------------------------------------------------- |
| eSign / digital signature SaaS               | Rejected  | Crowded (Digio, Leegality, SignDesk, Zoho Sign). Requires licensed CA partnership in India.        |
| UPI / payment collection SaaS                | Rejected  | Crowded (Razorpay, Cashfree, PhonePe). Requires PSP partnership + RBI compliance.                  |
| AI Document Operations Platform (this)       | **Chosen**| EDMS + workflow space less crowded for SMB. No regulated moat blocking entry. BookLinq IP transfers.|
| Horizontal "workflow builder"                | Rejected  | Direct competition with Zoho Workflow / Kissflow at our weakest. No defensible wedge.              |

- **Rationale.** Channel-native AI (WhatsApp + multilingual agent) is BookLinq's strongest
  IP. Doc operations is the adjacent market where that IP wins and where compliance is
  not a moat against entrants.
- **Trade-offs accepted.** Smaller per-customer ACV than payments. Slower revenue ramp.
  Classification accuracy is the make-or-break.
- **Revisit when.** Phase 4 customer feedback says "we'd actually pay for eSign on top
  of approved docs" — then expansion to eSign becomes a Phase 7+ topic, not a pivot.

---

## D-002 — Vertical wedge: invoice / expense approval for Tally users

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Document workflow is too broad to ship. Which vertical wedge first?

**Options considered.**

| Wedge                              | Verdict   | Why                                                                                  |
| ---------------------------------- | --------- | ------------------------------------------------------------------------------------ |
| Invoice / expense approval         | **Chosen**| Largest TAM in Indian SMB. Tally integration is unloved engineering = the moat.       |
| HR documents (leave / reimbursement)| Rejected  | Smaller market, lower urgency, less compelling AI angle.                              |
| Vendor / contract approval         | Rejected  | Higher ACV but long enterprise sales cycle. Wrong shape for PLG self-onboarding.       |
| Horizontal workflow builder        | Rejected  | (See D-001) Zoho wins.                                                                |

- **Rationale.** Invoice approval is high-frequency, has clear ROI ("save 5 hours / week"),
  and binds to Tally — Tally is the system of record for Indian SMB accounting and the
  partnership channel for distribution.
- **Trade-offs accepted.** Tally integration is legacy tech (TDL, ODBC). Field-level
  Indian invoice variants will take 6+ months to nail. We dedicate that time deliberately.
- **Revisit when.** After 10 paying SMBs in invoice flow — then start adjacent
  workflows (expense → vendor → contract) as Phase 7+.

---

## D-003 — EDMS base: `filebrowser` vendored as library

- **Status.** Decided (with Phase 1 prototype to validate)
- **Date.** 2026-05-21
- **Context.** We need a document storage + browse UI layer. Build, or vendor an OSS base?

**Options considered.**

| Option                         | Verdict      | Key signal                                                                              |
| ------------------------------ | ------------ | --------------------------------------------------------------------------------------- |
| `filebrowser` (gtsteffaniak fork)| **Chosen**  | Single Go binary, file/folder UI + preview + sharing free, active fork.                  |
| `Paperless-ngx`                | Strong runner-up | Already a doc-management product with OCR / tag / correspondent. 70K+ GitHub stars. Heavier (Python + Django). Best alternative if filebrowser falls short. |
| `Mayan EDMS`                   | Rejected     | Closest to "EDMS with workflow built-in" but Python + complex deployment. Overkill for V1. |
| `Docspell`                     | Rejected     | JVM stack adds operational weight we don't want.                                         |
| `Nextcloud`                    | Rejected     | Too heavy, PHP, enterprise framing wrong for SMB.                                        |
| `Seafile` / `OwnCloud`         | Rejected     | Sync-focused, not workflow-friendly.                                                     |
| Build from scratch             | Rejected     | 3–6 months of frontend work that's already commoditized.                                 |

- **Rationale.** filebrowser is the lightest base that delivers a usable file UI
  immediately. We do not need its data model to be our data model — workflow state lives
  alongside it in our own Postgres tables.
- **Trade-offs accepted.** filebrowser's mental model is "files in folders," not
  "documents as entities." We bridge that with our own document object model (see
  [architecture.md](architecture.md)). gtsteffaniak fork is maintained by one person —
  small bus-factor risk on CVE response time.
- **Revisit when.** Phase 1 prototype hits real integration friction (auth, embedding,
  iframe seams) — fall back to Paperless-ngx as the runner-up.

---

## D-004 — `filebrowser` integration: vendor, never fork

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Given we use filebrowser, how do we include it in our stack?

**Options considered.**

| Approach                              | Verdict           | Rationale                                                                |
| ------------------------------------- | ----------------- | ------------------------------------------------------------------------ |
| Fork (own copy in `devslab-kr/`)      | Rejected          | Inherit every CVE, every upstream rebase, every customization conflict.  |
| Vendor as library / binary (no fork)  | **Chosen**        | Treat as a dependency. Customization lives in our wrapper, not their code.|
| Run as sidecar binary (process)       | **Default for Phase 0** | Cleanest separation, lowest coupling. UI via iframe + signed cookie. |
| Embed as Go library                   | Backup option     | More integrated UI but pulls Go service into our Cloudflare-Workers stack.|
| Reuse only frontend build             | Rejected          | Most work, most control, but we don't need that level of customization.   |

- **Rationale.** Forking is the most common way OSS-on-OSS startups fail. Vendoring as
  a process boundary keeps upstream updates trivial.
- **Trade-offs accepted.** Slight UX seam at the iframe boundary. Two processes to deploy.
- **Revisit when.** Phase 1 prototype shows the iframe seam blocks the LinqAI chat
  overlay. Fall back to "embed as Go library."

---

## D-005 — Storage: Cloudflare R2

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Where do file bytes live?

**Options considered.**

| Option                          | Verdict   | Why                                                                          |
| ------------------------------- | --------- | ---------------------------------------------------------------------------- |
| Cloudflare R2                   | **Chosen**| No egress fees, S3-compatible API, edges in India, BookLinq is already on Cloudflare.|
| AWS S3                          | Rejected  | Egress cost adds up at SMB volumes. No platform consolidation upside.         |
| Self-host (filebrowser local FS)| Backlog   | Needed for on-prem option later. Not V1.                                      |
| Backblaze B2                    | Rejected  | Cheaper raw storage but less integrated with Workers.                         |

- **Trade-offs accepted.** Cloudflare lock-in. Mitigated by `linq-vault` abstraction —
  swapping backends is a contained refactor.
- **Revisit when.** On-prem deployment becomes a sales requirement.

---

## D-006 — Workflow state DB: Postgres (Neon), not D1

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Where does workflow state, audit log, document metadata live?

**Options considered.**

| Option            | Verdict   | Why                                                                       |
| ----------------- | --------- | ------------------------------------------------------------------------- |
| Postgres (Neon)   | **Chosen**| Audit + workflow joins are non-trivial. Postgres is the safe default.      |
| D1 (SQLite on Workers)| Rejected | Cheaper but limited for the join-heavy queries we'll do.                  |
| Hyperdrive + own Postgres| Backlog | Option if we hit Neon limits.                                            |

- **Trade-offs accepted.** Slightly higher monthly cost than D1. Cold-start latency on
  Neon free tier (mitigated by Neon's auto-resume).
- **Revisit when.** Per-tenant data volumes exceed Neon's comfortable ceiling.

---

## D-007 — OCR provider: deferred, with shortlist

- **Status.** Deferred (decide in Phase 5)
- **Date.** 2026-05-21
- **Context.** Which OCR engine produces structured fields from Indian invoice images?

**Shortlist for Phase 5 evaluation.**

| Engine                  | Cost                | Quality        | Notes                                              |
| ----------------------- | ------------------- | -------------- | -------------------------------------------------- |
| Tesseract (local)       | Free                | Mediocre on scans | Default for cheap path. Run inside Worker or sidecar.|
| Google Document AI      | Paid per page       | Excellent      | Strong on invoice layouts including Indian variants. |
| AWS Textract            | Paid per page       | Excellent      | Comparable to Google.                              |
| Mistral OCR             | Paid                | Newer, promising | Worth a Phase 5 bake-off.                        |
| Multimodal LLM only (no OCR)| Per-token cost  | Variable       | Tempting for V1; quality drops on noisy scans.     |

- **Rationale for deferral.** We can't pick without a test corpus of real Indian
  invoices. The accuracy floor is the criterion, and we won't have data until Phase 2.
- **Default during Phase 2–4.** Multimodal LLM only (Workers AI vision model) — cheapest
  to ship, retire if accuracy plateaus below 90%.

---

## D-008 — LLM model: inherit BookLinq's choice

- **Status.** Decided (with same override-via-env policy as BookLinq)
- **Date.** 2026-05-21
- **Context.** Which LLM powers the FlowLinq agent runtime?

**Inherits from BookLinq** (see `apps/api/src/whatsapp/ai/workers-ai.ts::DEFAULT_MODEL`):

| Model                                 | Status                                                  |
| ------------------------------------- | ------------------------------------------------------- |
| `@cf/zai-org/glm-4.7-flash`           | **Default.** Multilingual + tool calling. Fits well.    |
| `@cf/openai/gpt-oss-120b`             | Validated fallback. ~3x cost, top stability.            |
| `@cf/meta/llama-3.3-70b-instruct-fp8-fast` | Failed — Korean accuracy.                          |
| `@cf/meta/llama-4-scout-17b-16e-instruct` | Failed — Korean accuracy.                           |

- **Rationale.** No reason to diverge from BookLinq's already-validated stack.
  `linq-agent` will expose the same `WHATSAPP_AI_MODEL` env override.
- **Trade-offs accepted.** Locked to Cloudflare Workers AI. Mitigated by
  `linq-agent`'s provider abstraction — Anthropic / OpenAI direct are swap candidates if needed.
- **Revisit when.** A new model in Workers AI catalogue meaningfully outperforms
  glm-4.7-flash on Hindi or Indian invoice classification.

---

## D-009 — Geography: India first

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Which market does V1 target?

**Options considered.**

| Region                | Verdict   | Why                                                                            |
| --------------------- | --------- | ------------------------------------------------------------------------------ |
| India (SMB)           | **Chosen**| Largest under-served SMB market in workflow / EDMS. Tally is the integration moat.|
| Korea / Japan         | Rejected  | Smaller TAM, more entrenched local SaaS, less WhatsApp prevalence.              |
| US / Western SMB      | Rejected  | Saturated with Box / Docusign / Bill.com. Wrong economics for our cost structure.|
| SEA (Vietnam / Indonesia / Philippines)| Backlog | Similar profile to India, revisit Phase 6+.                          |

- **Trade-offs accepted.** Indian SMB ACV ($6–60 / month) requires high-volume, PLG
  motion. DPDP Act 2023 data localization may demand India edge presence (Cloudflare
  India edge handles this).
- **Revisit when.** First 10 paying Indian SMBs are in production. Then SEA expansion.

---

## D-010 — Brand name: FlowLinq

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Product name within the `[purpose]Linq` family.

**Top candidates considered.**

| Name        | Signal                            | Why not                                                                    |
| ----------- | --------------------------------- | -------------------------------------------------------------------------- |
| **FlowLinq**| Document flow, workflow           | **Chosen.** Matches BookLinq cadence, captures workflow nature, India-pronounceable.|
| VaultLinq   | Storage / archive / security      | 2nd choice. Stronger for "self-host EDMS" framing if we pivot that way.    |
| DocuLinq    | Document-centric, direct          | Generic, 3 syllables, less distinct.                                       |
| TrailLinq   | Audit trail emphasis              | Differentiating but doesn't communicate product category at first read.    |
| OpsLinq     | Operations, original brand sketch | "Ops" reads as tech jargon to SMB owners.                                  |
| PaperLinq   | Paperless office                  | Too narrow — limits expansion beyond paper docs.                            |
| FolioLinq   | Document portfolio                | Sophisticated but obscure.                                                  |

- **Rationale.** FlowLinq elevator pitch — "FlowLinq manages where your company's
  documents flow." Same syllable rhythm as BookLinq makes the family obvious.
- **Trade-offs accepted.** "Flow" is over-used in B2B SaaS branding; the `Linq` suffix
  carries the differentiation.
- **Revisit when.** Trademark search (USPTO / IP India / WIPO Class 9, 42) hits a
  conflict — VaultLinq is the fallback.

---

## D-011 — Domain: `getflowlinq.app` (canonical), defer `.com`

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Primary domain to ship on.

**Findings.**

- `flowlinq.com` is held by a parker. Acquisition price unknown; defer until traction.
- `flowlink.com` (similar spelling) is held by unrelated companies (Dutch wholesaler,
  eCommerce integration) — not direct competitors.
- "flowlinq" SERP is empty — easy to dominate.

**Plan.**

| Domain                  | Action                                                                            |
| ----------------------- | --------------------------------------------------------------------------------- |
| `getflowlinq.app`       | **Buy now.** Canonical. `.app` enforces HTTPS via Google HSTS preload.            |
| `flowlinq.in`           | Buy when Indian entity / cofounder exists (requires Indian registrant). Trust signal.|
| `flowlinq.io`           | Defensive, ~$30. Optional.                                                        |
| `flowlinq.co`           | Defensive, ~$30. Optional.                                                        |
| `flowlinq.com`          | Defer. Renegotiate after traction; parker pricing inflates with our success.       |

- **Trade-offs accepted.** Users will mistype `flowlink`. Google autocorrect will push
  them to it for 12+ months. Brand education is the cost.
- **Revisit when.** Phase 4 (first paying customer) — then weigh `.com` acquisition.

---

## D-012 — Repo structure: shared kit + product repo (Option B)

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** How to organize code shared between BookLinq, FlowLinq, and future products.

**Options considered.**

| Option                                  | Verdict   | Why                                                                          |
| --------------------------------------- | --------- | ---------------------------------------------------------------------------- |
| A — Multi-repo full (each shared pkg = own repo)| Rejected  | Publish ceremony × N. Coordination overhead too high for small team.       |
| B — `linq-kit` shared monorepo + product repos| **Chosen**| Shared code is atomic-refactor friendly. Products keep independent deploy / secrets / access.|
| C — Single monorepo with everything     | Rejected  | Migration cost for BookLinq + deploy / secret separation harder. Sells less well per product.|

- **Rationale.** With multiple products planned and shared safety / agent layer
  crystallizing, B gives the best long-run flexibility. C tempted us — vetoed for
  the reasons above.
- **Trade-offs accepted.** Publish cycle (kit version bump → product PR) is real
  friction. Mitigated by Changesets + Renovate auto-PR.
- **Revisit when.** Headcount > 5 — then evaluate splitting kit packages into their
  own repos for parallel team ownership.

(See kit-side rationale: `linq-kit/docs/decisions.md` D-101.)

---

## D-013 — Product features explicitly OUT of scope for V1

- **Status.** Decided
- **Date.** 2026-05-21

| Feature                                | Reason for exclusion                                                       |
| -------------------------------------- | -------------------------------------------------------------------------- |
| eSign / digital signature              | Regulated. Requires CA partnership. (D-001)                                |
| UPI / payment collection               | Regulated. Requires PSP partnership.                                        |
| General workflow builder               | Loses to Zoho. (D-002)                                                      |
| Mobile app                             | WhatsApp IS the mobile app.                                                 |
| Web-form-first UX for uploaders        | Wrong channel for Indian SMB workforce.                                     |
| Enterprise SSO / SAML / custom roles   | Not our buyer in V1.                                                        |
| Multi-LLM provider switching in V1     | Inherit BookLinq's stack until there's reason to diverge. (D-008)           |
| Custom-built OCR pipeline              | Use existing engines. Switch only when accuracy floor demands. (D-007)      |

**Revisit when.** Phase 6+ — adjacent expansion considered as separate ventures, not
feature flags.

---

## How to add a decision here

1. Pick the next free `D-NNN` number.
2. Use the section template at the top of this file.
3. Always list **options considered** even if obvious — future readers won't have your context.
4. State **revisit when** explicitly. A decision with no revisit trigger is a religion.
5. Link to relevant code, PRs, or external pages in the rationale.
