현재 변경사항을 분석하고 git commit을 수행해줘.

추가 지시사항이 있다면 반영해: $ARGUMENTS

## 절차

1. `git status`와 `git diff --staged`를 확인해. staged 파일이 없으면 `git diff`로 unstaged 변경사항을 확인해.
2. staged 파일이 없고 unstaged 변경사항만 있는 경우, 변경된 파일 목록을 보여주고 전체 stage(`git add -A`) 할지 선택적으로 stage 할지 사용자에게 물어봐.
3. 변경 내용을 분석해서 Conventional Commits 형식의 커밋 메시지를 작성해:
   - `feat:` 새 기능
   - `fix:` 버그 수정
   - `refactor:` 리팩토링 (기능 변경 없음)
   - `docs:` 문서 변경
   - `test:` 테스트 추가/수정
   - `chore:` 빌드, 설정, 의존성 등
   - `style:` 포매팅, 세미콜론 등 (코드 의미 변경 없음)
4. 변경 범위가 명확하면 scope를 포함해: `feat(content-script):`, `fix(supabase):`
5. 여러 종류의 변경이 섞여 있으면 분리 커밋을 제안해.

## 커밋 메시지 규칙

- 제목: 영어, 50자 이내, 소문자 시작, 마침표 없음
- 본문: 필요한 경우에만 작성. "왜" 변경했는지 설명. 한국어 가능.
- 예시:
  ```
  feat(collector): add ChatGPT conversation detection

  ChatGPT의 대화 컨테이너에 MutationObserver를 연결하여
  새 메시지를 실시간으로 감지하는 로직 구현
  ```

## 작성한 커밋 메시지를 사용자에게 보여주고 승인을 받은 후에만 `git commit`을 실행해.