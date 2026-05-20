# Architecture

🌐 [한국어](architecture.ko.md)

How FlowLinq is built. Decisions captured here are intentionally durable — change them
only with a written rationale.

## Layered model

```text
┌─────────────────────────────────────────────────────────────────┐
│                     Customer Channel                             │
│  WhatsApp · Web Chat · Email · Voice · Instagram DM · KakaoTalk  │
└─────────────────────────────────────────────────────────────────┘
                                │
                ── linq-channel adapter normalization ──
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Agent Runtime                              │
│   intent · tool selection · memory · policy (linq-agent)         │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Tool Gateway                               │
│   ingestDocument · classifyDocument · extractFields              │
│   routeDocument · requestApproval · recordDecision               │
│   postToBridge · searchDocuments · listMyTasks                   │
└─────────────────────────────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
┌──────────────┐       ┌────────────────┐      ┌────────────────┐
│  linq-vault  │       │   linq-lens    │      │   linq-flow    │
│   storage    │       │  OCR + extract │      │   workflow     │
│  + filebrowser│      │                │      │  state machine │
└──────────────┘       └────────────────┘      └────────────────┘
                                                        │
                                                        ▼
                                              ┌────────────────┐
                                              │  linq-bridge   │
                                              │  Tally · Zoho  │
                                              │  ERPNext · CRM │
                                              └────────────────┘

   Cross-cutting:
   ┌──────────────┐     ┌──────────────┐
   │  linq-core   │     │   linq-ops   │
   │ types · util │     │ audit · pol  │
   └──────────────┘     └──────────────┘
```

Every horizontal layer is a `linq-kit` package — FlowLinq composes them and adds
product-specific tools.

## `filebrowser` vendoring strategy

**Decision: vendor as a library / embedded binary, never fork.**

Why:

- We need its file UI, preview, upload, share, permissions — months of frontend work
  for free.
- We do **not** need to extend its internals — workflow state lives in our own document
  model alongside it, not inside filebrowser's data store.
- A fork means owning every CVE response, every upstream rebase, every customization
  conflict. We don't have the headcount.

Tactical options (decide in Phase 0):

1. **Run the binary as a sidecar.** filebrowser runs as its own process, FlowLinq embeds
   the UI via iframe + signed-cookie auth. Cleanest separation. Slight UX seam.
2. **Embed as a Go library.** filebrowser is Go; we wrap it as a library and host inside
   our own Go service. More integrated UI, but requires a Go service in the stack.
3. **Reuse only its front-end build, write our own backend.** Most work, most control.
   Likely unnecessary.

Default plan: **option 1 (sidecar)** unless integration friction blocks the chat
overlay. Revisit after Phase 1.

What lives where:

| Concern                        | Location                                |
| ------------------------------ | --------------------------------------- |
| File bytes (PDF, image)        | R2 / S3 (via `linq-vault`)              |
| File browsing UI               | filebrowser (sidecar)                   |
| Folder / permission UI         | filebrowser                             |
| Document metadata              | Our DB (`documents` table, via `linq-vault`) |
| Workflow state                 | Our DB (`workflow_instances`, via `linq-flow`) |
| Extracted fields               | Our DB (`document_fields`, via `linq-lens`) |
| Audit trail                    | Our DB (via `linq-ops`)                 |
| LinqAI chat                    | FlowLinq web app (overlay on filebrowser UI) |

## Document object model

A FlowLinq document is **not** just a file. It is an entity with:

```ts
type Document = {
  id: DocumentId
  businessId: BusinessId
  uploadedBy: UserId | ExternalSenderId
  ingestChannel: 'whatsapp' | 'web' | 'email' | 'voice' | 'instagram' | 'kakao'

  // Storage
  storageRef: StorageRef         // R2 key + filebrowser path
  mimeType: string
  sizeBytes: number

  // Classification (from linq-lens)
  classification: 'invoice' | 'po' | 'expense' | 'contract' | 'receipt' | 'unknown'
  classificationConfidence: number

  // Extracted fields (varies by classification)
  extractedFields: Record<string, ExtractedField>

  // Workflow (from linq-flow)
  workflowInstanceId: WorkflowInstanceId | null
  state: 'PENDING' | 'ROUTED' | 'APPROVED' | 'REJECTED' | 'POSTED' | 'ARCHIVED'

  // Audit
  createdAt: ISODateString
  updatedAt: ISODateString
}
```

The filesystem hierarchy in filebrowser is a *view* over this — folders are derived
from `businessId × classification × state × month`, not the source of truth.

## Channels

FlowLinq accepts documents from every customer channel. Phase priority:

1. **WhatsApp** — primary intake. Photo of an invoice → `ingestDocument` tool.
2. **Web upload** — for the manager UI side.
3. **Email forward** — `documents@<tenant>.flowlinq.app` style address (later).
4. **Voice** — describe an invoice over the phone (later, novelty value).
5. **Instagram DM / KakaoTalk** — niche, last priority.

All channels normalize through `linq-channel` into a common `IncomingMessage` with
attachments.

## AI / agent stack

- **Agent runtime**: `linq-agent` (same runtime BookLinq uses).
- **Model**: same as BookLinq's `WHATSAPP_AI_MODEL` setting (`@cf/zai-org/glm-4.7-flash`
  default). Override via env. Multilingual + tool-calling required.
- **Tool gateway**: FlowLinq-specific tools listed above. Each tool has:
  - Pre-call policy check (via `linq-ops`)
  - Post-call audit log (via `linq-ops`)
  - Idempotency key on side-effecting tools
- **Memory**: short-term within conversation, long-term per user (preferences,
  approval history).

## Multi-tenant model

- One **business** (tenant) per customer (an Indian SMB).
- Inside a business: **users** (employees) with roles (uploader / approver / admin).
- Documents, workflows, and audit logs are scoped to a business.
- Row-level isolation via `businessId` foreign key on every table.

## Storage

- File bytes: **Cloudflare R2** (cheap egress, India edge points).
- Structured data: **Postgres** (Neon for cloud, optional self-host Postgres for
  on-prem customers).
- Cache: Cloudflare KV for hot lookups.

## Deployment topology

- **Cloud-hosted SaaS** is the primary distribution. FlowLinq runs on Cloudflare
  Workers + R2 + a managed Postgres.
- **Self-host option** for SMBs that demand on-prem (single-tenant Docker Compose
  bundle: filebrowser + Postgres + a Worker-equivalent Node service). Backlog.

## Open questions

- filebrowser sidecar vs embedded — final call after Phase 1 prototype.
- Postgres vs D1 for workflow state — D1 is cheaper, Postgres is more capable for
  the kind of joins audit + workflow searches need. Default to Postgres (Neon).
- OCR provider — Tesseract local for cost, cloud (Google Doc AI, AWS Textract) for
  hard cases. Pick after Phase 5 accuracy testing.
