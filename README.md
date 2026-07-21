# n8n-workflows

> 실무 문제를 발견하고, 노코드 자동화로 해결한 워크플로우 모음

반복적인 업무에서 비효율을 발견할 때마다 n8n으로 자동화 파이프라인을 설계하고 구축합니다.
각 워크플로우는 실제 업무 환경에서 사용 중이거나, 특정 비즈니스 시나리오를 기반으로 설계되었습니다.

---

## 📁 레포 구조

```
n8n-workflows/
├── README.md
├── workflows/
│   ├── 01-google-meet-action-items/       # 구현 · 사용 중
│   │   ├── README.md                       # 문제 정의, 설계 의도, 결과
│   │   ├── workflow.json                    # n8n export 파일
│   │   ├── flow-diagram.png                 # 플로우 스크린샷
│   │   └── demo.mp4 (또는 Loom 링크)
│   │
│   ├── 02-channeltalk-inquiry-triage/      # 설계 완료 · 구현 예정
│   │   ├── README.md
│   │   ├── workflow.json
│   │   └── flow-diagram.png
│   │
│   └── 03-channeltalk-onboarding-checker/  # 설계 중
│       ├── README.md
│       ├── workflow.json
│       └── flow-diagram.png
│
└── docs/
    └── n8n-setup-guide.md                   # n8n 로컬 설치 및 워크플로우 import 방법
```

---

## 🚀 워크플로우 목록

### 01. Google Meet 회의록 → 액션아이템 자동 추출  ·  `구현 · 사용 중`

| 항목 | 내용 |
|------|------|
| **해결한 문제** | 회의 후 회의록을 정리하고 할 일을 분배·등록하는 반복 작업 (매 회의마다 수작업) |
| **자동화 결과** | 회의록 생성 → AI가 요약·할 일 추출 → GitHub 이슈 자동 등록 + Slack 보고 (수작업 0) |
| **기술 스택** | n8n · Google Drive/Meet · Gemini API (Google) · GitHub API · Slack |

**플로우 요약**

```
Google Meet 회의록 생성 (Google Drive)
    → n8n 트리거
    → 회의록 텍스트 수집
    → Gemini API로 요약 · 할 일 추출 (담당자 · 내용)
    → JSON 파싱 · 분기
    → GitHub 이슈 자동 등록
    → Slack으로 회의 요약 · 할 일 보고
```

📂 [상세 문서 보기](workflows/01-google-meet-action-items/README.md)

---

### 02. 채널톡 상담 문의 → 자동 분류·태깅·라우팅 (+ FAQ 1차 응답)  ·  `설계 완료`

| 항목 | 내용 |
|------|------|
| **해결한 문제** | 매일 쌓이는 반복 CS 문의를 상담원이 수동으로 분류·배정하고, FAQ성 질문까지 일일이 응대 |
| **자동화 결과** | 문의가 들어오는 즉시 AI가 유형·긴급도 분류 → 자동 태깅·담당 배정, FAQ성은 봇이 1차 응답 → 상담원은 진짜 중요한 대화에 집중 |
| **기술 스택** | n8n · Channel Talk Open API (Webhook · Bot · ChatTag) · Gemini/Claude API · Slack |

**플로우 요약**

```
채널톡 상담 열림 / 새 메시지 (Webhook: userChatOpened)
    → n8n Webhook 트리거   ※ 폴링 대신 push로 rate limit 절약
    → (필요 시) Open API로 대화 컨텍스트 조회
    → Gemini/Claude로 문의 분류(유형·긴급도) + 요약 + FAQ 매칭 답변 생성 (JSON)
    → 분기(Switch)
        ├─ FAQ로 해결 가능  → Bot API로 상담에 자동 답변 전송
        └─ 사람 필요        → ChatTag로 태그 부착 + 담당 그룹/매니저 배정, 긴급 건 Slack 알림
    → 429 Too Many Requests 시 x-ratelimit-reset 기반 백오프 후 재시도
```

> **API 메모** — Base `https://api.channel.io`, 인증 헤더 `x-access-key` · `x-access-secret`.
> Rate limit은 leaky-bucket 방식(비 Enterprise 기준 대부분 엔드포인트 버킷 1,000·초당 10건, `GET /open/user-chats` 목록은 버킷 100). 그래서 목록 폴링 대신 **Webhook 이벤트 기반**으로 설계했습니다.

📂 [상세 문서 보기](workflows/02-channeltalk-inquiry-triage/README.md)

---

### 03. 채널톡 고객사 온보딩 체크리스트 자동 관리  ·  `설계 중`

| 항목 | 내용 |
|------|------|
| **해결한 문제** | 신규 고객사 온보딩 시 세팅 항목을 수동 추적. 누락 발생 시 고객 불만으로 이어짐 |
| **자동화 결과** | 신규 고객 등록 → 온보딩 체크리스트 자동 생성 → 진행 상황 추적 → 미완료 항목 리마인드 |
| **기술 스택** | n8n · Channel Talk Open API · Google Sheets · Slack |

**플로우 요약**

```
채널톡 신규 고객 등록 (Webhook)
    → 고객 정보 파싱 (회사명 · 플랜 · 담당자)
    → Google Sheets에 온보딩 체크리스트 자동 생성
        - [ ] ALF 초기 세팅
        - [ ] Knowledge Base 데이터 입력
        - [ ] 자동 응답 시나리오 설정
        - [ ] 테스트 메시지 발송
        - [ ] 담당자 교육 일정 확정
    → 담당 AX 컨설턴트에게 Slack 알림
    → 매일 Cron으로 미완료 항목 체크 → 리마인드 발송
```

📂 [상세 문서 보기](workflows/03-channeltalk-onboarding-checker/README.md)

---

## 🛠 기술 스택

| 카테고리 | 도구 |
|---------|------|
| 자동화 엔진 | n8n (self-hosted) |
| AI | Gemini API (Google) · Claude API (Anthropic) |
| 외부 서비스 | Channel Talk Open API · Google Meet/Drive · GitHub API · Google Sheets |
| 알림 | Slack · Email (SMTP) |
| 연동 방식 | Webhook · REST API · JSON |

---

## 💡 설계 원칙

**1. 문제 먼저, 도구는 나중에**
— 자동화할 가치가 있는 반복 업무를 먼저 식별하고, 그 다음에 n8n으로 구현합니다.

**2. 사람이 판단해야 할 건 사람에게**
— AI가 분류·초안·1차 응답을 만들되, 애매하거나 중요한 건은 항상 담당자에게 넘깁니다. 완전 자동이 아니라 "자동 처리 + 사람 확인" 구조입니다.

**3. 실패해도 추적 가능하게**
— 모든 워크플로우에 에러 핸들링과 로그를 포함합니다. 실패 시 Slack 알림으로 즉시 인지할 수 있습니다.

**4. 외부 API의 제약을 설계에 반영**
— 폴링 대신 Webhook(push)을 우선하고, Rate limit(429)에는 응답 헤더(`x-ratelimit-reset`) 기반 백오프·재시도를 넣어 안정적으로 동작하게 만듭니다.

---

## 📝 관련 글

- [n8n 자동화 시리즈 — Velog](https://velog.io/@00kang_jh/posts)

---

## 🚀 로컬에서 실행하기

```bash
# 1. n8n 설치
npm install n8n -g

# 2. n8n 실행
n8n start

# 3. 워크플로우 import
# n8n UI → Settings → Import from file → workflows/ 폴더의 .json 파일 선택
```

자세한 설정 가이드는 [n8n-setup-guide.md](docs/n8n-setup-guide.md)를 참고하세요.
