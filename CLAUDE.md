# CLAUDE.md — Revolta

## Project Overview

Revolta는 AI 대화를 자동으로 수집·관리하는 크롬 확장 프로그램이다.
ChatGPT, Claude, Gemini의 대화를 자동 감지하여 클라우드에 저장하고,
태깅, 일일 리포트, 전문 검색 기능을 제공한다.

### 향후 로드맵

- Safari Web Extension
- iOS / Android 대시보드 앱 (React Native)
- 팀 협업 기능 (공유 태그, 공유 컬렉션)

---

## Tech Stack

### Chrome Extension (Frontend)

- **Runtime**: Manifest V3
- **Language**: TypeScript 5.x (strict mode)
- **UI Framework**: React 19 + Vite
- **Styling**: Tailwind CSS 4
- **State Management**: Zustand
- **Build Tool**: Vite + CRXJS (크롬 확장 HMR 지원)
- **Content Script Injection**: 플랫폼별 content script (ChatGPT, Claude, Gemini)

### Backend / Cloud

- **BaaS**: Supabase
  - **Database**: PostgreSQL (pg_trgm + Full-Text Search)
  - **Auth**: Supabase Auth (Google OAuth, Email/Password)
  - **Realtime**: Supabase Realtime (대시보드 실시간 동기화)
  - **Storage**: Supabase Storage (대화 첨부파일, 이미지)
  - **Edge Functions**: Deno 기반 서버리스 함수 (일일 리포트 생성, AI 자동 태깅, AI 요약)

### 공통

- **Monorepo**: Turborepo
- **Package Manager**: pnpm
- **Testing**: Vitest + Playwright (E2E)
- **Linting**: ESLint 9 (flat config) + Prettier
- **CI/CD**: GitHub Actions

### 선택 근거

- **Supabase**: Auth, DB, Realtime, Storage가 통합되어 1인/소규모 개발에 최적. Row Level Security로 멀티테넌트 보안 확보. REST/GraphQL API 자동 생성으로 향후 모바일 앱 연동이 즉시 가능.
- **CRXJS + Vite**: Manifest V3 확장 프로그램의 HMR 지원. React 컴포넌트 핫 리로드로 개발 속도 극대화.
- **Turborepo**: 확장 프로그램, 공유 라이브러리, Edge Functions를 하나의 모노레포로 관리. 향후 모바일 앱 패키지 추가 용이.
- **Zustand**: 확장 프로그램 popup, options, content script 간 상태 공유에 적합. Redux 대비 보일러플레이트 최소화.

---

## Project Structure

```
revolta/
├── CLAUDE.md
├── README.md
├── turbo.json
├── pnpm-workspace.yaml
├── .claude/
│   ├── settings.json
│   ├── settings.local.json
│   └── commands/
│       ├── plan.md
│       ├── review.md
│       └── test-coverage.md
├── packages/
│   ├── shared/                  # 공유 타입, 유틸, 상수
│   │   ├── src/
│   │   │   ├── types/           # 공통 TypeScript 타입
│   │   │   ├── schemas/         # Zod 런타임 유효성 검증 스키마
│   │   │   ├── utils/           # 공통 유틸리티 함수
│   │   │   └── constants/       # 공통 상수, 플랫폼별 DOM 선택자
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── supabase/                # Supabase 설정, 마이그레이션
│       ├── migrations/
│       ├── functions/            # Edge Functions
│       │   ├── daily-report/
│       │   ├── ai-tagging/
│       │   ├── ai-summary/
│       │   └── _shared/          # Edge Functions 공통 유틸
│       ├── seed.sql
│       └── config.toml
├── apps/
│   └── extension/               # Chrome Extension (Manifest V3)
│       ├── manifest.json
│       ├── vite.config.ts
│       ├── src/
│       │   ├── background/      # Service Worker
│       │   │   ├── index.ts
│       │   │   ├── collector.ts      # 대화 수집 오케스트레이터
│       │   │   └── storage-sync.ts   # Supabase 동기화
│       │   ├── content/         # Content Scripts (플랫폼별)
│       │   │   ├── chatgpt/
│       │   │   ├── claude/
│       │   │   └── gemini/
│       │   ├── popup/           # Extension Popup UI
│       │   │   ├── App.tsx
│       │   │   ├── pages/
│       │   │   └── components/
│       │   ├── options/         # Options 페이지 (설정)
│       │   ├── sidepanel/       # Side Panel UI (검색, 태그 관리)
│       │   ├── hooks/           # React 커스텀 훅
│       │   ├── stores/          # Zustand 스토어 (tag, report, search 등)
│       │   ├── lib/             # API 클라이언트, 헬퍼
│       │   │   ├── api/         # Supabase CRUD 래퍼 (conversations, messages, tags, search, reports)
│       │   │   └── supabase.ts  # Supabase 클라이언트 초기화
│       │   └── styles/          # Tailwind 글로벌 스타일
│       ├── tests/
│       │   ├── unit/
│       │   └── e2e/
│       └── package.json
└── docs/
    ├── architecture.md
    ├── api-schema.md
    └── plans/                   # 구현 계획 파일
```

