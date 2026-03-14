# 0001. Turborepo + pnpm 모노레포 세팅

> GitHub Issue: #10
> 상위 이슈: #1 [Setup] Chrome 확장 프로그램 프로젝트 초기 세팅

---

## 목표

Turborepo + pnpm workspace 기반 모노레포 구조를 생성하여, Extension / Shared / Supabase 패키지를 하나의 레포에서 관리할 수 있도록 한다. `pnpm dev`, `pnpm build`, `pnpm test`, `pnpm lint` 명령이 turbo 파이프라인을 통해 전체 워크스페이스에서 실행되는 것이 최종 목표.

## 범위

**포함:**
- pnpm 설치 및 root 설정
- pnpm-workspace.yaml 구성
- turbo.json 파이프라인 설정
- 3개 패키지 디렉토리 및 package.json 생성 (`apps/extension`, `packages/shared`, `packages/supabase`)
- root package.json 스크립트 정의
- 패키지 간 의존성 참조 확인

**포함되지 않음:**
- Vite, React, TypeScript 등 실제 빌드 도구 세팅 (#11에서 수행)
- ESLint, Prettier 설정 (#12에서 수행)
- Supabase CLI 세팅 (#20에서 수행)
- 테스트 프레임워크 설정 (#18에서 수행)

## 구현 단계

- [ ] Step 1: pnpm 설치 🟢
  - `corepack enable && corepack prepare pnpm@latest --activate` 또는 `npm install -g pnpm`
  - pnpm 버전 확인

- [ ] Step 2: root `package.json` 생성 🟢
  - `name`: `revolta`
  - `private`: true
  - `packageManager`: pnpm 버전 고정
  - `scripts`: `dev`, `build`, `test`, `lint` (turbo 위임)
  - `devDependencies`: `turbo`

- [ ] Step 3: `pnpm-workspace.yaml` 생성 🟢
  ```yaml
  packages:
    - "apps/*"
    - "packages/*"
  ```

- [ ] Step 4: `turbo.json` 생성 🟡
  - `pipeline` 정의:
    - `build`: `dependsOn: ["^build"]`, outputs: `["dist/**"]`
    - `dev`: `cache: false`, persistent: true
    - `test`: `dependsOn: ["build"]`
    - `lint`: 독립 실행
  - `globalDependencies`: 필요 시 root 설정 파일 포함

- [ ] Step 5: `apps/extension/package.json` 생성 🟢
  - `name`: `extension`
  - `private`: true
  - `scripts`: `dev`, `build`, `test`, `lint` (placeholder — 후속 이슈에서 실제 명령 연결)
  - `dependencies`: `@revolta/shared` (workspace 참조)

- [ ] Step 6: `packages/shared/package.json` 생성 🟢
  - `name`: `@revolta/shared`
  - `main` / `types` 엔트리포인트
  - `scripts`: `build`, `test`, `lint` (placeholder)
  - `src/index.ts` 배럴 export 파일 생성 (빈 파일)

- [ ] Step 7: `packages/supabase/package.json` 생성 🟢
  - `name`: `@revolta/supabase`
  - `scripts`: `db:push`, `db:reset`, `functions:serve` (placeholder)

- [ ] Step 8: `pnpm install` 및 turbo 파이프라인 동작 확인 🟡
  - `pnpm install` — lockfile 생성 확인
  - `pnpm build` — turbo가 3개 패키지를 순서대로 빌드 시도
  - `pnpm dev` — turbo가 persistent 모드로 실행
  - `pnpm lint`, `pnpm test` — 에러 없이 실행 (placeholder이므로 no-op)

- [ ] Step 9: `.gitignore` 업데이트 🟢
  - `node_modules/`, `.turbo/`, `dist/` 추가

- [ ] Step 10: 패키지 간 의존성 참조 검증 🟡
  - `apps/extension`에서 `@revolta/shared` import 가능 확인
  - turbo `dependsOn: ["^build"]`에 의해 shared → extension 순서로 빌드되는지 확인

## 영향받는 파일

**생성:**
- `package.json` (root)
- `pnpm-workspace.yaml`
- `turbo.json`
- `apps/extension/package.json`
- `packages/shared/package.json`
- `packages/shared/src/index.ts`
- `packages/supabase/package.json`
- `pnpm-lock.yaml` (자동 생성)

**수정:**
- `.gitignore` — node_modules, .turbo, dist 추가

## 멀티 에이전트 활용 계획

이 작업은 규모가 작고 파일 간 의존성이 순차적이므로 **단일 에이전트**로 충분하다.

## 리스크 및 대응

| 리스크 | 대응 |
|--------|------|
| pnpm이 현재 환경에 미설치 | corepack 또는 npm -g로 설치. Node v25.8.1에서 corepack 지원 확인 필요 |
| Turborepo 최신 버전과 pnpm 호환성 | turbo@latest 설치 후 `turbo run build` 동작 확인. 문제 시 버전 고정 |
| placeholder 스크립트가 turbo에서 에러 | `echo "no-op"` 또는 `exit 0`으로 임시 처리하여 파이프라인 흐름 유지 |
| packages/shared를 extension에서 참조 시 빌드 없이 resolve 실패 | `tsconfig` paths 설정은 #12에서 처리. 이 단계에서는 package.json exports 필드로 기본 참조만 확인 |

## 의존성

- **선행 작업**: 없음 (첫 번째 작업)
- **환경 요구사항**: Node.js (v25.8.1 확인됨), pnpm (설치 필요)
- **후속 이슈**: #11 (Vite+CRXJS+React), #17 (shared 상세 구조), #12 (TS+Lint)
