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
│   ├── 01-google-meet-action-items/
│   │   ├── README.md              # 문제 정의, 설계 의도, 결과
│   │   ├── workflow.json          # n8n export 파일
│   │   ├── flow-diagram.png       # 플로우 스크린샷
│   │   └── demo.mp4 (또는 Loom 링크)
│   │
│   ├── 02-channeltalk-faq-generator/
│   │   ├── README.md
│   │   ├── workflow.json
│   │   └── flow-diagram.png
│   │
│   └── 03-channeltalk-onboarding-checker/
│       ├── README.md
│       ├── workflow.json
│       └── flow-diagram.png
│
└── docs/
    └── n8n-setup-guide.md         # n8n 로컬 설치 및 워크플로우 import 방법
```

---

## 🚀 워크플로우 목록

### 01. Google Meet 회의록 → 액션아이템 자동 추출

| 항목 | 내용 |
|------|------|
| **해결한 문제** | 회의 후 액션아이템을 수동으로 정리하고 이슈 트래커에 등록하는 반복 작업 (건당 15~20분 소요) |
| **자동화 결과** | 회의 종료 → 회의록 자동 수집 → AI가 액션아이템 추출 → 이슈 트래커 자동 등록 (소요 시간: 0분) |
| **기술 스택** | n8n · Google Meet API · Claude API · Webhook |

**플로우 요약**

```
Google Meet 종료
    → Webhook 트리거
    → 회의록 텍스트 수집
    → Claude API로 액션아이템 추출 (담당자, 마감일, 내용)
    → JSON 파싱
    → 이슈 트래커에 자동 등록
    → Slack/Email 알림 발송
```

📂 [상세 문서 보기](workflows/01-google-meet-action-items/README.md)

---

### 02. 채널톡 고객 문의 → FAQ 초안 자동 생성

| 항목 | 내용 |
|------|------|
| **해결한 문제** | 고객 문의가 쌓여도 FAQ로 정리되지 않아 같은 질문이 반복됨. CS 담당자가 수동으로 FAQ를 작성하는 데 시간 소요 |
| **자동화 결과** | 최근 N일간 고객 문의를 자동 수집 → 유사 질문 클러스터링 → FAQ 초안 자동 생성 → 담당자 검토 요청 |
| **기술 스택** | n8n · Channel Talk Open API · Claude API · Webhook |

**플로우 요약**

```
Cron 트리거 (매일 09:00)
    → Channel Talk API로 최근 7일 문의 수집
    → 문의 내용 텍스트 전처리
    → Claude API로 유사 질문 그룹핑 + FAQ Q&A 초안 생성
    → Google Sheets에 FAQ 초안 저장
    → Slack으로 담당자에게 검토 요청 알림
```

📂 [상세 문서 보기](workflows/02-channeltalk-faq-generator/README.md)

---

### 03. 채널톡 고객사 온보딩 체크리스트 자동 관리

| 항목 | 내용 |
|------|------|
| **해결한 문제** | 신규 고객사 온보딩 시 세팅 항목을 수동으로 추적. 누락 발생 시 고객 불만으로 이어짐 |
| **자동화 결과** | 신규 고객 등록 → 온보딩 체크리스트 자동 생성 → 진행 상황 추적 → 미완료 항목 리마인드 |
| **기술 스택** | n8n · Channel Talk Open API · Google Sheets · Slack Webhook |

**플로우 요약**

```
Channel Talk 신규 고객 등록 Webhook
    → 고객 정보 파싱 (회사명, 플랜, 담당자)
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
| AI | Claude API (Anthropic) |
| 외부 서비스 | Channel Talk Open API · Google Meet API · Google Sheets API |
| 알림 | Slack Webhook · Email (SMTP) |
| 데이터 포맷 | JSON · Webhook |

---

## 💡 설계 원칙

**1. 문제 먼저, 도구는 나중에**
— 자동화할 가치가 있는 반복 업무를 먼저 식별하고, 그 다음에 n8n으로 구현합니다.

**2. 사람이 판단해야 할 건 사람에게**
— AI가 초안을 만들되, 최종 확인은 항상 담당자가 합니다. 완전 자동이 아니라 "자동 제안 + 사람 승인" 구조입니다.

**3. 실패해도 추적 가능하게**
— 모든 워크플로우에 에러 핸들링과 로그를 포함합니다. 실패 시 Slack 알림으로 즉시 인지할 수 있습니다.

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