---

## Development Commands

```bash
# 전체
pnpm install                    # 의존성 설치
pnpm dev                        # 전체 개발 서버 시작 (turbo)
pnpm build                      # 전체 빌드
pnpm test                       # 전체 테스트
pnpm lint                       # 전체 린트

# Extension
pnpm --filter extension dev     # 확장 프로그램 개발 모드 (HMR)
pnpm --filter extension build   # 프로덕션 빌드
pnpm --filter extension test    # 확장 프로그램 테스트만

# Supabase
pnpm --filter supabase db:push        # 마이그레이션 적용
pnpm --filter supabase db:reset       # DB 리셋 + 시드
pnpm --filter supabase functions:serve # Edge Functions 로컬 실행

# Git
gh pr create                    # PR 생성 (GitHub CLI 사용)
gh pr merge                     # PR 머지
```

---

## Code Conventions

### 명명 규칙
- 변수/함수: `camelCase`
- 컴포넌트/클래스/타입: `PascalCase`
- 상수: `UPPER_SNAKE_CASE`
- 파일명: 컴포넌트는 `PascalCase.tsx`, 나머지는 `kebab-case.ts`
- 디렉토리: `kebab-case`

### TypeScript
- `strict: true` 필수. `any` 사용 금지.
- 모든 함수에 반환 타입 명시.
- 공유 타입은 반드시 `packages/shared/src/types/`에 정의.
- Zod로 런타임 validation (API 응답, 사용자 입력).

### React
- 함수형 컴포넌트 + React Hooks만 사용.
- 컴포넌트 파일 하나에 하나의 export default 컴포넌트.
- Props 타입은 컴포넌트 파일 내에서 interface로 정의.
- 커스텀 훅은 `use` 접두사 필수 (`useConversations`, `useTags`).

### 에러 핸들링
- 모든 async 함수에 try-catch 필수.
- 사용자 대면 에러는 에러 바운더리로 처리.
- Supabase 호출 실패 시 exponential backoff 재시도.
- Content script에서의 DOM 접근 실패는 graceful degradation.

### 커밋 메시지
- Conventional Commits 형식: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- 한국어 또는 영어 (혼용 가능하나 제목은 영어 권장)

### 테스트
- 새 기능 추가 시 단위 테스트 필수.
- Content script 변경 시 해당 플랫폼 E2E 테스트 업데이트.
- 테스트 커버리지 80% 이상 유지 목표.

---

## Workflow Rules

### 계획 우선 원칙 (MANDATORY)
1. **모든 구현 작업 전에 반드시 plan을 먼저 작성**한다.
2. 계획은 `docs/plans/` 디렉토리에 마크다운 파일로 저장한다.
   - 파일명: GitHub 이슈 번호를 접두사로 사용 (예: `0010-turborepo-pnpm-monorepo-setup.md`)
3. 계획에는 다음을 포함한다:
   - 목표 및 범위
   - 구현 단계 (체크리스트)
   - 영향받는 파일 목록
   - 예상 리스크 및 대응 방안
   - 멀티 에이전트 활용 여부 및 에이전트 역할 분담
4. **계획을 사용자에게 보여주고 승인을 받은 후에만 코드 작성을 시작**한다.
5. 구현 중 계획에서 벗어나는 변경이 필요하면 사용자에게 알리고 계획을 업데이트한다.

### 멀티 에이전트 활용 가이드
복잡한 작업은 여러 에이전트를 병렬로 활용하여 효율을 높인다.

**에이전트를 분리해야 하는 경우:**
- 서로 독립적인 파일/모듈을 동시에 작업할 때
- 프론트엔드 + 백엔드를 동시에 구현할 때
- 코드 구현과 테스트 작성을 분리할 때
- 여러 플랫폼 content script를 동시에 개발할 때

