# Roadmap

🌐 [English](roadmap.md)

빈 리포에서 유료 고객 제품까지 FlowLinq의 진화 경로.

이 roadmap은 **무엇을 건너뛸지**에 대해서도 의견이 있다. Skip 항목은 의도적 — 이유는
[why.ko.md](why.ko.md).

## Phase 0 — Foundation

**목표.** 리포가 `linq-kit` 소비 가능 상태, 첫 feature 작업 시작 가능.

- [ ] `package.json`, `pnpm-workspace.yaml` (필요 시), `tsconfig.json`
- [ ] GitHub Packages 인증 `.npmrc`
- [ ] CI: 매 PR에서 `pnpm install` + `pnpm test` + `pnpm build`
- [ ] 첫 `@devslab-kr/linq-core` 의존성 추가 (인증 + 설치 경로 증명)
- [ ] 도메인 `getflowlinq.app` 구매 (Cloudflare Registrar)
- [ ] Landing page placeholder ("coming soon" + 이메일 capture)
- [ ] `filebrowser` vendoring 전략 결정 (sidecar vs embed)

**완료 기준.** 빈 FlowLinq 앱이 시작되고, `linq-core` symbol을 import하고,
`getflowlinq.app`에 land.

## Phase 1 — 문서 수집

**목표.** WhatsApp으로 문서 도착, R2에 저장, filebrowser UI에서 보임.

- [ ] WhatsApp 어댑터 (`linq-kit` Phase 2에서 BookLinq에서 추출된 `linq-channel` 사용)
- [ ] R2 bucket + signed upload (`linq-vault`)
- [ ] `filebrowser` 실행 (선택한 vendoring 전략)
- [ ] WhatsApp → R2 → filebrowser flow end-to-end 동작
- [ ] `documents` 테이블 생성 (id, businessId, storageRef, ingestChannel, createdAt)
- [ ] Multi-tenant scoping (당분간 single test business)
- [ ] `linq-ops` 통한 "document arrived" 이벤트 audit log

**완료 기준.** 테스트 폰의 실제 WhatsApp 사진이 전송 10초 이내에 filebrowser에 나타남.

## Phase 2 — 분류 + 추출

**목표.** 문서가 자동 분류되고 필드가 추출됨.

- [ ] OCR 파이프라인 (`linq-lens`)
- [ ] LLM 분류: invoice / PO / expense / contract / receipt / unknown
- [ ] Confidence scoring 포함 필드 추출
- [ ] 인도 invoice 변형: GST 번호, HSN 코드, vendor format
- [ ] Confidence threshold 로직 — low confidence → "사람 검토 필요"
- [ ] Low-confidence 케이스용 manual correction UI (filebrowser overlay)
- [ ] 정확도 추적용 실제 인도 invoice 50+ 테스트 corpus

**완료 기준.** 인도 invoice 50개 테스트 corpus에서 분류 정확도 >90%, 필드 추출 정확도 >80%.

## Phase 3 — Workflow 엔진

**목표.** 문서가 승인자에게 라우팅되고 WhatsApp으로 승인됨.

- [ ] Workflow state machine (`linq-flow`): `PENDING → ROUTED → APPROVED/REJECTED → POSTED`
- [ ] Approval chain 설정 (금액 × 카테고리 × 부서)
- [ ] 승인자에게 WhatsApp 알림: "Vendor X가 ₹Y invoice 보냈습니다. 승인할까요?"
- [ ] 승인자 답신 파싱 ("approve", "승인", "reject — quantity wrong")
- [ ] `linq-ops` 통한 매 transition audit trail
- [ ] 매니저 대시보드 (web) — 대기 중 승인 표시
- [ ] 무응답 승인 리마인더 (24h, 48h)

**완료 기준.** End-to-end: invoice 도착 → 분류 → 라우팅 → WhatsApp 승인 → state가
APPROVED + 전체 audit log.

## Phase 4 — Tally 통합

**목표.** 승인된 invoice가 실제 Tally 인스턴스에 기재. 첫 유료 고객.

- [ ] `linq-bridge` Tally connector — field test 후 TDL 또는 ODBC 결정
- [ ] Idempotency: 승인된 invoice 재기재해도 Tally에 중복 안 됨
- [ ] 정산 리포트 — FlowLinq 내용 vs Tally 내용
- [ ] Self-serve 온보딩: SMB가 setup wizard로 Tally 연결
- [ ] 첫 인도 SMB가 유료 결제
- [ ] 가격 결정 (잠정: 문서량 tier로 $20–60/월)

**완료 기준.** 유료 인도 SMB 한 곳이 FlowLinq에서 Tally로 30일 이상의 승인된 invoice 기재.

## Phase 5 — 대화형 검색

**목표.** 사용자가 LinqAI 채팅으로 문서와 근거 자료를 찾음.

- [ ] filebrowser UI에 LinqAI 채팅 overlay embed
- [ ] 문서 semantic search (embedding + vector store)
- [ ] Tool gateway: `searchDocuments`, `findEvidence`, `summarizeApprovals`
- [ ] 다국어 UI: 영어, 힌디어, 한국어
- [ ] "지난달 vendor X invoice 다 보여줘" / "show me last month's vendor X invoices" 둘 다 동작
- [ ] 출처 인용 — 답변이 특정 document ID를 가리킴

**완료 기준.** 서로 다른 SMB 사용자 3명이 "이제 수동 검색 안 하고 FlowLinq한테 물어본다"라고 말함.

## Phase 6 — Scale out

**목표.** 더 많은 bridge, 더 많은 채널, 더 많은 tenant.

- [ ] Zoho Books connector
- [ ] ERPNext connector
- [ ] 이메일 forward 수집 (`documents@<tenant>.flowlinq.app`)
- [ ] Voice 채널 (전화로 invoice 설명)
- [ ] Multi-tenant hardening (tenant isolation, tenant별 rate limit)
- [ ] On-prem 요구 SMB용 self-host 번들 (Docker Compose)
- [ ] 유료 SMB 10+

**완료 기준.** 매출이 recurring하고, 제품이 수동 온보딩 없이 tenant를 흡수 가능.

## Phase 7+ — 인접 확장

Phase 6 이후에만 고려:

- HR 문서 (휴가, 환급, 온보딩)
- Vendor / 계약 승인 워크플로우
- eSign 통합 (eMudhra / Sify 파트너십)
- 승인된 invoice의 UPI 수금 (Razorpay / Cashfree)

각각은 별도의 제품 결정 + 자체 go-to-market 위험. "phase 8 feature"로 다루지 말고
**별도 venture**로 다룬다.

## 이 roadmap이 의도적으로 건너뛴 것

- **범용 workflow builder.** Zoho에 패배. Wedge가 필요로 하는 workflow만 만든다.
- **Web-form-first UX.** 채널은 WhatsApp. Web은 매니저용, uploader용 아님.
- **모바일 앱.** WhatsApp이 모바일 앱이다.
- **V1에서 multi-LLM provider.** BookLinq이 정착한 거 그대로 — 나중에 재검토.
- **커스텀 OCR.** 동작하는 거 쓰고, 정확도 바닥 닿을 때만 교체.
- **Enterprise feature (SSO, SAML, custom role).** V1 구매자 아님.

## Cadence

- Phase 목표: 각 ~6–8주, 그러나 Phase 길이는 metric이 아니다.
  각 phase의 **Definition of Done**이 metric.
- 현재 phase의 Done criteria 충족 시에만 다음 phase로. 반쪽짜리 완료 금지.
