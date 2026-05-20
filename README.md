# FlowLinq

🌐 [한국어](README.ko.md)

**AI-powered Document Operations Platform — WhatsApp-native EDMS for Indian SMBs.**

FlowLinq takes the chaos of invoices, expense receipts, POs, and contracts arriving via
WhatsApp, email, and photo, and turns it into a structured, auditable approval flow
that posts the final result to existing accounting systems (Tally, Zoho Books, ERPNext).

## Why FlowLinq exists

Indian SMBs run document operations on WhatsApp + email + Excel + Tally. Approval chains
are verbal, audit trails don't exist, and reconciliation is manual. Existing EDMS
(DocuWare, M-Files, Kissflow) are web-form-first — the wrong channel for the Indian
SMB workforce.

FlowLinq is **WhatsApp-first**, **AI-native**, and **integrates with the tools SMBs
already use**.

Full positioning rationale: [docs/why.md](docs/why.md).

## Product flow

```text
[ Customer Channel ]
    WhatsApp / Web / Email / Voice / Instagram DM / KakaoTalk
        │
        ▼
[ Agent Runtime ]
    intent · tool selection · conversation memory · policy enforcement
        │
        ▼
[ Tool Gateway ]
    ingestDocument · classifyDocument · extractFields · routeDocument
    requestApproval · recordDecision · postToBridge · searchDocuments
        │
        ▼
[ Existing System ]
    Tally · Zoho Books · ERPNext · CRM · ERP
```

User-facing workflow:

1. **Intake** — document arrives on any channel
2. **Classify** — invoice / PO / expense / contract / receipt
3. **Extract** — vendor, amount, GST, dates, line items
4. **Route** — assigned by policy (amount × category × department)
5. **Approve** — manager approves over WhatsApp
6. **Decide** — approval / rejection recorded with reason
7. **Post** — pushed to Tally / Zoho Books / ERPNext
8. **Search** — chat with LinqAI to find documents and source evidence

Full architecture: [docs/architecture.md](docs/architecture.md).

## Family

FlowLinq is part of the **Linq** product family at devslab-kr:

| Product      | Purpose                              | Repo                                                |
| ------------ | ------------------------------------ | --------------------------------------------------- |
| **BookLinq** | WhatsApp-first booking SaaS          | [jlc488/booklinq](https://github.com/jlc488/booklinq) |
| **FlowLinq** | AI Document Operations Platform      | this repo                                           |

Both products share the platform monorepo:

→ [`devslab-kr/linq-kit`](https://github.com/devslab-kr/linq-kit) — agent runtime, channel adapters, workflow engine, OCR/extraction, external system bridges.

## Tech stack

- **TypeScript 6**
- **Cloudflare Workers** + **R2** for storage, **D1 / Postgres** for workflow state
- **`filebrowser`** (gtsteffaniak fork) — vendored as a library for the document UI layer
- **LinqAI** chat overlay for conversational search
- **pnpm**, **Vite 8**, **Vitest**
- Domain: **`getflowlinq.app`** (planned)

## Architecture decisions (locked)

- **`filebrowser` is vendored as a library, not forked.** It provides the file-browsing
  UI surface; everything workflow-related lives in our own document model alongside it.
  See [docs/architecture.md](docs/architecture.md).
- **Single source of agent + channel logic**: `@devslab-kr/linq-*` packages.
  FlowLinq does not duplicate agent runtime code.
- **Indian market first.** Tally integration > Zoho Books > ERPNext, in that order.
- **No eSign / no UPI** in V1. Document workflow only — leave regulated layers for later
  once the channel + workflow value is proven.

## Roadmap

Full plan: [docs/roadmap.md](docs/roadmap.md). Phases at a glance:

- [ ] **Phase 0** — Foundation: repo bootstrap, `linq-kit` consumption, `filebrowser` vendoring decision
- [ ] **Phase 1** — Document intake from WhatsApp → R2 + filebrowser UI
- [ ] **Phase 2** — OCR + classification (`linq-lens`) — invoice / PO / expense / contract / receipt
- [ ] **Phase 3** — Workflow engine (`linq-flow`) — routing, approval, audit trail
- [ ] **Phase 4** — Tally integration (`linq-bridge`) — first paying Indian SMB
- [ ] **Phase 5** — Conversational search — "show me last month's invoices from vendor X"
- [ ] **Phase 6** — Zoho Books + ERPNext bridges, multi-tenant scale

## Status

**Not yet bootstrapped.** This repo currently holds only documentation. Code lands once
`linq-kit` Phase 0 publishes its first package.

## License

Proprietary — devslab-kr internal use only.