**에이전트 역할 정의:**
- `codegen` (편집 가능): 기능 구현 담당. 지정된 디렉토리만 수정.
- `tester` (편집 가능, tests/ 범위): 테스트 코드 작성 전담.
- `reviewer` (읽기 전용): 코드 리뷰 및 개선 제안. 직접 수정 불가.
- `docs` (편집 가능, docs/ + README 범위): 문서 업데이트 전담.

**에이전트 분배 예시:**
```
기능: "ChatGPT content script 구현"
├── Agent 1 (codegen): apps/extension/src/content/chatgpt/ 구현
├── Agent 2 (codegen): packages/shared/src/types/ 타입 정의
├── Agent 3 (tester): apps/extension/tests/ 테스트 작성
└── Agent 4 (docs): docs/plans/ 계획 업데이트
```

### 작업 단위
- 한 번에 하나의 기능 단위만 구현한다.
- 기능 하나가 너무 크면 하위 태스크로 분해한다.
- 각 태스크 완료 시 git commit을 수행한다.

### 컨텍스트 관리
- 세션이 길어지면 `/compact`로 압축한다.
- 압축 시 다음을 반드시 보존한다:
  - 현재 편집 중인 파일 경로
  - 테스트 실패 메시지
  - 이번 세션에서 내린 아키텍처 결정사항
  - 현재 진행 중인 plan 파일 경로
- 완전히 다른 작업으로 전환 시 `/clear` 사용.

---

## Database Schema (Supabase / PostgreSQL)

핵심 테이블 참고용:

```sql
-- conversations: 수집된 AI 대화 (platform, title, url, started_at 등)
-- messages: 개별 메시지 (conversation_id, role, content, order)
-- tags: 사용자 정의 + 시스템 자동 태그
-- conversation_tags: 대화-태그 다대다 관계
-- daily_reports: 일일 리포트 (summary, topic_distribution, platform_distribution)
-- user_settings: 사용자 설정 (collection_enabled, platforms 등)
```

- `pg_trgm` 확장: 유사도 검색 (오타 허용, 부분 매칭)
- Full-Text Search: `tsvector` + GIN 인덱스 (`conversations`, `messages`)
- 상세 스키마는 `packages/supabase/migrations/`를 참조.

---

## Content Script 플랫폼 가이드

각 AI 플랫폼의 DOM 구조가 다르므로 플랫폼별 전략이 필요하다.

| 플랫폼 | URL 패턴 | 세션 ID 추출 | 대화 감지 방식 |
|---------|----------|-------------|----------------|
| ChatGPT | `chat.openai.com/*` | `/c/{id}` | MutationObserver on conversation container |
| Claude | `claude.ai/*` | `/chat/{id}` | MutationObserver on message list |
| Gemini | `gemini.google.com/*` | `/app/{id}` | MutationObserver on chat container |

- 각 content script는 독립적으로 동작해야 한다.
- 공통 로직은 `packages/shared/`로 추출한다.
- DOM 구조 변경에 대비해 선택자를 상수로 관리한다 (`shared/src/constants/{platform}-selectors.ts`).
- SPA 네비게이션 감지 필수 (`popstate` / `pushState` / `replaceState` 후킹).
- 스트리밍 응답 완료 판별: 인디케이터 소멸 감지 + debounce 기반 텍스트 안정화.

---

## Background Service Worker 아키텍처

Content Script와 Background 간 메시지 프로토콜:

| 메시지 타입 | 방향 | 설명 |
|------------|------|------|
| `CONVERSATION_STARTED` | Content → Background | 새 대화 세션 감지 |
| `MESSAGE_COLLECTED` | Content → Background | 개별 메시지 수집 완료 |
| `CONVERSATION_UPDATED` | Content → Background | 대화 메타데이터 갱신 |

- `collector.ts`: 수집 오케스트레이터. 메시지 수신, 인메모리 큐, 배치 처리.
- `storage-sync.ts`: Supabase 동기화. exponential backoff 재시도, 오프라인 큐 (`chrome.storage.local`).
- 다중 탭 동시 수집 지원. 탭별 데이터 격리.

---

## Security Notes

- Supabase Row Level Security (RLS) 필수 적용.
- API 키는 절대 코드에 하드코딩하지 않는다 (`.env` + `import.meta.env` 사용).
- Content script에서 수집한 데이터는 background service worker를 통해서만 외부 전송.
- 사용자 동의 없이 대화 데이터를 수집하지 않는다 (opt-in).
- 최초 설치 시 명시적 동의 플로우 필수. 플랫폼별 수집 on/off 개별 제어 가능.
