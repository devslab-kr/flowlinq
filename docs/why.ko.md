# Why FlowLinq

🌐 [English](why.md)

시장 포지셔닝, 경쟁 지형, 그리고 wedge.

## 시장

인도 SMB (5–200명)는 문서 운영을 다음의 카오스한 혼합으로 굴린다:

- **WhatsApp** — vendor가 invoice 사진 보내면, 매니저가 forward
- **이메일** + 아무도 일관되게 안 여는 PDF 첨부
- **Excel** — 승인 추적 (대부분 누락)
- **Tally** — 회계의 system of record
- **전화** — 승인 ("haan, approve kar do")

문제는 EDMS가 없어서가 아니라, 기존 EDMS가 인도 SMB 인력이 받아들이지 않을 *다른 일하는 방식*
(web form, 전용 소프트웨어)을 요구한다는 점이다.

## 왜 지금인가

세 가지 타이밍이 수렴:

1. **WhatsApp Business API 성숙** — Meta Cloud API가 지난 18개월 동안 WhatsApp을
   SMB 가격대에 programmable하게 만듦.
2. **LLM tool-calling 신뢰성** — 오픈 가중 다국어 모델이 인도 invoice 분류와 힌디/영어
   메시지 파싱을 production에 쓸 수준으로 도달.
3. **인도 SMB 디지털화 push** — GST 컴플라이언스, UPI 보편화, 코로나 이후 변화로
   인도 SMB가 처음으로 cloud tool에 익숙해짐.

## 경쟁 지형

| Player                | Channel        | 인도 SMB 점유 | AI-native | 약점                             |
| --------------------- | -------------- | ------------- | --------- | -------------------------------- |
| **Zoho Workflow**     | Web/email      | 매우 높음     | No        | WhatsApp + AI 둘 다 약함         |
| **Kissflow**          | Web            | 중간          | 제한적    | Enterprise 중심, mid-market+     |
| **DocuWare / M-Files** | Web            | 낮음          | 제한적    | SMB에 너무 무겁고 비쌈           |
| **Frappe / ERPNext**  | Web            | 높음 (개발자) | No        | 배포에 integrator 필요           |
| **Paperless-ngx**     | Self-host web  | 마이너        | No        | 워크플로우 없음, 인도 포커스 없음 |
| **FlowLinq**          | WhatsApp-first | (target)      | Yes       | 신생, traction 없음              |

시장은 크고 SMB 레이어에서 통합 안 됨. 방어 가능한 wedge는 **채널 + AI 결합** — Zoho는
WhatsApp-first로 빠르게 pivot하기에 너무 크고, AI-native 신생들은 대부분 서구
mid-market 노린다.

## Wedge

> **이미 Tally를 쓰는 인도 SMB를 위한 WhatsApp-native invoice / expense 승인.**

구체적으로:

- Vendor가 사업체 WhatsApp 번호로 invoice 사진을 보냄.
- FlowLinq이 분류하고, vendor / 금액 / GST를 추출하고, 승인자에게 라우팅.
- 승인자가 WhatsApp 메시지 받음: "Vendor ABC가 ₹45,000 invoice 보냈습니다. 승인할까요?"
- 승인자는 "approve" 또는 "reject — quantity wrong"으로 답신.
- FlowLinq이 승인된 invoice를 전체 audit trail과 함께 Tally에 기재.

이건 단일 workflow지 platform이 아니다. 이 하나의 workflow를 못 박은 *뒤*에야
platform이 될 자격을 얻는다.

## V1에서 만들지 않을 것

- **eSign / 전자서명** — CA 파트너십 (eMudhra, Sify, NSDL)이 필요한 규제 레이어.
  Workflow 비즈니스가 traction과 자본을 잡은 뒤 재진입.
- **결제 / UPI 수금** — PSP 파트너십 (Razorpay, Cashfree)과 RBI 컴플라이언스 필요.
  같은 논리: V1 아님.
- **범용 workflow builder** — Zoho와 정면승부. 패배.
- **HR / 휴가 / 환급** — 인접 vertical. Invoice flow에 유료 고객 생긴 뒤 보류 해제.

## 유통

- **WhatsApp self-onboarding PLG.** 시작에 demo call 불필요. WhatsApp으로 가입,
  Tally 인스턴스 지정, 10분 안에 첫 invoice 수집.
- **Tally 생태계 파트너십.** Tally는 연동 앱 파트너 프로그램 운영. 긴 구애 필요하지만 올바른 채널.
- **Bottom-up bookkeeper 채널.** 인도 bookkeeper 한 명이 SMB 10–50개를 담당 — bookkeeper
  한 명 전환하면 SMB 10+ 따라옴.

## 받아들이는 위험

- **분류 정확도가 사활.** 80% → 95%는 긴 길; 95% 미만은 승인자 신뢰를 빠르게 잃음.
  Build 시간 첫 6개월을 여기에 투자.
- **Meta의 WhatsApp 정책 변경.** `linq-channel` 추상화로 mitigation — 다른 채널
  (web, voice)을 escape hatch로 추가 가능.
- **Tally 통합은 사랑받지 못하는 엔지니어링.** TDL은 legacy, ODBC도 legacy. 화려한 버전은
  없다. 사랑받지 못하기 때문에 moat가 된다.

## 받아들이지 않는 위험

- **인도용으로 US 스타일 SaaS 만들기.** 인도 SMB는 가격 기대치 ($6–60/월 ACV), 신뢰 시그널,
  언어 mix가 다르다. India first로 설계.
- **eSign + 결제 + workflow 동시 진행.** 규제 진입 세 개. 죽음. 하나 골라서 증명하고 확장.
