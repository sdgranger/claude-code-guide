# Claude Code 확장하기 가이드

> 출처: [Claude Code 확장하기](https://code.claude.com/docs/ko/features-overview)
> CLAUDE.md, Skills, Subagents, Hooks, MCP, Plugins을 언제, 어떻게 사용할지 이해합니다.

---

## 1. 확장 기능 전체 지도

```
┌─────────────────────────────────────────────────────────┐
│                    에이전트 루프                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │컨텍스트수집│→│ 작업 수행 │→│ 결과 검증 │→ (반복)      │
│  └──────────┘  └──────────┘  └──────────┘              │
│       ↑              ↑              ↑                   │
│  CLAUDE.md      MCP 서버        Hooks                   │
│  Skills        Subagents       Plugins                  │
│  (항상 로드)    (온디맨드)       (이벤트 트리거)           │
└─────────────────────────────────────────────────────────┘
```

### 기능별 역할 요약

| 기능 | 수행 작업 | 사용 시기 | 예시 |
|------|---------|---------|------|
| **CLAUDE.md** | 모든 대화에 로드되는 지속 컨텍스트 | "항상 X를 수행" 규칙 | "pnpm 사용, 커밋 전 테스트 실행" |
| **Skill** | 지침, 지식, 호출 가능한 워크플로우 | 재사용 가능한 콘텐츠, 반복 작업 | `/deploy`, API 스타일 가이드 |
| **Subagent** | 격리된 컨텍스트에서 작업 후 요약 반환 | 컨텍스트 격리, 병렬 작업 | 보안 리뷰, 코드 검색 |
| **Agent Team** | 여러 독립 세션 조정 | 대규모 병렬 연구/개발 | 보안+성능+테스트 동시 검토 |
| **MCP** | 외부 서비스 연결 | 외부 데이터/작업 | DB 쿼리, Slack 게시 |
| **Hook** | 이벤트에서 실행되는 결정론적 스크립트 | 예외 없는 자동화 (LLM 없음) | 편집 후 ESLint 자동 실행 |
| **Plugin** | 위 기능들을 번들로 패키징 | 팀/커뮤니티 공유 | 코드 인텔리전스 플러그인 |

- [ ] 각 확장 기능의 역할 차이 이해하기
- [ ] 현재 프로젝트에 가장 필요한 확장 기능 판단하기

---

## 2. 기능 선택 가이드: "어떤 걸 써야 할까?"

### CLAUDE.md vs Skill vs Rules

| 측면 | CLAUDE.md | `.claude/rules/` | Skill |
|------|-----------|-------------------|-------|
| **로드 시점** | 모든 세션 | 모든 세션 또는 파일 매칭 시 | 온디맨드 |
| **범위** | 전체 프로젝트 | 파일 경로로 범위 지정 가능 | 작업별 |
| **최적 용도** | 핵심 규칙, 빌드 명령 | 언어별/디렉토리별 가이드 | 참조 자료, 반복 워크플로우 |

**판단 기준**:
- Claude가 **항상** 알아야 하면 → **CLAUDE.md**
- 특정 파일 작업 시만 필요하면 → **Rules** (`.claude/rules/`)
- 가끔만 필요하거나 `/명령`으로 실행하면 → **Skill**

> **경험 법칙**: CLAUDE.md를 200줄 이하로 유지하세요.
> 커지면 참조 콘텐츠를 Skill이나 Rules로 이동하세요.

### Skill vs Subagent

| 측면 | Skill | Subagent |
|------|-------|----------|
| **정의** | 재사용 가능한 지침/워크플로우 | 격리된 워커 |
| **주요 이점** | 컨텍스트 간 콘텐츠 공유 | 컨텍스트 격리 (요약만 반환) |
| **최적 용도** | 참조 자료, 호출 가능 워크플로우 | 많은 파일 읽기, 병렬 작업 |

> **컨텍스트 격리가 필요하면 Subagent**, 콘텐츠 공유가 필요하면 Skill

### MCP vs Skill

| 측면 | MCP | Skill |
|------|-----|-------|
| **제공** | 도구 및 데이터 접근 (능력) | 지식, 워크플로우 (방법) |
| **예시** | DB 연결, Slack 통합 | DB 스키마 문서, 쿼리 패턴 |

> MCP = **"무엇을 할 수 있는가"** / Skill = **"어떻게 잘 할 것인가"**
> 함께 사용하면 강력합니다: MCP로 DB 연결 + Skill로 쿼리 패턴 가르치기

- [ ] 프로젝트에 필요한 기능 유형 구분해보기
- [ ] CLAUDE.md가 200줄 넘으면 Rules/Skill로 분리 계획 세우기

---

## 3. CLAUDE.md 심화

### 위치별 범위

| 위치 | 범위 | 공유 |
|------|------|------|
| `./CLAUDE.md` | 프로젝트 (팀 공유) | git에 체크인 |
| `~/.claude/CLAUDE.md` | 개인 (모든 프로젝트) | 본인만 |
| `.claude/rules/*.md` | 파일/디렉토리별 | git에 체크인 |

### 파일 import 문법
```markdown
# CLAUDE.md
프로젝트 개요는 @README.md 참조
npm 명령은 @package.json 참조

# 추가 지침
- git 워크플로우: @docs/git-instructions.md
- 개인 설정: @~/.claude/my-project-instructions.md
```

- [ ] 프로젝트 CLAUDE.md에 `@파일명` import 사용해보기

---

## 4. Skills (커스텀 슬래시 명령)

> **2026 변경**: 기존 **Custom Commands(`.claude/commands/`)는 Skills로 통합**되었습니다.
> `.claude/commands/deploy.md` 와 `.claude/skills/deploy/SKILL.md` 는 동일하게 `/deploy`로 동작.
> 기존 commands 파일은 그대로 작동하며, Skills는 보조 파일/자동 호출/추가 frontmatter 옵션 제공.

### Skill 위치별 범위

| 위치 | 적용 |
|------|------|
| `~/.claude/skills/<이름>/SKILL.md` | 모든 프로젝트 (개인) |
| `.claude/skills/<이름>/SKILL.md` | 현재 프로젝트만 (팀 공유) |
| `<plugin>/skills/<이름>/SKILL.md` | 플러그인 활성화된 곳 |
| 관리형 (organization) | 조직 전체 |

> **Live change detection**: 기존 skills 디렉토리에 파일 추가/수정/삭제는 **세션 재시작 없이** 즉시 반영됨. 단, 최상위 skills 디렉토리를 새로 만든 경우엔 재시작 필요.

### Skill 생성 예시

```markdown
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: GitHub 이슈 분석 및 수정
argument-hint: [issue-number]
disable-model-invocation: true
---
GitHub 이슈를 분석하고 수정하세요: $ARGUMENTS.

1. `gh issue view`로 이슈 세부 정보 가져오기
2. 관련 파일 코드베이스 검색
3. 수정 구현
4. 테스트 작성 및 실행
5. 커밋하고 PR 생성
```

호출: `/fix-issue 1234`

### Skill Frontmatter 전체 옵션

| 필드 | 필수 | 설명 |
|------|:---:|------|
| `name` | - | 슬래시 명령 이름 (생략 시 디렉토리 이름) |
| `description` | 권장 | Claude가 자동 호출 판단에 사용. **1,536자에서 잘림** |
| `when_to_use` | - | 추가 트리거 문구. description과 합쳐 1,536자 캡 |
| `argument-hint` | - | 자동완성 힌트 (예: `[filename] [format]`) |
| `arguments` | - | 명명 인수 리스트. `$이름` 으로 본문에서 치환 |
| `disable-model-invocation` | - | `true` 시 자동 호출 차단 (사용자만 트리거) |
| `user-invocable` | - | `false` 시 `/` 메뉴에서 숨김 (백그라운드 지식용) |
| `allowed-tools` | - | 권한 묻지 않고 쓸 도구 목록 |
| `model` | - | 스킬 실행 중 모델 오버라이드 (`inherit` 가능) |
| `effort` | - | 스킬 실행 중 노력 수준 오버라이드 |
| `context: fork` | - | 서브에이전트로 격리 실행 |
| `agent` | - | `context: fork` 시 사용할 서브에이전트 타입 |
| `hooks` | - | 스킬 라이프사이클에 한정된 훅 |
| `paths` | - | glob 패턴 매칭 시에만 자동 로드 (예: `src/**/*.ts`) |
| `shell` | - | `bash` (기본) 또는 `powershell` |

### 사용 가능한 문자열 치환

| 변수 | 의미 |
|------|------|
| `$ARGUMENTS` | 호출 시 인수 전체 |
| `$ARGUMENTS[N]` / `$N` | N번째 인수 (0-based) |
| `$이름` | frontmatter `arguments`에 선언된 명명 인수 |
| `${CLAUDE_SESSION_ID}` | 현재 세션 ID |
| `${CLAUDE_SKILL_DIR}` | 스킬 SKILL.md 위치 (보조 스크립트 참조용) |

### Skill Content 라이프사이클 (중요!)

```
스킬 호출 → SKILL.md 전체가 단일 메시지로 컨텍스트 진입
         → 세션 끝까지 유지됨
         → Claude는 이후 턴에서 SKILL.md 재읽기 안 함
```

> **함의**: 스킬 본문은 "이 작업 동안 계속 따라야 할 지침"으로 작성하세요. 일회성 단계 나열보다 표준 지침 형태가 효과적.

### 압축(compaction) 시 동작

자동 압축이 일어나도 **가장 최근에 호출된 스킬은 다시 첨부**됩니다.
- 스킬당 최초 5,000 토큰 보존
- 재첨부 스킬 합산 25,000 토큰 예산 (최신 호출부터 채움 — 오래된 스킬은 떨어져 나갈 수 있음)

### 번들 스킬 (Claude Code 내장)

| 스킬 | 역할 |
|------|------|
| `/simplify [focus]` | 변경 파일을 3개 에이전트가 병렬 리뷰 후 수정 |
| `/batch <지시>` | 대규모 변경을 5~30개 단위로 분해, worktree에서 병렬 실행 + PR 생성 |
| `/debug [설명]` | 디버그 로깅 활성화 + 세션 로그 분석 |
| `/loop [interval] [prompt]` | 프롬프트 주기적 반복 실행 (interval 생략 시 자체 페이싱) |
| `/claude-api [migrate]` | Claude API/Managed Agents 레퍼런스 로드 (모델 마이그레이션 자동화) |
| `/fewer-permission-prompts` | 트랜스크립트 분석 후 안전 명령 allowlist 자동 추가 |

> **Tip**: 부작용이 있는 Skill (배포, 외부 API 호출 등)에는 `disable-model-invocation: true` 설정하세요. 자동 호출 방지 + 컨텍스트 절약.

- [ ] `.claude/skills/` 또는 `~/.claude/skills/`에 간단한 Skill 하나 만들어보기
- [ ] `/skills` 로 사용 가능한 스킬 목록 확인 (`t` 키로 토큰 정렬)
- [ ] 번들 스킬 (`/simplify`, `/debug`) 한 번씩 실행해보기

---

## 5. Subagents (격리된 워커)

### 사용자 정의 Subagent 생성

`.claude/agents/` 디렉토리에 마크다운 파일 생성:

```markdown
# .claude/agents/security-reviewer.md
---
name: security-reviewer
description: 보안 취약점에 대한 코드 검토
tools: Read, Grep, Glob, Bash
model: opus
---
선임 보안 엔지니어로서 다음을 검토하세요:
- 주입 취약점 (SQL, XSS, 명령 주입)
- 인증/권한 부여 결함
- 코드 내 비밀 또는 자격 증명
- 안전하지 않은 데이터 처리
```

### Subagent 사용
```
"subagent를 사용하여 이 코드를 보안 문제에 대해 검토해"
```

### Subagent의 장점
```
메인 컨텍스트: [질문] → [Subagent 요약만 반환] → [계속 작업]
                          ↓
Subagent 컨텍스트: [파일1 읽기] → [파일2 읽기] → ... → [요약 생성]
                   (메인 컨텍스트에 영향 없음!)
```

### Subagent의 컨텍스트는 메인과 어떻게 다른가?

| 항목 | 메인 세션 | Subagent |
|------|----------|----------|
| 시스템 프롬프트 | 전체 | 더 짧음 (custom agent는 자체 정의) |
| CLAUDE.md | 로드 | **별도 로드** (subagent 컨텍스트에 카운트) |
| 자동 메모리 | 메인 MEMORY.md | **상속 안 됨** (`memory:` frontmatter 있으면 별도 MEMORY.md) |
| MCP 서버 + 스킬 | 모두 | 대부분 동일 |
| 부모 대화 기록 | - | **없음** (작업 지시만 받음) |
| Plan 모드 도구 | 사용 가능 | 제외 (재귀 방지) |
| 백그라운드 태스크 도구 | 사용 가능 | 제외 |
| Agent 도구 자체 | 사용 가능 | **기본적으로 제외** (재귀 방지) |
| 결과 반환 | - | **최종 텍스트 응답만** + 토큰 메타데이터 |

> **컨텍스트 절약 효과**: 위 docs 예시 — Subagent가 6,100토큰 분의 파일을 읽었지만, 메인은 420토큰 요약만 받음 = **약 14배 절약**.
>
> **Built-in Explore와 Plan 에이전트는 CLAUDE.md를 안 읽음** (더 작은 컨텍스트로 빠른 탐색).

### `isolation: worktree` — 안전한 실험

```yaml
# .claude/agents/experimental.md
---
name: experimental
isolation: worktree
description: 위험한 실험을 격리된 git worktree에서 실행
---
```

> 서브에이전트가 저장소의 격리된 복사본에서 작업 → 실패해도 메인 작업 영향 없음.

- [ ] `/agents`로 사용 가능한 Subagent 확인
- [ ] `.claude/agents/`에 커스텀 Subagent 하나 만들어보기
- [ ] "subagent를 사용하여 조사해" 패턴 사용해보기
- [ ] Built-in Explore agent로 광범위 탐색 위임해보기

---

## 6. MCP 서버 (외부 서비스 연결)

### MCP Transport 종류 (3가지)

| Transport | 용도 |
|-----------|------|
| `--transport http` | 원격 HTTP 서버 (가장 일반적) |
| `--transport sse` | 원격 SSE (Server-Sent Events) 서버 |
| `--transport stdio` | 로컬 프로세스 (npx 등으로 실행) |

### MCP 추가 방법
```bash
# 원격 HTTP (GitHub, Notion, Sentry 등)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 원격 SSE (Asana 등)
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 로컬 stdio (DB, 파일시스템)
claude mcp add --transport stdio db -- npx @bytebase/dbhub --dsn "postgresql://..."
```

### MCP 관리
```bash
claude mcp list    # 설치된 MCP 서버 확인
/mcp               # 세션 내에서 연결 상태 및 OAuth 관리
```

### MCP 범위
| 범위 | 설정 방법 | 저장 위치 |
|------|---------|----------|
| **로컬** (기본) | `--scope local` | 사용자별 (해당 프로젝트) |
| **프로젝트** (팀 공유) | `--scope project` | `.mcp.json` (git 체크인) |
| **사용자** (개인 전역) | `--scope user` | 모든 프로젝트에서 사용 |

### MCP 도구 이름 규칙

MCP 서버의 도구는 `mcp__<server>__<tool>` 형식으로 노출됩니다 (예: `mcp__github__create_issue`).
- Hook matcher에서 `mcp__github\.*` 처럼 정규식으로 필터링 가능
- Permission rule에서도 동일 패턴 사용

### MCP Prompts (슬래시 명령으로)

MCP 서버가 prompt를 제공하면 `/mcp__<server>__<prompt>` 형태로 자동 등록됩니다.

> **주의**: MCP 서버는 세션 시작 시 모든 도구 정의가 로드되어 컨텍스트를 소비합니다.
> 사용하지 않는 서버는 연결 해제하세요.

- [ ] `claude mcp list`로 현재 MCP 서버 확인
- [ ] `/mcp`로 서버별 토큰 비용 확인해보기

---

## 7. Hooks (결정론적 자동화)

### Hook 이벤트 (라이프사이클별)

**세션 단위 (Session)**
| 이벤트 | 타이밍 | matcher 값 |
|--------|--------|-----------|
| `SessionStart` | 세션 시작/재개 | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | 세션 종료 | (없음) |

**턴 단위 (Per-turn)**
| 이벤트 | 타이밍 | 활용 |
|--------|--------|------|
| `UserPromptSubmit` | 사용자 입력 직후, Claude 처리 전 | 프롬프트 검증/변환 |
| `Stop` | 턴 정상 완료 | 결과 검증, 알림 |
| `StopFailure` | API 에러로 턴 종료 | 에러 로깅 (output/exit code 무시됨) |

**도구 호출 단위 (Per-tool)**
| 이벤트 | 타이밍 | matcher |
|--------|--------|---------|
| `PreToolUse` | 도구 실행 전 | tool 이름 (`Bash`, `Edit\|Write`, `mcp__.*`) |
| `PostToolUse` | 도구 실행 성공 후 | tool 이름 |
| `PostToolUseFailure` | 도구 실행 실패 후 | tool 이름 |
| `PostToolBatch` | 병렬 도구 호출 묶음 후 | (matcher 없음) |
| `PermissionRequest` | 권한 요청 발생 시 | tool 이름 |
| `PermissionDenied` | auto 모드에서 차단 시 | tool 이름 |

**서브에이전트 / 백그라운드**
| 이벤트 | 타이밍 |
|--------|--------|
| `SubagentStart` / `SubagentStop` | 서브에이전트 시작/종료 |
| `TaskCreated` / `TaskCompleted` | 백그라운드 태스크 |
| `WorktreeCreate` / `WorktreeRemove` | git worktree 생성/제거 |

**비동기 / 기타**
| 이벤트 | 타이밍 |
|--------|--------|
| `PreCompact` / `PostCompact` | 컨텍스트 압축 전/후 |
| `Notification` | 알림 발생 (`permission_prompt`, `idle_prompt` 등) |
| `FileChanged` | 감시 중인 파일이 디스크에서 변경 |
| `CwdChanged` | 작업 디렉토리 변경 |
| `TeammateIdle` | 팀원 에이전트 idle 상태 |

### Hook Handler 종류 (5가지)

Hook은 단순 셸 명령만이 아닙니다. 다음 모두 가능:

| Type | 용도 |
|------|------|
| **Shell command** | 가장 일반적 (포맷터, 린터 등) |
| **HTTP endpoint** | 외부 서비스에 POST (Slack 알림, 로깅 등) |
| **MCP tool** | 등록된 MCP 도구 호출 |
| **Prompt** | Claude에게 추가 프롬프트 주입 |
| **Agent** | 서브에이전트 실행 |

### Hook 설정 예시 (`.claude/settings.json`)
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $FILE_PATH"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

### Hook 위치 (4단계)

| 위치 | 적용 범위 |
|------|---------|
| 관리형 (organization) | 조직 전체 |
| `~/.claude/settings.json` | 본인 (모든 프로젝트) |
| `.claude/settings.json` | 프로젝트 (팀 공유) |
| `.claude/settings.local.json` | 프로젝트 (개인) |
| 스킬 frontmatter `hooks:` | 스킬 라이프사이클 한정 |
| 서브에이전트 frontmatter `hooks:` | 해당 서브에이전트 한정 |

### CLAUDE.md 지침 vs Hook

| | CLAUDE.md 지침 | Hook |
|---|---|---|
| **실행** | 권고 (Claude가 판단) | 결정론적 (반드시 실행) |
| **예시** | "포맷팅 규칙을 따르세요" | 편집 후 prettier 자동 실행 |
| **사용** | 가이드라인 | 예외 없는 규칙 |

> **예외 없이 매번 실행해야 하면 → Hook**
> Claude가 상황에 따라 판단하면 → CLAUDE.md 지침

- [ ] `/hooks`로 현재 설정된 Hook 확인
- [ ] 알림 Hook 설정해보기 (Claude가 완료/입력 필요 시 알림)
- [ ] 위험 명령 차단 Hook 작성해보기 (`PreToolUse` + `Bash` matcher)

---

## 8. 컨텍스트 비용 이해하기

### 기능별 컨텍스트 비용

| 기능 | 로드 시기 | 컨텍스트 비용 |
|------|---------|-------------|
| **CLAUDE.md** | 세션 시작 | 모든 요청 (항상) |
| **Rules** (`paths:` 있음) | 파일 매칭 시 | 해당 파일 작업 시만 |
| **Skill (description)** | 세션 시작 | 낮음 (설명만 상시, 1,536자 캡) |
| **Skill (body)** | 호출 시 | 호출된 후 세션 끝까지 유지 |
| **MCP** | 세션 시작 | 모든 요청 (도구 정의 전체) |
| **Subagent** | 생성 시 | 메인에서 격리 (요약만 반환) |
| **Hook** | 트리거 시 | 0 (Claude 컨텍스트 외부에서 실행) |

> **압축(compaction) 시 토큰 예산**:
> - 가장 최근 호출된 스킬은 다시 첨부 (각 5,000 토큰까지)
> - 재첨부 합산 25,000 토큰 한도 (오래된 스킬은 떨어짐)
> - CLAUDE.md는 디스크에서 다시 로드되므로 보존됨

### 컨텍스트 최적화 전략

```
비용 높음 ← ─────────────────────────── → 비용 낮음

CLAUDE.md   MCP 도구   Skill 설명   Subagent   Hook
(항상 로드)  (항상 로드)  (상시 설명)   (격리됨)   (0 비용)
```

> **핵심**: 항상 필요한 것만 CLAUDE.md에, 가끔 필요한 것은 Skill에,
> 많은 탐색이 필요한 작업은 Subagent에, 자동화는 Hook에.

- [ ] `/context`로 현재 컨텍스트 사용량 확인
- [ ] 불필요한 MCP 서버 연결 해제하기

---

## 9. Plugins (모든 확장을 묶어서 배포)

### Plugin이란?

`.claude-plugin/plugin.json` 매니페스트가 있는 디렉토리. **여러 확장 기능을 하나의 패키지로 묶어** 팀/커뮤니티에 배포 가능.

### Plugin vs 단순 `.claude/` 설정 비교

| | `.claude/` 설정 | Plugin |
|---|----------------|--------|
| 위치 | 프로젝트 루트 | 별도 디렉토리 (분리 배포) |
| 호출 | `/command` | `/plugin-name:command` (네임스페이스) |
| 공유 방법 | git 체크인 | **마켓플레이스로 배포** |
| 버전 관리 | 프로젝트 git | 독립 버저닝 |
| 재사용 | 단일 프로젝트 | 여러 프로젝트 |

### Plugin 구조

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json       # 매니페스트 (이름/설명/버전)
├── commands/             # 슬래시 명령
├── agents/               # 서브에이전트
├── skills/               # 스킬 (각각 SKILL.md)
├── hooks/                # 훅 스크립트
├── bin/                  # 활성화 시 PATH에 추가될 실행 파일
└── settings.json         # 기본 설정 (agent, subagentStatusLine만 지원)
```

> ⚠️ **흔한 실수**: `commands/`, `agents/`, `skills/`, `hooks/` 를 `.claude-plugin/` 안에 넣지 말 것. 그것들은 **plugin 루트**에 있어야 함. `.claude-plugin/`에는 `plugin.json`만.

### plugin.json 기본 매니페스트

```json
{
  "name": "my-first-plugin",
  "version": "0.1.0",
  "description": "팀 공통 워크플로우 모음"
}
```

### 로컬 개발/테스트

```bash
# 로컬 디렉토리를 플러그인으로 로드 (개발 중)
claude --plugin-dir ./my-plugin

# 여러 개도 가능
claude --plugin-dir ./plugin-a --plugin-dir ./plugin-b
```

> 이미 설치된 마켓플레이스 플러그인과 같은 이름이면 **로컬 복사본이 우선** (재설치 없이 변경 테스트 가능).

### Plugin의 특별 기능

| 기능 | 설명 |
|------|------|
| **`bin/` 디렉토리** | 활성화 시 Bash 도구의 PATH에 추가됨 → CLI 도구 번들 가능 |
| **`settings.json`의 `agent`** | 플러그인의 커스텀 에이전트를 **메인 스레드로 활성화** → 시스템 프롬프트/도구 제한/모델까지 변경 |
| **LSP 서버 통합** | 플러그인에 LSP 서버 포함 가능 (TypeScript, Python, Rust 등은 공식 마켓플레이스에 사전 빌드 존재) |
| **Background monitors** | 플러그인이 백그라운드 모니터 추가 가능 |
| **버전 관리** | 마켓플레이스 통한 버전 핀, 업데이트 알림 |

### 관리 명령

```bash
/plugin                  # 플러그인 관리 메뉴
/reload-plugins          # 변경사항 재로드 (재시작 없이)
claude plugin install <name>@<marketplace>
```

### Plugin 사용 시기 결정

```
이 워크플로우를...
├─ 나만 쓴다 → ~/.claude/skills/
├─ 이 프로젝트 팀과 쓴다 → .claude/skills/ (git 체크인)
└─ 여러 팀/프로젝트/커뮤니티와 쓴다 → Plugin
```

- [ ] `/plugin`으로 플러그인 관리 메뉴 확인
- [ ] 공식 마켓플레이스에서 흥미로운 플러그인 1개 설치해보기
- [ ] 본인이 자주 쓰는 Skill을 플러그인으로 만들어보기

---

## 10. 기능 결합 패턴

| 패턴 | 작동 방식 | 예시 |
|------|---------|------|
| **Skill + MCP** | MCP가 연결, Skill이 사용법 가르침 | MCP=DB 연결, Skill=스키마/쿼리 패턴 |
| **Skill + Subagent** | Skill이 병렬 작업을 Subagent에 위임 | `/audit`이 보안/성능/스타일 agent 시작 |
| **CLAUDE.md + Skill** | 항상 규칙 + 온디맨드 참조 자료 | "API 규칙 따르세요" + 전체 API 가이드 |
| **Hook + MCP** | Hook이 MCP를 통해 외부 작업 트리거 | 중요 파일 편집 시 Slack 알림 |
| **Plugin + 다중 기능** | Plugin이 commands+skills+hooks 묶음 배포 | 팀 코드 리뷰 플러그인 (스킬+훅+에이전트) |

- [ ] 두 가지 이상의 기능을 결합하여 워크플로우 만들어보기

---

> **정리**: 확장 기능을 선택할 때의 핵심 질문
> 1. **항상 필요한가?** → CLAUDE.md / Rules
> 2. **가끔 필요한가?** → Skill
> 3. **외부 연결이 필요한가?** → MCP
> 4. **컨텍스트 격리가 필요한가?** → Subagent
> 5. **예외 없이 자동화해야 하는가?** → Hook
> 6. **팀/커뮤니티와 공유해야 하는가?** → **Plugin**
