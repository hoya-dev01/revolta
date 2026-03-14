멀티 에이전트로 다음 작업을 병렬 수행해줘: $ARGUMENTS

## 실행 절차

1. 먼저 작업을 분석하고, 병렬로 처리할 수 있는 독립적인 하위 작업으로 분해해.
2. 각 하위 작업에 적절한 에이전트 역할을 할당해:
   - `codegen`: 코드 구현 (worktree 격리)
   - `tester`: 테스트 작성 (worktree 격리)
   - `reviewer`: 코드 리뷰 (읽기 전용)
   - `docs`: 문서 업데이트
3. 에이전트 간 의존성이 있는 경우 실행 순서를 명시해.
4. 사용자에게 에이전트 분배 계획을 보여주고 승인을 받아.
5. 승인 후 `claude -w` (worktree 플래그)를 사용하여 에이전트를 병렬 실행해.

## 에이전트 실행 형식

```bash
# 예시: 병렬 에이전트 실행
claude -w --agent codegen -p "ChatGPT content script의 대화 감지 로직 구현" &
claude -w --agent tester -p "ChatGPT content script 단위 테스트 작성" &
wait
```

## 결과 통합

모든 에이전트 작업이 완료되면:
1. 각 worktree의 변경사항을 리뷰
2. 충돌이 있는 경우 수동 해결 방안 제시
3. 메인 브랜치에 머지
4. plan 파일 체크리스트 업데이트

⚠️ 에이전트 분배 계획을 먼저 보여주고, 사용자 승인 후에만 실행해.
