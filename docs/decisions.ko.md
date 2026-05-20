# Decision Log

🌐 [English](decisions.md)

자명하지 않은 제품·기술 선택 전부 — 무엇을 골랐고, 무엇을 버렸고, 언제 다시 볼지.
결정에 반박하기 전에 먼저 읽을 것. 이미 검토한 대안들이 여기 있다.

각 항목 구조:

- **Status** — Decided / Deferred / X일 때 재검토
- **Date** — 결정(또는 마지막 수정) 일자
- **Context** — 어떤 질문에 답하는가
- **Options considered** — 검토했던 모든 대안
- **Decision** — 선택
- **Rationale** — 이유
- **Trade-offs accepted** — 의식적으로 받아들인 비용
- **Revisit when** — 다시 열어볼 trigger

Kit 레벨 공통 결정은
[`devslab-kr/linq-kit/docs/decisions.ko.md`](https://github.com/devslab-kr/linq-kit/blob/main/docs/decisions.ko.md).

---

## D-001 — 시장 방향: AI Document Operations Platform

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** BookLinq booking SaaS 다음으로 Linq stack을 어디로 가져갈지.

**검토 대안.**

| Option                                       | Verdict   | Why                                                                                                |
| -------------------------------------------- | --------- | -------------------------------------------------------------------------------------------------- |
| eSign / 전자서명 SaaS                        | Rejected  | 포화 (Digio, Leegality, SignDesk, Zoho Sign). 인도에서 licensed CA 파트너십 필요.                  |
| UPI / 결제 수금 SaaS                         | Rejected  | 포화 (Razorpay, Cashfree, PhonePe). PSP 파트너십 + RBI 컴플라이언스 필요.                          |
| AI Document Operations Platform (이것)       | **Chosen**| SMB EDMS + workflow 영역은 덜 포화. 규제 진입 장벽 없음. BookLinq IP 그대로 transfer.              |
| 범용 "workflow builder"                      | Rejected  | Zoho Workflow / Kissflow와 정면승부. 방어 가능 wedge 없음.                                         |

- **Rationale.** Channel-native AI (WhatsApp + 다국어 agent)는 BookLinq의 가장 강한 IP.
  Doc operations는 이 IP가 통하고 컴플라이언스가 진입 장벽이 아닌 인접 시장.
- **Trade-offs accepted.** 결제 대비 고객당 ACV 작음. Revenue ramp 느림.
  분류 정확도가 사활.
- **Revisit when.** Phase 4 고객이 "승인된 문서 위에 eSign 있으면 돈 더 낸다"고 말할 때
  — 그때 eSign이 Phase 7+ 확장 주제가 됨. Pivot 아님.

---

## D-002 — Vertical wedge: Tally 사용 SMB의 invoice / expense 승인

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Document workflow는 출시하기에 너무 넓다. 어느 vertical wedge부터?

**검토 대안.**

| Wedge                              | Verdict   | Why                                                                                  |
| ---------------------------------- | --------- | ------------------------------------------------------------------------------------ |
| Invoice / expense 승인             | **Chosen**| 인도 SMB에서 가장 큰 TAM. Tally 통합이 사랑받지 못하는 엔지니어링 = moat.             |
| HR 문서 (휴가 / 환급)              | Rejected  | 시장 작음, urgency 낮음, AI 차별점 약함.                                              |
| Vendor / 계약 승인                 | Rejected  | ACV 높지만 enterprise sales cycle 김. PLG self-onboarding에 맞지 않는 shape.          |
| 범용 workflow builder              | Rejected  | (D-001 참조) Zoho 승리.                                                               |

- **Rationale.** Invoice 승인은 빈도 높고, ROI 명확 ("주당 5시간 절약"),
  Tally에 묶임 — Tally는 인도 SMB 회계의 system of record이자 유통 파트너 채널.
- **Trade-offs accepted.** Tally 통합은 legacy 기술 (TDL, ODBC). 인도 invoice field
  variant 정복에 6+ 개월. 의도적으로 그 시간 할당.
- **Revisit when.** Invoice flow에 유료 SMB 10명 확보 후 — 인접 워크플로우
  (expense → vendor → contract)를 Phase 7+로 시작.

---

## D-003 — EDMS base: `filebrowser` library vendoring

- **Status.** Decided (Phase 1 prototype로 검증)
- **Date.** 2026-05-21
- **Context.** Document 저장 + browse UI 레이어 필요. 만들 것인가 OSS base 가져올 것인가?

**검토 대안.**

| Option                          | Verdict      | Key signal                                                                              |
| ------------------------------- | ------------ | --------------------------------------------------------------------------------------- |
| `filebrowser` (gtsteffaniak fork)| **Chosen**   | 단일 Go 바이너리, file/folder UI + preview + sharing 공짜, 활발한 fork.                 |
| `Paperless-ngx`                 | Strong runner-up | 이미 OCR / tag / correspondent 가진 doc-management 제품. GitHub star 70K+. 무겁다 (Python + Django). filebrowser 실패 시 최선의 대안. |
| `Mayan EDMS`                    | Rejected     | "Workflow 내장 EDMS"에 가장 가깝지만 Python + 복잡한 배포. V1에 과함.                  |
| `Docspell`                      | Rejected     | JVM stack — 우리가 원치 않는 운영 무게.                                                 |
| `Nextcloud`                     | Rejected     | 너무 무거움, PHP, enterprise 프레이밍 SMB에 안 맞음.                                    |
| `Seafile` / `OwnCloud`          | Rejected     | Sync 중심, 워크플로우 친화 아님.                                                        |
| Build from scratch              | Rejected     | 이미 commoditize된 frontend 3–6개월 작업.                                              |

- **Rationale.** filebrowser는 즉시 사용 가능한 file UI를 주는 가장 가벼운 base.
  filebrowser의 data model이 우리 data model일 필요 없음 — 워크플로우 상태는
  옆에 Postgres에 따로 둠.
- **Trade-offs accepted.** filebrowser 멘탈 모델은 "폴더 안 파일", "entity로서의 문서" 아님.
  우리 document object model로 bridge ([architecture.ko.md](architecture.ko.md) 참조).
  gtsteffaniak fork는 1인 유지보수 — CVE 대응 시간에 작은 bus-factor 위험.
- **Revisit when.** Phase 1 prototype에서 실제 통합 friction (auth, embedding,
  iframe seam) 발견 — Paperless-ngx로 fallback.

---

## D-004 — `filebrowser` 통합: vendor, fork 금지

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** filebrowser를 쓴다면 stack에 어떻게 포함시킬지.

**검토 대안.**

| Approach                              | Verdict           | Rationale                                                                |
| ------------------------------------- | ----------------- | ------------------------------------------------------------------------ |
| Fork (`devslab-kr/`에 자체 사본)      | Rejected          | 모든 CVE, 모든 upstream rebase, 모든 커스터마이징 충돌 떠안기.            |
| Library / binary로 vendor (no fork)   | **Chosen**        | 의존성으로 다룸. 커스터마이징은 우리 wrapper에, 그들 코드에 안 함.        |
| 프로세스로 sidecar 실행               | **Phase 0 default**| 가장 깔끔한 분리, 가장 낮은 결합. UI는 iframe + signed cookie.            |
| Go library로 embed                    | Backup            | UI 통합 더 매끄럽지만 Cloudflare Workers stack에 Go 서비스 끌어들임.      |
| Frontend build만 재사용               | Rejected          | 가장 많은 작업, 가장 많은 제어. 그 정도 커스터마이징 필요 없음.           |

- **Rationale.** Forking은 OSS-on-OSS 스타트업이 망하는 가장 흔한 방법.
  프로세스 boundary로 vendor하면 upstream 업데이트가 trivial.
- **Trade-offs accepted.** iframe boundary에서 약간의 UX seam. 배포할 프로세스 2개.
- **Revisit when.** Phase 1 prototype에서 iframe seam이 LinqAI 채팅 overlay를 막을 때
  — "Go library로 embed"로 fallback.

---

## D-005 — Storage: Cloudflare R2

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** 파일 바이트는 어디 사는가?

**검토 대안.**

| Option                          | Verdict   | Why                                                                          |
| ------------------------------- | --------- | ---------------------------------------------------------------------------- |
| Cloudflare R2                   | **Chosen**| Egress 무료, S3 호환 API, 인도 edge, BookLinq가 이미 Cloudflare에 있음.       |
| AWS S3                          | Rejected  | SMB 볼륨에서 egress cost 누적. 플랫폼 통합 이득 없음.                          |
| Self-host (filebrowser local FS)| Backlog   | On-prem 옵션에 나중 필요. V1 아님.                                            |
| Backblaze B2                    | Rejected  | Raw storage는 더 저렴하지만 Workers 통합 약함.                                |

- **Trade-offs accepted.** Cloudflare lock-in. `linq-vault` 추상화로 mitigation —
  backend 교체는 contained refactor.
- **Revisit when.** On-prem 배포가 sales 요구사항이 될 때.

---

## D-006 — Workflow state DB: Postgres (Neon), D1 아님

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** Workflow state, audit log, document metadata는 어디 사는가?

**검토 대안.**

| Option            | Verdict   | Why                                                                       |
| ----------------- | --------- | ------------------------------------------------------------------------- |
| Postgres (Neon)   | **Chosen**| Audit + workflow join이 자명하지 않음. Postgres가 안전 default.            |
| D1 (Workers SQLite)| Rejected | 저렴하지만 우리가 할 join-heavy 쿼리에 제한.                              |
| Hyperdrive + 자체 Postgres| Backlog | Neon 한계 닿으면 옵션.                                              |

- **Trade-offs accepted.** D1보다 약간 높은 월 비용. Neon 무료 tier cold-start
  지연 (Neon auto-resume으로 mitigation).
- **Revisit when.** Tenant별 데이터 볼륨이 Neon 안락 ceiling 초과.

---

## D-007 — OCR provider: deferred + shortlist

- **Status.** Deferred (Phase 5에서 결정)
- **Date.** 2026-05-21
- **Context.** 어떤 OCR 엔진이 인도 invoice 이미지에서 구조화 필드를 뽑는가?

**Phase 5 평가용 shortlist.**

| Engine                  | 비용                | 품질             | 메모                                              |
| ----------------------- | ------------------- | ---------------- | -------------------------------------------------- |
| Tesseract (local)       | 무료                | 스캔에 mediocre  | 저렴 경로 default. Worker 또는 sidecar 안에서.     |
| Google Document AI      | 페이지당 유료       | 우수             | 인도 변형 포함 invoice layout에 강함.              |
| AWS Textract            | 페이지당 유료       | 우수             | Google과 비슷.                                     |
| Mistral OCR             | 유료                | 신생, 유망       | Phase 5 bake-off 가치.                            |
| Multimodal LLM only (OCR 없음)| 토큰당 비용    | 가변             | V1에 매력적; 노이즈 스캔에서 품질 떨어짐.          |

- **Deferral 근거.** 실제 인도 invoice 테스트 corpus 없이 결정 불가능. 정확도 바닥이
  기준이고, Phase 2까지 데이터 없음.
- **Phase 2–4 동안 default.** Multimodal LLM only (Workers AI vision 모델) — 출시
  가장 저렴, 90% 미만에서 정체 시 은퇴.

---

## D-008 — LLM model: BookLinq 선택 상속

- **Status.** Decided (BookLinq와 동일한 env override 정책)
- **Date.** 2026-05-21
- **Context.** FlowLinq agent runtime을 어떤 LLM이 구동하는가?

**BookLinq에서 상속** (`apps/api/src/whatsapp/ai/workers-ai.ts::DEFAULT_MODEL` 참조):

| Model                                 | Status                                                  |
| ------------------------------------- | ------------------------------------------------------- |
| `@cf/zai-org/glm-4.7-flash`           | **Default.** 다국어 + tool calling. Fit 좋음.           |
| `@cf/openai/gpt-oss-120b`             | 검증된 fallback. ~3x 비용, 최고 안정성.                 |
| `@cf/meta/llama-3.3-70b-instruct-fp8-fast` | 실패 — 한국어 정확도.                              |
| `@cf/meta/llama-4-scout-17b-16e-instruct` | 실패 — 한국어 정확도.                               |

- **Rationale.** BookLinq의 이미 검증된 stack과 분리할 이유 없음. `linq-agent`가
  동일한 `WHATSAPP_AI_MODEL` env override를 expose.
- **Trade-offs accepted.** Cloudflare Workers AI에 lock. `linq-agent`의 provider
  추상화로 mitigation — 필요시 Anthropic / OpenAI direct가 swap 후보.
- **Revisit when.** Workers AI 카탈로그의 새 모델이 힌디어나 인도 invoice 분류에서
  glm-4.7-flash를 의미 있게 상회.

> **인프라 관점 (모델 선택과 분리).** 추론이 *어디서* 도는가 (Cloudflare managed vs
> Together / Fireworks managed vs vLLM / SGLang / TGI self-host)는 kit 레벨에서 추적 —
> [`linq-kit/docs/decisions.ko.md` D-110](https://github.com/devslab-kr/linq-kit/blob/main/docs/decisions.ko.md#d-110)
> 참조. DPDP Act 시행은 BookLinq 단독으로 정당화 안 되는 India self-host를 강제할 수 있는
> FlowLinq-특화 trigger; 이 문서가 그 watch를 owns함.

---

## D-009 — 지역: India first

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** V1은 어느 시장을 노리는가?

**검토 대안.**

| Region                | Verdict   | Why                                                                            |
| --------------------- | --------- | ------------------------------------------------------------------------------ |
| 인도 (SMB)            | **Chosen**| Workflow / EDMS에서 가장 underserved한 SMB 시장. Tally가 통합 moat.            |
| 한국 / 일본           | Rejected  | TAM 작음, 토종 SaaS 견고, WhatsApp 사용 낮음.                                  |
| 미국 / 서구 SMB       | Rejected  | Box / Docusign / Bill.com으로 포화. 우리 cost 구조에 맞지 않는 경제학.          |
| SEA (베트남 / 인도네시아 / 필리핀)| Backlog | 인도와 유사 프로필. Phase 6+에서 재검토.                              |

- **Trade-offs accepted.** 인도 SMB ACV ($6–60/월)는 high-volume PLG motion 필요.
  DPDP Act 2023 데이터 로컬라이제이션 — 인도 edge presence 필요할 수 있음
  (Cloudflare 인도 edge가 처리).
- **Revisit when.** 첫 인도 SMB 10명 production에 진입. 그때 SEA 확장.

---

## D-010 — 브랜드 이름: FlowLinq

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** `[purpose]Linq` family 내 제품 이름.

**검토한 top 후보.**

| Name        | Signal                            | Why not                                                                    |
| ----------- | --------------------------------- | -------------------------------------------------------------------------- |
| **FlowLinq**| Document flow, workflow           | **Chosen.** BookLinq cadence 일치, 워크플로우 본질 포착, 인도 발음 가능.   |
| VaultLinq   | Storage / archive / security      | 2순위. "self-host EDMS" 프레이밍으로 pivot 시 더 강함.                     |
| DocuLinq    | 문서 중심, 직설                   | Generic, 3음절, 덜 distinct.                                               |
| TrailLinq   | Audit trail 강조                  | 차별화는 좋지만 첫 인상에 카테고리 전달 안 됨.                             |
| OpsLinq     | Operations, 원래 brand sketch     | "Ops"가 SMB 사장에게 tech jargon으로 읽힘.                                 |
| PaperLinq   | Paperless office                  | 너무 좁음 — paper doc 너머 확장 제한.                                       |
| FolioLinq   | Document portfolio                | 세련되지만 obscure.                                                         |

- **Rationale.** FlowLinq elevator pitch — "FlowLinq manages where your company's
  documents flow." BookLinq와 같은 음절 리듬으로 family 명확.
- **Trade-offs accepted.** "Flow"는 B2B SaaS 브랜딩에 과사용; `Linq` 접미사가 차별화 운반.
- **Revisit when.** 트레이드마크 검색 (USPTO / IP India / WIPO Class 9, 42)에서 충돌
  — VaultLinq이 fallback.

---

## D-011 — Domain: `getflowlinq.app` (canonical), `.com` 보류

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** 출시할 메인 도메인.

**조사 결과.**

- `flowlinq.com`은 parker 보유 중. 인수 가격 미상; traction까지 보류.
- `flowlink.com` (유사 spelling)은 무관 회사들이 보유 (네덜란드 도매상, eCommerce 통합)
  — 직접 경쟁자 아님.
- "flowlinq" SERP는 비어있음 — 점령 쉬움.

**계획.**

| Domain                  | Action                                                                            |
| ----------------------- | --------------------------------------------------------------------------------- |
| `getflowlinq.app`       | **지금 구매.** Canonical. `.app`은 Google HSTS preload로 HTTPS 강제.              |
| `flowlinq.in`           | 인도 법인 / cofounder 생길 때 구매 (인도 registrant 필요). Trust signal.          |
| `flowlinq.io`           | Defensive, ~$30. 옵션.                                                            |
| `flowlinq.co`           | Defensive, ~$30. 옵션.                                                            |
| `flowlinq.com`          | 보류. Traction 후 재협상; parker 가격은 우리 성공과 함께 인플레이션.              |

- **Trade-offs accepted.** 사용자가 `flowlink`로 오타. Google 자동 보정이 12+개월 동안
  거기로 push. 브랜드 교육이 비용.
- **Revisit when.** Phase 4 (첫 유료 고객) — 그때 `.com` 인수 weigh.

---

## D-012 — Repo 구조: 공유 kit + product repo (Option B)

- **Status.** Decided
- **Date.** 2026-05-21
- **Context.** BookLinq, FlowLinq, 미래 제품들 사이의 공유 코드를 어떻게 조직할지.

**검토 대안.**

| Option                                  | Verdict   | Why                                                                          |
| --------------------------------------- | --------- | ---------------------------------------------------------------------------- |
| A — 완전 multi-repo (각 공유 패키지 = 자체 repo)| Rejected | Publish 의례 × N. 작은 팀에 조정 오버헤드 너무 큼.                  |
| B — `linq-kit` 공유 monorepo + product repo| **Chosen** | 공유 코드 atomic refactor 친화. 제품은 독립 배포 / secret / access. |
| C — 모든 것 단일 monorepo               | Rejected  | BookLinq 이주 비용 + 배포 / secret 분리 어려움. 제품별 판매 약함.            |

- **Rationale.** 여러 제품 + 공유 safety/agent layer가 명확해지는 상황에서 B가 장기
  유연성 최선. C에 끌렸지만 위 이유로 거부.
- **Trade-offs accepted.** Publish cycle (kit version bump → product PR)이 실제
  friction. Changesets + Renovate auto-PR로 mitigation.
- **Revisit when.** 인력 > 5명 — 그때 kit 패키지를 자체 repo로 분할해서 병렬 팀
  소유권 평가.

(Kit 측 근거: `linq-kit/docs/decisions.ko.md` D-101 참조.)

---

## D-013 — V1 명시적 out-of-scope 기능

- **Status.** Decided
- **Date.** 2026-05-21

| Feature                                | 제외 이유                                                                  |
| -------------------------------------- | -------------------------------------------------------------------------- |
| eSign / 전자서명                       | 규제. CA 파트너십 필요. (D-001)                                            |
| UPI / 결제 수금                        | 규제. PSP 파트너십 필요.                                                   |
| 범용 workflow builder                  | Zoho에 패배. (D-002)                                                       |
| 모바일 앱                              | WhatsApp이 모바일 앱이다.                                                  |
| Uploader용 web-form-first UX           | 인도 SMB 인력에 잘못된 채널.                                               |
| Enterprise SSO / SAML / custom role    | V1 구매자 아님.                                                            |
| V1에서 multi-LLM provider 전환         | 분리할 이유 생기기 전까지 BookLinq stack 상속. (D-008)                     |
| 자체 OCR 파이프라인                    | 기존 엔진 사용. 정확도 바닥이 요구할 때만 교체. (D-007)                    |

**Revisit when.** Phase 6+ — 인접 확장은 feature flag가 아닌 별도 venture로 고려.

---

## 새 결정 추가법

1. 다음 빈 `D-NNN` 번호 픽.
2. 이 파일 상단의 section template 사용.
3. 명백해 보여도 **검토 대안** 항상 나열 — 미래 reader는 우리 context 모름.
4. **Revisit when** 명시. Revisit trigger 없는 결정은 종교다.
5. Rationale에서 관련 코드, PR, 외부 페이지 링크.
