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

### Skill 생성 방법

`.claude/skills/` 디렉토리에 `SKILL.md` 파일 생성:

```markdown
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: GitHub 이슈 분석 및 수정
disable-model-invocation: true
---
GitHub 이슈를 분석하고 수정하세요: $ARGUMENTS.

1. `gh issue view`로 이슈 세부 정보 가져오기
2. 관련 파일 코드베이스 검색
3. 수정 구현
4. 테스트 작성 및 실행
5. 커밋하고 PR 생성
```

### Skill 호출
```bash
/fix-issue 1234
```

### Skill Frontmatter 옵션

| 필드 | 설명 |
|------|------|
| `name` | Skill 이름 (호출 시 사용) |
| `description` | Claude가 관련성 판단에 사용하는 설명 |
| `disable-model-invocation: true` | 수동 호출만 (자동 로드 안 함) |
| `context: fork` | 격리된 컨텍스트에서 실행 |
| `argument-hint` | 인수 힌트 표시 |

> **Tip**: 부작용이 있는 Skill에는 `disable-model-invocation: true` 설정하세요.
> 컨텍스트도 절약되고, 사용자만 트리거할 수 있습니다.

- [ ] `.claude/skills/` 디렉토리에 간단한 Skill 하나 만들어보기
- [ ] `/skill명 인수`로 실행해보기

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

- [ ] `/agents`로 사용 가능한 Subagent 확인
- [ ] `.claude/agents/`에 커스텀 Subagent 하나 만들어보기
- [ ] "subagent를 사용하여 조사해" 패턴 사용해보기

---

## 6. MCP 서버 (외부 서비스 연결)

### MCP 추가 방법
```bash
# GitHub 연동
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 데이터베이스 연동
claude mcp add --transport stdio db -- npx @bytebase/dbhub --dsn "postgresql://..."
```

### MCP 관리
```bash
claude mcp list    # 설치된 MCP 서버 확인
/mcp               # 세션 내에서 연결 상태 및 토큰 비용 확인
```

### MCP 범위
| 범위 | 설정 방법 |
|------|---------|
| **로컬** (개인/프로젝트) | `--scope local` |
| **프로젝트** (팀 공유) | `--scope project` |
| **사용자** (모든 프로젝트) | `--scope user` |

> **주의**: MCP 서버는 세션 시작 시 모든 도구 정의가 로드되어 컨텍스트를 소비합니다.
> 사용하지 않는 서버는 연결 해제하세요.

- [ ] `claude mcp list`로 현재 MCP 서버 확인
- [ ] `/mcp`로 서버별 토큰 비용 확인해보기

---

## 7. Hooks (결정론적 자동화)

### Hook 이벤트

| 이벤트 | 타이밍 | 활용 |
|--------|--------|------|
| `PreToolUse` | 도구 실행 전 | 위험 명령 차단, 입력 검증 |
| `PostToolUse` | 도구 실행 후 | 자동 포맷팅, 린팅 |
| `Stop` | 작업 완료 시 | 결과 검증, 알림 |
| `Notification` | 입력 필요 시 | 데스크톱 알림 |
| `InstructionsLoaded` | 지침 로드 시 | 디버깅, 로깅 |

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
    ]
  }
}
```

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

---

## 8. 컨텍스트 비용 이해하기

### 기능별 컨텍스트 비용

| 기능 | 로드 시기 | 컨텍스트 비용 |
|------|---------|-------------|
| **CLAUDE.md** | 세션 시작 | 모든 요청 (항상) |
| **Rules** (`paths:` 있음) | 파일 매칭 시 | 해당 파일 작업 시만 |
| **Skill** | 사용 시 | 낮음 (설명만 상시 로드) |
| **MCP** | 세션 시작 | 모든 요청 (도구 정의) |
| **Subagent** | 생성 시 | 메인에서 격리 |
| **Hook** | 트리거 시 | 0 |

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

## 9. 기능 결합 패턴

| 패턴 | 작동 방식 | 예시 |
|------|---------|------|
| **Skill + MCP** | MCP가 연결, Skill이 사용법 가르침 | MCP=DB 연결, Skill=스키마/쿼리 패턴 |
| **Skill + Subagent** | Skill이 병렬 작업을 Subagent에 위임 | `/audit`이 보안/성능/스타일 agent 시작 |
| **CLAUDE.md + Skill** | 항상 규칙 + 온디맨드 참조 자료 | "API 규칙 따르세요" + 전체 API 가이드 |
| **Hook + MCP** | Hook이 MCP를 통해 외부 작업 트리거 | 중요 파일 편집 시 Slack 알림 |

- [ ] 두 가지 이상의 기능을 결합하여 워크플로우 만들어보기

---

> **정리**: 확장 기능을 선택할 때의 핵심 질문
> 1. **항상 필요한가?** → CLAUDE.md / Rules
> 2. **가끔 필요한가?** → Skill
> 3. **외부 연결이 필요한가?** → MCP
> 4. **컨텍스트 격리가 필요한가?** → Subagent
> 5. **예외 없이 자동화해야 하는가?** → Hook
> 6. **팀/커뮤니티와 공유해야 하는가?** → Plugin
