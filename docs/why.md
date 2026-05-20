# Why FlowLinq

🌐 [한국어](why.ko.md)

Market positioning, competitive landscape, and the wedge.

## The market

Indian SMBs (5–200 employees) run their document operations on a chaotic mix of:

- **WhatsApp** for everything — vendor sends an invoice photo, manager forwards it
- **Email** with PDF attachments nobody opens consistently
- **Excel** for tracking approvals (mostly missing)
- **Tally** as the system of record for accounting
- **Phone calls** for approvals ("haan, approve kar do")

The pain isn't lack of an EDMS — it's that existing EDMS demand a different way of
working (web forms, dedicated software) that the Indian SMB workforce will not adopt.

## Why now

Three timing signals converged:

1. **WhatsApp Business API maturity** — Meta Cloud API made WhatsApp programmable at
   SMB-affordable pricing in the last 18 months.
2. **LLM tool-calling reliability** — open-weight multilingual models now classify Indian
   invoices and parse Hindi/English messages well enough to ship.
3. **Indian SMB digitization push** — GST compliance, UPI ubiquity, and the post-COVID
   shift have made Indian SMBs comfortable with cloud tools for the first time.

## Competitive landscape

| Player                | Channel        | India SMB presence | AI-native | Weak spot                          |
| --------------------- | -------------- | ------------------ | --------- | ---------------------------------- |
| **Zoho Workflow**     | Web/email      | Very high          | No        | WhatsApp + AI both weak            |
| **Kissflow**          | Web            | Medium             | Limited   | Enterprise-focused, mid-market+    |
| **DocuWare / M-Files** | Web            | Low                | Limited   | Too heavy and expensive for SMB    |
| **Frappe / ERPNext**  | Web            | High (developers)  | No        | Needs an integrator to deploy      |
| **Paperless-ngx**     | Self-host web  | Niche              | No        | No workflow, no India focus        |
| **FlowLinq**          | WhatsApp-first | (target)           | Yes       | Brand new, no traction yet         |

The market is large and not consolidated at the SMB layer. The defensible wedge is
**channel + AI** combined — Zoho is too big to pivot to WhatsApp-first quickly, and
the AI-native entrants are mostly going after Western mid-market.

## The wedge

> **WhatsApp-native invoice / expense approval for Indian SMBs that already use Tally.**

Concretely:

- Vendor sends an invoice photo over WhatsApp to the business's number.
- FlowLinq classifies it, extracts vendor / amount / GST, routes to the approver.
- Approver gets a WhatsApp message: "Vendor ABC sent invoice for ₹45,000. Approve?"
- Approver replies "approve" or "reject — quantity wrong".
- FlowLinq posts the approved invoice to Tally with full audit trail.

This is a single workflow, not a platform. We **earn** the right to be a platform by
nailing this one workflow first.

## What we will not build (V1)

- **eSign / digital signature** — regulated layer requiring CA partnerships
  (eMudhra, Sify, NSDL). Re-enter once the workflow business has traction and capital.
- **Payment / UPI collection** — requires PSP partnership (Razorpay, Cashfree) and
  RBI compliance. Same logic: not in V1.
- **General-purpose workflow builder** — head-to-head with Zoho. Lose.
- **HR / leave / reimbursement** — adjacent vertical, deferred until invoice flow has
  paying customers.

## Distribution

- **PLG via WhatsApp self-onboarding.** No demo call required to start. Sign up via
  WhatsApp, point to your Tally instance, ingest first invoice in 10 minutes.
- **Tally ecosystem partnerships.** Tally has a partner program for connected apps.
  Long courtship but the right channel.
- **Bottom-up bookkeeper channel.** Indian bookkeepers serve 10–50 SMBs each — converting
  one bookkeeper gets you 10+ SMBs.

## Risks we accept

- **Classification accuracy is the make-or-break.** 80% → 95% is a long road; sub-95%
  loses approver trust quickly. We dedicate the first 6 months of build time to this.
- **WhatsApp policy changes from Meta.** Mitigated by `linq-channel` abstraction — we
  can add other channels (web, voice) as escape hatches.
- **Tally integration is unloved engineering.** TDL is legacy, ODBC is legacy. There is
  no glamorous version of this work. It is the moat precisely because it is unloved.

## Risks we don't accept

- **Building a US-style SaaS for India.** Indian SMB has different price expectations
  ($6–60 / month ACV), different trust signals, different language mix. We design for
  India first.
- **Trying to do eSign + payment + workflow simultaneously.** Three regulated entries.
  We die. Pick one, prove it, then expand.
