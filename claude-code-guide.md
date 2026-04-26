# Claude Code 학습 가이드

> 시간날 때마다 돌아와서 하나씩 체크하며 학습합니다.
> Claude Code에게 "claude-code-guide.md 이어서 학습하자" 라고 말하면 됩니다.

---

## 학습 로드맵

이 가이드는 **빠른 실습(Phase 1~6)** + **심화 학습(Deep Dive 1~5)** 으로 구성됩니다.
Phase를 진행하면서 관련된 심화 가이드를 함께 읽으면 더 깊이 이해할 수 있습니다.

```
Phase 1  기본 사용법          ←─── Deep Dive 1. 작동 방식
Phase 2  CLAUDE.md 설정       ←─── Deep Dive 3. 지침 및 메모리
Phase 3  권한 및 보안
Phase 4  효율적인 작업 패턴    ←─── Deep Dive 4. 일반적인 워크플로우
                              ←─── Deep Dive 5. 모범 사례
Phase 5  커스터마이징          ←─── Deep Dive 2. 확장하기
Phase 6  고급 기능
```

### 심화 가이드 (별도 문서)

| # | 가이드 | 핵심 질문 | 연결 Phase | 원본 |
|---|--------|---------|-----------|------|
| 1 | [작동 방식](claude-code-how-it-works.md) | Claude Code는 내부적으로 어떻게 동작하는가? | Phase 1 | [공식](https://code.claude.com/docs/ko/how-claude-code-works) |
| 2 | [확장하기](claude-code-extensions.md) | Skill, Subagent, Hook, MCP은 언제 어떤 걸 쓰나? | Phase 5, 6 | [공식](https://code.claude.com/docs/ko/features-overview) |
| 3 | [지침 및 메모리](claude-code-memory.md) | CLAUDE.md와 자동 메모리를 어떻게 관리하나? | Phase 2 | [공식](https://code.claude.com/docs/ko/memory) |
| 4 | [일반적인 워크플로우](claude-code-workflows.md) | 실전에서 어떤 패턴으로 작업하나? | Phase 4 | [공식](https://code.claude.com/docs/ko/common-workflows) |
| 5 | [모범 사례](claude-code-best-practices.md) | 흔한 실수를 피하고 최대한 활용하려면? | 전체 | [공식](https://code.claude.com/docs/ko/best-practices) |

---

## Phase 1: 기본 사용법 익히기

> **심화**: [Deep Dive 1. 작동 방식](claude-code-how-it-works.md) — 에이전트 루프, 모델, 내장 도구, 컨텍스트 윈도우

### 1.1 CLI 기본 명령어
- [ ] `claude` — 대화형 세션 시작
- [ ] `claude "질문"` — 초기 프롬프트와 함께 시작
- [ ] `claude -c` — 가장 최근 대화 이어서 계속
- [ ] `claude -r "세션명" "질문"` — 특정 세션(ID/이름)을 이어서 시작 + 초기 프롬프트
- [ ] `claude -p "질문"` — 비대화형(headless) 모드 (결과만 출력, CI/CD에 유용)
- [ ] `cat file | claude -p "분석해줘"` — 파이프로 입력 전달
- [ ] `claude update` — 최신 버전으로 업데이트
- [ ] `claude install [version|stable|latest]` — 특정 버전 설치/재설치
- [ ] `claude auth login` / `auth logout` / `auth status` — 계정 인증 관리
- [ ] `claude agents` — 설정된 서브에이전트 목록 확인
- [ ] `claude setup-token` — CI/스크립트용 장기 OAuth 토큰 생성
- [ ] `claude --bare -p "질문"` — 최소 모드 (hooks/skills/MCP 등 자동 발견 생략, 빠른 실행)

### 1.2 세션 내 슬래시 명령어
- [ ] `/help` — 사용 가능한 명령어 확인
- [ ] `/clear` — 대화 초기화 (새 작업 시작 시). aliases: `/reset`, `/new`
- [ ] `/compact` — 컨텍스트 압축 (긴 작업 중 메모리 절약)
- [ ] `/compact 최근 파일 변경 내용에 집중` — 압축 시 유지할 내용 지정
- [ ] `/model` — 모델 전환 (sonnet, opus, haiku)
- [ ] `/usage` — 세션 비용/사용량/통계 (`/cost`, `/stats`는 alias)
- [ ] `/config` — 설정 메뉴 (alias: `/settings`)
- [ ] `/ide` — IDE 연결
- [ ] `/memory` — 자동 메모리 확인/편집
- [ ] `/init` — 프로젝트용 CLAUDE.md 자동 생성
- [ ] `/rename` — 현재 세션 이름 지정 (나중에 `-r`로 이어서 가능)
- [ ] `/btw` — 사이드 질문 (현재 컨텍스트에 영향 안 줌)
- [ ] `/context` — 컨텍스트 사용 현황 확인
- [ ] `/resume` — 세션 ID/이름으로 재개 (alias: `/continue`)
- [ ] `/rewind` — 코드/대화를 이전 시점으로 되돌리기 (aliases: `/checkpoint`, `/undo`)
- [ ] `/plan [설명]` — 즉시 plan 모드 진입 (선택적으로 작업 내용 전달)
- [ ] `/effort [low|medium|high|xhigh|max|auto]` — 모델 노력 수준 조정
- [ ] `/skills` — 사용 가능한 스킬 목록
- [ ] `/agents` — 서브에이전트 관리
- [ ] `/permissions` — 권한 규칙 관리 (alias: `/allowed-tools`)
- [ ] `/hooks` — Hook 설정 확인
- [ ] `/mcp` — MCP 서버 연결 관리
- [ ] `/diff` — 미커밋 변경 + 턴별 diff 뷰어
- [ ] `/doctor` — 설치/설정 진단 (`f` 키로 자동 수정)
- [ ] `/review [PR]` — PR 로컬 리뷰
- [ ] `/security-review` — 현재 브랜치 변경사항 보안 분석
- [ ] `/recap` — 현재 세션 한 줄 요약
- [ ] `/insights` — 세션 분석 리포트 생성
- [ ] `/tasks` — 백그라운드 작업 목록 (alias: `/bashes`)
- [ ] `/statusline` — 상태바 커스터마이징
- [ ] `/copy [N]` — 마지막(또는 N번째) 응답 클립보드 복사

### 1.3 키보드 단축키
- [ ] `Ctrl+C` — 현재 입력 또는 생성 중단
- [ ] `Ctrl+D` — 세션 종료
- [ ] `Ctrl+O` — Transcript 뷰어 토글 (사고 과정/도구 사용 상세)
- [ ] `Ctrl+G` — 프롬프트를 기본 텍스트 에디터에서 편집
- [ ] `Esc` + `Esc` — 이전 시점으로 되감기/대화 요약 (rewind)
- [ ] `Shift+Tab` — 권한 모드 순환 (`default` → `acceptEdits` → `plan` → 활성화된 모드)
- [ ] `Option+P` / `Alt+P` — 모델 빠른 전환
- [ ] `Option+T` / `Alt+T` — Extended Thinking 토글
- [ ] `\ + Enter` — 여러 줄 입력
- [ ] `Ctrl+B` — Bash 명령/에이전트를 백그라운드로 보내기 (tmux는 두 번)
- [ ] `Ctrl+T` — 태스크 리스트 보기/숨기기
- [ ] `Ctrl+R` — 명령어 히스토리 역방향 검색
- [ ] `Ctrl+L` — 입력란 비우고 화면 다시 그리기 (대화는 유지)
- [ ] `Ctrl+X Ctrl+K` — 모든 백그라운드 에이전트 종료 (3초 내 두 번 누르기)
- [ ] `Option+O` / `Alt+O` — Fast 모드 토글

### 1.4 빠른 입력
- [ ] `@파일명` — 파일/폴더 참조 (자동완성)
- [ ] `!명령어` — Bash 직접 실행 모드
- [ ] `/명령어` — 슬래시 명령어/스킬 실행
- [ ] 이미지 붙여넣기 (`Ctrl+V` / `Cmd+V`) — 스크린샷 분석

### 1.5 심화 학습 체크
- [ ] [작동 방식](claude-code-how-it-works.md) 1장: 에이전트 루프 3단계 (수집→수행→검증) 이해
- [ ] [작동 방식](claude-code-how-it-works.md) 2장: 모델별 차이 (Sonnet/Opus/Haiku) 이해
- [ ] [작동 방식](claude-code-how-it-works.md) 3장: 5가지 도구 범주 (파일/검색/실행/웹/코드 인텔리전스) 이해
- [ ] [작동 방식](claude-code-how-it-works.md) 6장: 컨텍스트 윈도우가 왜 가장 중요한 리소스인지 이해

---

## Phase 2: 프로젝트 설정 (CLAUDE.md)

> **심화**: [Deep Dive 3. 지침 및 메모리](claude-code-memory.md) — CLAUDE.md 작성법, 자동 메모리, Rules, 문제 해결

### 2.1 CLAUDE.md 기본
- [ ] `CLAUDE.md`란? — Claude가 매 세션 시작 시 읽는 프로젝트 지침서
- [ ] `/init` 실행하여 자동 생성해보기
- [ ] 직접 작성해보기: 빌드 명령, 테스트 명령, 코드 스타일

### 2.2 CLAUDE.md 위치별 역할
| 위치 | 범위 | 용도 |
|------|------|------|
| `./CLAUDE.md` | 프로젝트 (팀 공유) | 빌드/테스트/코드 규칙 |
| `.claude/CLAUDE.md` | 프로젝트 (팀 공유) | 동일 (폴더 정리용) |
| `~/.claude/CLAUDE.md` | 개인 (모든 프로젝트) | 개인 선호 설정 |

- [ ] 프로젝트용 CLAUDE.md 작성 (빌드, 테스트 명령 포함)
- [ ] 개인용 `~/.claude/CLAUDE.md` 작성 (응답 스타일, 언어 선호)

### 2.3 CLAUDE.md 작성 팁
```markdown
# 좋은 예
- `npm test` 로 테스트 실행
- 2-space 들여쓰기 사용
- 커밋 전 반드시 테스트 통과 확인

# 나쁜 예 (불필요한 내용)
- 깨끗한 코드를 작성하세요 (당연한 말)
- JavaScript에서 const를 사용하세요 (표준 관행)
```

- [ ] 프로젝트에 맞는 CLAUDE.md 작성 완료
- [ ] `@파일명` 문법으로 다른 문서 import 해보기: `# See @README.md`

### 2.4 Path-scoped 규칙
```
.claude/rules/api.md      ← API 관련 파일 편집 시 자동 로드
.claude/rules/frontend.md ← 프론트엔드 파일 편집 시 자동 로드
```

- [ ] `.claude/rules/` 디렉토리 생성
- [ ] 경로별 규칙 파일 작성해보기 (프론트매터에 `paths:` 지정)

### 2.5 심화 학습 체크
- [ ] [지침 및 메모리](claude-code-memory.md) 2장: CLAUDE.md 로드 순서 이해 (상위→현재→하위)
- [ ] [지침 및 메모리](claude-code-memory.md) 2.2장: 효과적인 지침 작성의 3원칙 (구체적/간결/일관적) 이해
- [ ] [지침 및 메모리](claude-code-memory.md) 3장: `.claude/rules/` 경로별 규칙과 glob 패턴 이해
- [ ] [지침 및 메모리](claude-code-memory.md) 4장: 자동 메모리 (MEMORY.md) 확인 및 편집해보기
- [ ] [지침 및 메모리](claude-code-memory.md) 5장: CLAUDE.md vs 자동 메모리 vs Skill 선택 기준 이해

---

## Phase 3: 권한 및 보안 설정

### 3.1 권한 모드
| 모드 | 자동 허용 범위 | 사용 시나리오 |
|------|-----------------|-------------|
| `default` | 사전 승인된 도구만 | 안전한 기본값 |
| `acceptEdits` | 읽기 + 파일 편집 + 일반 fs 명령(`mkdir`,`mv`,`cp` 등) | 코딩 작업 중 (편집 자동 승인) |
| `plan` | 읽기 전용 | 코드 탐색/리뷰 (변경 없이 계획만) |
| `auto` | 모든 작업 + 백그라운드 안전 분류기 | 장시간 작업, 프롬프트 피로 감소 |
| `dontAsk` | 사전 승인된 도구만 (그 외는 자동 거부) | 잠긴 CI/스크립트 (완전 비대화형) |
| `bypassPermissions` | 보호 경로 외 전부 | 격리된 컨테이너/VM에서만 사용 |

> `auto` 모드는 Sonnet 4.6/Opus 4.6/Opus 4.7에서만 사용 가능 (Haiku 미지원). 분류기가 3회 연속/누적 20회 차단하면 자동으로 default로 폴백.
> 어떤 모드든 [protected paths](https://code.claude.com/docs/en/permission-modes#protected-paths) 쓰기는 자동 승인되지 않음.

- [ ] `Shift+Tab`으로 모드 전환 해보기
- [ ] `/permissions` 명령어로 현재 설정 확인
- [ ] `claude --permission-mode plan` 으로 시작해보기

### 3.2 권한 규칙 설정
`.claude/settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git *)",
      "Read",
      "Edit(src/**/*.java)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Edit(.env*)"
    ]
  }
}
```

- [ ] `.claude/settings.json` 작성하여 프로젝트 공통 허용 명령 설정
- [ ] `.claude/settings.local.json` 작성하여 개인 설정 (gitignore에 추가)

---

## Phase 4: 효율적인 작업 패턴

> **심화**:
> - [Deep Dive 4. 일반적인 워크플로우](claude-code-workflows.md) — 코드 탐색, 버그 수정, 리팩토링, 테스트, PR, 세션 관리
> - [Deep Dive 5. 모범 사례](claude-code-best-practices.md) — 프롬프트 작성, 세션 관리, 실패 패턴 회피

### 4.1 에이전틱 작업 흐름
```
1. 탐색 → "이 프로젝트의 인증 구조를 분석해줘" (plan 모드)
2. 계획 → "OAuth 리팩토링 계획을 세워줘"
3. 구현 → "계획대로 구현해줘" (acceptEdits 모드)
4. 검증 → "테스트 실행하고 결과 확인해줘"
```

- [ ] plan 모드로 코드 분석 → 구현 모드로 전환하여 작업 해보기
- [ ] `Esc` 키로 중간에 멈추고 방향 수정해보기

### 4.2 컨텍스트 관리 전략
- [ ] 상태바에서 컨텍스트 사용률 확인하는 습관 들이기
- [ ] 작업 전환 시 `/clear` 사용
- [ ] 긴 작업 중 `/compact` 사용하여 압축
- [ ] 2번 수정해도 안 되면 `/clear` 후 더 나은 프롬프트로 재시작

### 4.3 Git 연동
- [ ] Claude에게 커밋 요청: "commit"
- [ ] Claude에게 PR 생성 요청: "PR 만들어줘"
- [ ] `claude --from-pr 123` — PR(GitHub/GitLab/Bitbucket URL 또는 번호) 기반으로 세션 시작
- [ ] worktree 사용: `claude -w feature-name` 또는 `--worktree` (병렬 작업, `<repo>/.claude/worktrees/<name>`에 생성)
- [ ] `/diff` — 미커밋 변경사항 + 턴별 diff 인터랙티브 뷰어
- [ ] `/review` 또는 `/security-review` — PR/현재 브랜치 변경 리뷰
- [ ] `/ultrareview` — 클라우드 기반 다중 에이전트 심층 코드 리뷰

### 4.4 비용 최적화
| 전략 | 절감 효과 |
|------|----------|
| 서브에이전트에 haiku 사용 | 50-70% |
| `/compact` 조기 실행 | 30-50% |
| CLAUDE.md 200줄 이내 유지 | 20-30% |
| 연관 없는 작업은 별도 세션 | 30-40% |

- [ ] `/usage` (구 `/cost`) 로 비용 확인해보기
- [ ] `claude -p --max-budget-usd 10.00 "..."` 옵션으로 예산 제한 설정해보기 (print 모드 전용)

### 4.5 심화 학습 체크
- [ ] [워크플로우](claude-code-workflows.md) 1장: 새 코드베이스 이해하기 패턴 실습
- [ ] [워크플로우](claude-code-workflows.md) 2장: 버그 수정 프롬프트 패턴 실습
- [ ] [워크플로우](claude-code-workflows.md) 4장: Plan Mode 워크플로우 실습
- [ ] [워크플로우](claude-code-workflows.md) 9장: Git Worktree로 병렬 세션 실습
- [ ] [모범 사례](claude-code-best-practices.md) 1장: 검증 수단 제공 패턴 연습
- [ ] [모범 사례](claude-code-best-practices.md) 2장: 탐색→계획→구현 3단계 실습
- [ ] [모범 사례](claude-code-best-practices.md) 3장: 4가지 프롬프트 전략 연습
- [ ] [모범 사례](claude-code-best-practices.md) 8장: 5가지 실패 패턴 인식하기

---

## Phase 5: 커스터마이징

> **심화**: [Deep Dive 2. 확장하기](claude-code-extensions.md) — 6가지 확장 기능 비교, 선택 가이드, 컨텍스트 비용

### 5.1 커스텀 스킬 (Slash Commands)

> **2026 변경**: `.claude/commands/`(구 Custom Commands)는 **Skills로 통합**되었습니다.
> `.claude/commands/deploy.md` 와 `.claude/skills/deploy/SKILL.md` 는 모두 `/deploy` 로 동작.
> 기존 `commands/` 파일은 그대로 동작하며, Skills는 추가 기능(보조 파일, 자동 호출, frontmatter 옵션) 제공.

`.claude/skills/deploy/SKILL.md`:
```yaml
---
name: deploy
description: 배포 프로세스 실행
argument-hint: [environment]
disable-model-invocation: true   # 사용자가 직접 호출할 때만 (자동 호출 차단)
---
$ARGUMENTS 환경에 배포합니다.
1. 테스트 실행
2. 빌드
3. 배포 스크립트 실행
```

**주요 frontmatter 필드** (전체 [공식 docs](https://code.claude.com/docs/en/skills#frontmatter-reference) 참고):

| 필드 | 용도 |
|------|------|
| `name` | 슬래시 명령어 이름 (생략 시 디렉토리 이름 사용) |
| `description` | 자동 호출 판단용 설명 (1,536자 캡) |
| `argument-hint` | 자동완성 시 표시할 인수 힌트 |
| `arguments` | 명명 인수 (`$이름` 치환) |
| `disable-model-invocation` | `true` 시 자동 호출 차단 (`/deploy`처럼 사용자가 트리거) |
| `user-invocable` | `false` 시 `/` 메뉴에서 숨김 (백그라운드 지식용) |
| `allowed-tools` | 권한 없이 사용 가능한 도구 |
| `model` / `effort` | 스킬 실행 중 모델/노력 수준 오버라이드 |
| `context: fork` | 서브에이전트로 격리 실행 |
| `paths` | glob 패턴과 일치하는 파일 작업 시에만 자동 로드 |
| `hooks` | 스킬 라이프사이클에 한정된 훅 |

**번들 스킬** (Claude Code 내장, 별도 설치 없이 사용 가능):
- `/simplify [focus]` — 변경된 파일을 3개 에이전트가 병렬 리뷰 후 수정
- `/batch <지시>` — 대규모 변경을 5~30개 단위로 분해해 병렬 실행 (worktree + PR)
- `/debug [설명]` — 디버그 로깅 활성화 + 세션 로그 분석
- `/loop [interval] [prompt]` — 프롬프트를 주기적으로 반복 실행
- `/claude-api [migrate|managed-agents-onboard]` — Claude API/Managed Agents 레퍼런스 로드

- [ ] `.claude/skills/` 디렉토리에 프로젝트 맞춤 스킬 생성
- [ ] `/deploy staging` 처럼 인수와 함께 실행해보기
- [ ] `/skills` 명령으로 사용 가능한 스킬 목록 확인 (`t` 키로 토큰 정렬)

### 5.2 커스텀 서브에이전트
`.claude/agents/code-reviewer.md`:
```yaml
---
name: code-reviewer
description: 코드 변경 시 품질/보안 리뷰 수행
tools: Read, Grep, Glob
model: haiku
---
코드 리뷰어입니다. 변경된 코드를 분석하여:
- 보안 취약점
- 성능 문제
- 코드 품질
을 리포트합니다.
```

- [ ] 읽기 전용 리뷰 에이전트 만들어보기
- [ ] `model: haiku` 설정으로 비용 절약형 에이전트 만들기
- [ ] `/agents` 명령어로 에이전트 목록 확인

### 5.3 Hooks (자동화)
`.claude/settings.json`:
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

| Hook 이벤트 | 타이밍 | 활용 |
|-------------|--------|------|
| `SessionStart` | 세션 시작/재개 시 | 환경 설정, 컨텍스트 주입 |
| `SessionEnd` | 세션 종료 시 | 로그 저장, 정리 |
| `UserPromptSubmit` | 사용자 입력 제출 직후 | 프롬프트 검증/변환 |
| `PreToolUse` | 도구 실행 전 | 위험 명령 차단 |
| `PostToolUse` | 도구 실행 성공 후 | 자동 포맷팅 |
| `PostToolUseFailure` | 도구 실행 실패 후 | 에러 처리/복구 |
| `Stop` | 턴 정상 완료 시 | 결과 검증 |
| `StopFailure` | API 에러로 턴 종료 시 | 에러 로깅 |
| `SubagentStop` | 서브에이전트 종료 시 | 서브에이전트별 후처리 |
| `FileChanged` | 감시 중인 파일이 디스크에서 변경 시 | 자동 재실행/검증 |
| `Notification` | 알림 발생 시 | 데스크톱 알림 |
| `PreCompact` / `PostCompact` | 컨텍스트 압축 전/후 | 압축 가이드 |

> Hook handler는 셸 명령, HTTP 엔드포인트, MCP 도구, 프롬프트, 에이전트 모두 가능.

- [ ] 파일 저장 후 자동 포맷팅 Hook 설정
- [ ] `/hooks` 명령어로 설정된 Hook 확인

### 5.4 심화 학습 체크
- [ ] [확장하기](claude-code-extensions.md) 2장: CLAUDE.md vs Skill vs Rules 선택 기준 이해
- [ ] [확장하기](claude-code-extensions.md) 4장: Skill 생성 및 frontmatter 옵션 이해
- [ ] [확장하기](claude-code-extensions.md) 5장: Subagent 격리 장점 이해
- [ ] [확장하기](claude-code-extensions.md) 7장: Hook vs CLAUDE.md 지침 차이 (결정론적 vs 권고)
- [ ] [확장하기](claude-code-extensions.md) 8장: 기능별 컨텍스트 비용 이해

---

## Phase 6: 고급 기능

### 6.1 MCP (Model Context Protocol) 서버
외부 도구/서비스와 Claude를 연결합니다. 3가지 전송 방식: `http`, `sse`, `stdio`.

```bash
# 원격 HTTP 서버 (GitHub, Notion 등)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 원격 SSE 서버 (Asana 등)
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 로컬 stdio 서버 (DB, 파일시스템 등)
claude mcp add --transport stdio db -- npx @bytebase/dbhub --dsn "postgresql://..."
```

- [ ] MCP 개념 이해: 외부 서비스의 도구를 Claude에 제공 (modelcontextprotocol.io)
- [ ] `claude mcp list` — 설치된 MCP 서버 확인
- [ ] `/mcp` — 세션 내에서 MCP 연결/OAuth 관리
- [ ] 필요한 MCP 서버 하나 설치해보기
- [ ] scope 이해: `--scope local` (개인 기본), `--scope project` (팀 공유, `.mcp.json`), `--scope user` (개인 전역)
- [ ] MCP 도구는 `mcp__<server>__<tool>` 이름 패턴 (Hook matcher에서 활용)

### 6.2 Headless 모드 (CI/CD 연동)
```bash
# GitHub Actions에서 코드 리뷰 자동화
claude -p "최근 변경사항 리뷰" --output-format json > review.json

# Pre-commit hook
claude -p "스테이징된 변경사항에 문제 없는지 확인" \
  --allowedTools "Bash(git diff HEAD)"
```

- [ ] `claude -p` 로 비대화형 실행해보기
- [ ] `--output-format json` 으로 구조화된 출력 받아보기

### 6.3 Extended Thinking (깊은 추론)
```bash
# 복잡한 문제에 더 깊이 생각하게 하기
/effort max

# 특정 질문에만 깊이 생각하게 하기
"ultrathink: 이 아키텍처의 트레이드오프를 분석해줘"

# 항상 thinking 활성화
# settings.json: { "alwaysThinkingEnabled": true }
```

- [ ] `Option+T` 로 Thinking 토글해보기
- [ ] `Ctrl+O` 로 사고 과정(Transcript 뷰어) 확인해보기
- [ ] `/effort` 레벨 조절해보기 (`low`/`medium`/`high`/`xhigh`/`max`/`auto`) — 모델별로 사용 가능 레벨 다름, `max`는 세션 한정
- [ ] `claude --effort high "..."` 로 시작 시 노력 레벨 지정

### 6.4 프로젝트 디렉토리 구조 정리
```
my-project/
├── CLAUDE.md                     # 프로젝트 지침
├── .claude/
│   ├── CLAUDE.md                 # 프로젝트 지침 (대안 위치)
│   ├── settings.json             # 공유 설정 (git에 포함)
│   ├── settings.local.json       # 개인 설정 (gitignore)
│   ├── rules/                    # 경로별 규칙
│   │   ├── api.md
│   │   └── frontend.md
│   ├── agents/                   # 커스텀 에이전트
│   │   └── code-reviewer.md
│   ├── skills/                   # 커스텀 스킬
│   │   └── deploy/SKILL.md
│   └── hooks/                    # Hook 스크립트
│       └── validate.sh
└── .gitignore
    # .claude/settings.local.json
    # .claude/worktrees/
```

- [ ] 현재 프로젝트에 `.claude/` 구조 세팅해보기
- [ ] `.gitignore`에 로컬 설정 파일 추가

### 6.5 심화 학습 체크
- [ ] [워크플로우](claude-code-workflows.md) 11장: Extended Thinking 구성 방법 이해
- [ ] [워크플로우](claude-code-workflows.md) 13장: Unix 유틸리티로 사용하기 (파이프, CI/CD)
- [ ] [모범 사례](claude-code-best-practices.md) 7장: 병렬 세션 & fan-out 자동화 이해

---

## 실전 팁 모음

### 프롬프트 작성 팁
| 나쁜 예 | 좋은 예 |
|---------|---------|
| "코드 고쳐줘" | "UserService.java의 NPE를 수정해줘. null 체크 추가" |
| "리팩토링해줘" | "AuthController의 인증 로직을 별도 서비스로 분리해줘" |
| "테스트 추가해줘" | "UserService.createUser()에 대한 단위 테스트를 JUnit5로 작성해줘" |

### 작업별 추천 모드
| 작업 | 모델 | 권한 모드 |
|------|------|----------|
| 코드 분석/이해 | sonnet | plan |
| 버그 수정 | sonnet | acceptEdits |
| 복잡한 설계 | opus | plan → default |
| 대량 리팩토링 | sonnet | acceptEdits 또는 auto |
| 빠른 수정 | haiku | acceptEdits |
| 장시간 자율 작업 | sonnet/opus | auto (안전 분류기 적용) |
| CI/스크립트 | sonnet | dontAsk + 명시적 allow rules |

### 문제 해결
| 상황 | 해결 |
|------|------|
| Claude가 계속 틀림 | `/clear` 후 더 구체적인 프롬프트로 재시작 |
| 컨텍스트 부족 | `/compact` 실행 후 핵심 내용 재전달 |
| 권한 문제 | `/permissions` 에서 설정 확인/수정 |
| 세션 디버깅 | `claude --debug` 또는 `Ctrl+O` |
| 이전 작업 이어가기 | `claude -c` 또는 `claude -r "세션명"` |

---

## 학습 진행 체크리스트

### 입문 (일상 업무에 충분)
- [ ] Phase 1 완료: CLI, 슬래시 명령어, 키보드 단축키
- [ ] Phase 2 완료: CLAUDE.md 작성 및 설정
- [ ] Deep Dive 1 읽기: [작동 방식](claude-code-how-it-works.md)
- [ ] Deep Dive 3 읽기: [지침 및 메모리](claude-code-memory.md)

### 중급 (효율 향상)
- [ ] Phase 3 완료: 권한 및 보안 설정
- [ ] Phase 4 완료: 에이전틱 작업, 컨텍스트 관리, Git 연동
- [ ] Deep Dive 4 읽기: [일반적인 워크플로우](claude-code-workflows.md)
- [ ] Deep Dive 5 읽기: [모범 사례](claude-code-best-practices.md)

### 고급 (자동화 & 확장)
- [ ] Phase 5 완료: Skills, Subagents, Hooks 커스터마이징
- [ ] Phase 6 완료: MCP, Headless, Extended Thinking
- [ ] Deep Dive 2 읽기: [확장하기](claude-code-extensions.md)

---

> **학습 방법**:
> 1. Phase를 순서대로 진행하며 체크박스를 체크합니다.
> 2. 각 Phase 끝의 "심화 학습 체크"에서 관련 Deep Dive 문서를 읽습니다.
> 3. 마지막 "학습 진행 체크리스트"로 전체 진행 상황을 확인합니다.
> 4. 시간날 때 "claude-code-guide.md 이어서 학습하자"로 돌아오세요.
