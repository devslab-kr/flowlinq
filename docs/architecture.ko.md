# Architecture

🌐 [English](architecture.md)

FlowLinq이 어떻게 만들어져 있는지. 여기 잡힌 결정은 의도적으로 durable —
쓰여진 이유 없이 바꾸지 않는다.

## Layered model

```text
┌─────────────────────────────────────────────────────────────────┐
│                     Customer Channel                             │
│  WhatsApp · Web Chat · Email · Voice · Instagram DM · KakaoTalk  │
└─────────────────────────────────────────────────────────────────┘
                                │
                ── linq-channel 어댑터 정규화 ──
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

각 horizontal layer는 `linq-kit` 패키지 — FlowLinq은 이들을 조합하고 제품 특화 tool만 추가.

## `filebrowser` vendoring 전략

**결정: library / embedded binary로 vendor하고, 절대 fork하지 않는다.**

이유:

- File UI, preview, upload, share, 권한 — 직접 만들면 수개월의 frontend 작업이 공짜.
- 내부 확장 **필요 없음** — 워크플로우 상태는 filebrowser data store가 아니라 우리
  document model에 따로 둔다.
- Fork는 모든 CVE 대응 + 모든 upstream rebase + 모든 커스터마이징 충돌을 떠안는 것.
  우리 인력으로 감당 불가.

전술 옵션 (Phase 0에서 결정):

1. **Binary를 sidecar로 실행.** filebrowser는 자기 프로세스로 돌고, FlowLinq는 iframe +
   signed-cookie 인증으로 UI embed. 가장 깔끔한 분리. UX seam이 약간.
2. **Go library로 embed.** filebrowser는 Go; library로 wrap해서 우리 Go 서비스 안에 호스팅.
   UI 통합 더 매끄럽지만 stack에 Go 서비스 필요.
3. **Frontend build만 가져오고 backend는 직접.** 가장 많은 작업, 가장 많은 제어. 필요 없을 듯.

기본 계획: **옵션 1 (sidecar)**. 채팅 overlay에 통합 friction이 발견되면 재검토.
Phase 1 prototype 후 최종 결정.

무엇이 어디 사는가:

| Concern                  | Location                                          |
| ------------------------ | ------------------------------------------------- |
| File 바이트 (PDF, image) | R2 / S3 (via `linq-vault`)                        |
| 파일 browsing UI         | filebrowser (sidecar)                             |
| 폴더 / 권한 UI           | filebrowser                                       |
| Document metadata        | 우리 DB (`documents` 테이블, via `linq-vault`)    |
| Workflow state           | 우리 DB (`workflow_instances`, via `linq-flow`)   |
| 추출된 필드              | 우리 DB (`document_fields`, via `linq-lens`)      |
| Audit trail              | 우리 DB (via `linq-ops`)                          |
| LinqAI 채팅              | FlowLinq web 앱 (filebrowser UI overlay)          |

## Document object model

FlowLinq document는 **파일이 아니다**. Entity다:

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

  // 분류 (from linq-lens)
  classification: 'invoice' | 'po' | 'expense' | 'contract' | 'receipt' | 'unknown'
  classificationConfidence: number

  // 추출 필드 (분류에 따라 다름)
  extractedFields: Record<string, ExtractedField>

  // Workflow (from linq-flow)
  workflowInstanceId: WorkflowInstanceId | null
  state: 'PENDING' | 'ROUTED' | 'APPROVED' | 'REJECTED' | 'POSTED' | 'ARCHIVED'

  // Audit
  createdAt: ISODateString
  updatedAt: ISODateString
}
```

filebrowser의 파일시스템 계층은 이 위의 *view* — 폴더는 `businessId × classification × state × month`에서
파생되지, source of truth가 아니다.

## Channels

FlowLinq은 모든 고객 채널에서 문서를 받는다. Phase 우선순위:

1. **WhatsApp** — 메인 수집. Invoice 사진 → `ingestDocument` tool.
2. **Web 업로드** — 매니저 UI 쪽.
3. **Email forward** — `documents@<tenant>.flowlinq.app` 형태 주소 (나중).
4. **Voice** — 전화로 invoice 설명 (나중, 신기함 가치).
5. **Instagram DM / KakaoTalk** — 마이너, 최저 우선순위.

모든 채널은 `linq-channel`을 통해 공통 `IncomingMessage` + attachment로 정규화.

## AI / agent stack

- **Agent runtime**: `linq-agent` (BookLinq가 쓰는 동일한 runtime).
- **Model**: BookLinq의 `WHATSAPP_AI_MODEL` 설정 동일 (`@cf/zai-org/glm-4.7-flash` default).
  Env로 override. 다국어 + tool-calling 필수.
- **Tool gateway**: 위 나열된 FlowLinq 전용 tool. 각 tool은:
  - Pre-call policy check (via `linq-ops`)
  - Post-call audit log (via `linq-ops`)
  - 부작용 있는 tool은 idempotency key
- **Memory**: 대화 내 short-term, 사용자별 long-term (선호, 승인 history).

## Multi-tenant model

- 고객 한 명 = 한 **business** (tenant, 인도 SMB).
- Business 안: **user** (직원) + 역할 (uploader / approver / admin).
- Document, workflow, audit log는 business에 scope.
- 모든 테이블 `businessId` FK로 row-level isolation.

## Storage

- File 바이트: **Cloudflare R2** (egress 저렴, 인도 edge point).
- 구조화 데이터: **Postgres** (cloud는 Neon, on-prem 옵션은 self-host Postgres).
- Cache: hot lookup용 Cloudflare KV.

## Deployment topology

- **Cloud SaaS**가 메인 배포. FlowLinq은 Cloudflare Workers + R2 + managed Postgres.
- **Self-host 옵션** — on-prem을 요구하는 SMB용 (single-tenant Docker Compose 번들:
  filebrowser + Postgres + Worker 등가 Node 서비스). Backlog.

## Open question

- filebrowser sidecar vs embed — Phase 1 prototype 후 최종 결정.
- workflow state용 Postgres vs D1 — D1이 싸지만 audit + workflow 검색에 필요한 join은
  Postgres가 강함. Default Postgres (Neon).
- OCR provider — cost는 Tesseract local, 어려운 케이스는 cloud (Google Doc AI, AWS Textract).
  Phase 5 정확도 테스트 후 결정.
