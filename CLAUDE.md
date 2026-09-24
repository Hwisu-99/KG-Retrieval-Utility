# Project Instructions for AI Agents

이 프로젝트의 공통 규칙은 `../AutoNote` 프로젝트와 동일하다.

## 커뮤니케이션

- 답변·설명·코드 주석은 항상 한국어로 작성한다. 기술 용어와 코드 식별자는 원문 그대로 둔다.
- Windows 콘솔에서 한글 stdout이 깨져 보일 수 있다 — 한글이 들어간 파일은 콘솔 출력 대신 파일을 직접 읽어서 확인한다.

## Git 커밋

- **커밋/푸시는 반드시 먼저 물어보고 승인받은 뒤에 한다.** auto mode에서도 예외 없다.
  작업을 끝내고 검증한 뒤, 무엇을 왜 바꿨는지 보고하고 "커밋하고 푸시할까?"라고 묻는다.
- 커밋 메시지는 영어, Conventional Commits 형식:

  ```
  <type>(<scope>): <소문자로 시작하는 한 줄 요약, 마침표 없음>

  <본문: 72자 안팎으로 줄바꿈. 무엇을 바꿨는지보다 "왜" 바꿨는지,
  이전 동작의 문제점, 설계상 선택 이유를 설명한다. 변경 파일이 여러
  개면 "- path/to/file.py: ..." 형태의 bullet로 파일별로 정리한다.>
  ```

  - `type`: `feat`, `fix`, `refactor`, `docs`, `chore`, `test` 등
  - `scope`: 변경된 영역 (예: `graph`, `ui`, `neo4j`, `readme`, `config`)
  - 사소한 변경은 본문 없이 제목 한 줄만 써도 된다.

## docs/ 작업 노트

- `docs/<area>/<task>.md`는 사용자의 개인 작업 노트(사고 과정, 설계 결정 기록)다. gitignore 대상이며 커밋에 포함되지 않는다.
- 작업을 마치고 커밋할 때, 변경된 영역을 설명하는 기존 docs 파일이 있으면 요청 없이 알아서 최신 상태로 업데이트한다. 새 docs 파일 생성은 별도로 요청받았을 때만 한다.

## 작업 관리 (Beads)

- 하위 태스크 관리는 Beads(`bd`)를 적극적으로 쓴다. TodoWrite/TaskCreate/markdown TODO 목록은 쓰지 않는다.
- 작업을 시작할 때 `bd ready`로 할 일을 확인하고, 착수하면 `bd update <id> --claim`, 끝나면 `bd close <id>`로 상태를 갱신한다.
- 여러 단계로 된 작업은 먼저 상위 이슈(`-t epic` 또는 `feature`)를 만들고, 하위 태스크는 `bd create "..." --parent <id>`로 쪼갠다.
  선후 관계가 있으면 `--deps blocked-by:<id>` 또는 `bd dep add`로 명시한다.
- 작업 도중 발견한 후속 작업·버그는 바로 이슈로 만든다 (`--deps discovered-from:<현재 id>`).
- 세션을 끝낼 때는 남은 작업을 이슈로 남기고 상태를 정리한 뒤 보고한다. 커밋/푸시/`bd dolt push`는 위 Git 규칙대로 먼저 물어본다.

## 설계 원칙

- 데이터 중복을 "어쩔 수 없다"며 제안하기 전에, 기존 데이터 모델에 필드 하나로 정규화할 수 있는 자리(per-source 리스트, join 구조 등)가 있는지 먼저 확인하고 그 방안을 우선 제시한다.

## Build & Test

_아직 없음 — 프로젝트 구성 후 추가_

## Architecture Overview

_아직 없음 — 프로젝트 구성 후 추가_


<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:1105d646 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/core-concepts/sync-concepts.md for details and anti-patterns.

## Agent Context Profiles

The managed Beads block is task-tracking guidance, not permission to override repository, user, or orchestrator instructions.

- **Conservative (default)**: Use `bd` for task tracking. Do not run git commits, git pushes, or Dolt remote sync unless explicitly asked. At handoff, report changed files, validation, and suggested next commands.
- **Minimal**: Keep tool instruction files as pointers to `bd prime`; use the same conservative git policy unless active instructions say otherwise.
- **Team-maintainer**: Only when the repository explicitly opts in, agents may close beads, run quality gates, commit, and push as part of session close. A current "do not commit" or "do not push" instruction still wins.

## Session Completion

This protocol applies when ending a Beads implementation workflow. It is subordinate to explicit user, repository, and orchestrator instructions.

1. **File issues for remaining work** - Create beads for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **Handle git/sync by active profile**:
   ```bash
   # Conservative/minimal/default: report status and proposed commands; wait for approval.
   git status

   # Team-maintainer opt-in only, unless current instructions forbid it:
   git pull --rebase
   git push
   git status
   ```
5. **Hand off** - Summarize changes, validation, issue status, and any blocked sync/commit/push step

**Critical rules:**
- Explicit user or orchestrator instructions override this Beads block.
- Do not commit or push without clear authority from the active profile or the current user request.
- If a required sync or push is blocked, stop and report the exact command and error.
<!-- END BEADS INTEGRATION -->
