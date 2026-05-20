# FlowLinq

🌐 [English](README.md)

**AI-powered Document Operations Platform — 인도 SMB를 위한 WhatsApp-native EDMS.**

FlowLinq은 WhatsApp, 이메일, 사진으로 들어오는 invoice, 영수증, PO, 계약서의 혼돈을
구조화되고 감사 가능한 승인 워크플로우로 변환하고, 최종 결과를 기존 회계 시스템
(Tally, Zoho Books, ERPNext)에 자동 기재한다.

## 왜 FlowLinq인가

인도 SMB는 문서 운영을 WhatsApp + 이메일 + Excel + Tally로 굴린다. 승인은 구두, audit trail은
없고, 정산은 수작업이다. 기존 EDMS (DocuWare, M-Files, Kissflow)는 web-form 중심 —
인도 SMB 인력의 작업 채널과 맞지 않는다.

FlowLinq은 **WhatsApp-first**, **AI-native**, 그리고 **SMB가 이미 쓰는 도구와 통합**된다.

전체 포지셔닝: [docs/why.ko.md](docs/why.ko.md).

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

사용자 워크플로우:

1. **수집** — 어떤 채널이든 문서가 들어옴
2. **분류** — invoice / PO / expense / contract / receipt
3. **추출** — vendor, 금액, GST, 일자, 항목별 라인
4. **라우팅** — policy 기반 (금액 × 카테고리 × 부서)
5. **승인 요청** — 매니저가 WhatsApp으로 승인
6. **결정 기록** — 승인 / 반려가 사유와 함께 기록
7. **외부 기재** — Tally / Zoho Books / ERPNext에 push
8. **검색** — LinqAI 채팅으로 문서·근거 자료 검색

전체 아키텍처: [docs/architecture.ko.md](docs/architecture.ko.md).

## Family

FlowLinq은 devslab-kr의 **Linq** 제품 family 일원:

| Product      | 목적                              | Repo                                                |
| ------------ | --------------------------------- | --------------------------------------------------- |
| **BookLinq** | WhatsApp-first booking SaaS       | [jlc488/booklinq](https://github.com/jlc488/booklinq) |
| **FlowLinq** | AI Document Operations Platform   | 이 리포                                             |

두 제품은 공유 platform monorepo를 의존:

→ [`devslab-kr/linq-kit`](https://github.com/devslab-kr/linq-kit) — agent runtime, channel adapter, workflow engine, OCR/추출, 외부 시스템 bridge.

## Tech stack

- **TypeScript 6**
- **Cloudflare Workers** + **R2** (저장), **D1 / Postgres** (워크플로우 상태)
- **`filebrowser`** (gtsteffaniak fork) — 문서 UI 레이어로 library vendoring
- **LinqAI** 채팅 overlay — 대화형 검색
- **pnpm**, **Vite 8**, **Vitest**
- Domain: **`getflowlinq.app`** (구매 예정)

## Architecture 결정 (locked)

- **`filebrowser`는 fork가 아니라 library로 vendor한다.** 문서 browsing UI 표면만
  제공받고, 워크플로우 관련은 전부 우리 document model에 둔다.
  [docs/architecture.ko.md](docs/architecture.ko.md) 참조.
- **Agent + channel 로직은 단일 출처**: `@devslab-kr/linq-*` 패키지.
  FlowLinq에서 agent runtime 코드 중복 금지.
- **인도 시장 우선.** 통합 순서: Tally > Zoho Books > ERPNext.
- **V1에 eSign / UPI 없음.** 문서 워크플로우만 — 채널 + 워크플로우 가치가 검증된 뒤
  규제 레이어 추가.

## Roadmap

전체 계획: [docs/roadmap.ko.md](docs/roadmap.ko.md). Phase 요약:

- [ ] **Phase 0** — Foundation: 리포 bootstrap, `linq-kit` 소비, `filebrowser` vendoring 결정
- [ ] **Phase 1** — WhatsApp으로 문서 수집 → R2 + filebrowser UI
- [ ] **Phase 2** — OCR + 분류 (`linq-lens`) — invoice / PO / expense / contract / receipt
- [ ] **Phase 3** — Workflow 엔진 (`linq-flow`) — 라우팅, 승인, audit trail
- [ ] **Phase 4** — Tally 통합 (`linq-bridge`) — 첫 인도 SMB 유료 고객
- [ ] **Phase 5** — 대화형 검색 — "지난달 vendor X invoice 다 보여줘"
- [ ] **Phase 6** — Zoho Books + ERPNext bridge, 멀티 테넌트 스케일

## Status

**아직 bootstrap 안 됨.** 현재 이 리포는 문서만 있다. `linq-kit` Phase 0이 첫 패키지를
publish하면 코드가 land한다.

## License

Proprietary — devslab-kr internal use only.
