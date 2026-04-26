# Claude Code 핵심 인사이트 모음

> 공식 문서를 다시 읽어도 한 번에 파악하기 어려운, **알고 있으면 큰 차이를 만드는** 인사이트들을 정리합니다.
> Phase별 학습 가이드(`claude-code-guide.md`)와 Deep Dive 문서들을 보완하는 메타 정리입니다.

---

## 🧭 인사이트 11선

### 1. Custom Commands → Skills 통합 (가장 큰 구조 변화)

**과거**: Custom Commands(`/명령`)와 Skills(자동 호출 지식)가 별개의 개념
**지금**: 하나로 통합. frontmatter로 역할 결정

| Frontmatter | 옛 명칭 | 동작 |
|------------|---------|------|
| `disable-model-invocation: true` | "Custom Command" | 사용자만 트리거 |
| `user-invocable: false` | "Skill" | Claude만 자동 호출 |
| 둘 다 미설정 | (혼합) | 양쪽 모두 가능 |

**실용 팁**: 새로 만들 땐 기본값(둘 다 미설정)으로 두고, 부작용 있는 작업(배포, 외부 API 호출)에만 `disable-model-invocation: true` 켜라.

---

### 2. Skill 본문은 "한 번 로드되면 끝까지 살아 있다"

```
스킬 호출 → SKILL.md 전체가 단일 메시지로 컨텍스트 진입
         → 세션 끝까지 유지
         → Claude는 이후 턴에 SKILL.md 재읽기 안 함
```

**함의**:
- 스킬 본문은 **"이 작업 동안 계속 따라야 할 표준 지침"** 형태로 작성
- "1단계 → 2단계 → 3단계" 같은 일회성 절차보다, **항상 적용되는 규칙** 형태가 효과적
- 압축 시에도 최근 호출된 스킬은 **5,000 토큰까지 보존** (총 25,000 토큰 예산)

---

### 3. `/compact` 후 무엇이 살아남는가 — 정확한 매트릭스

| 메커니즘 | 압축 후 |
|----------|--------|
| 시스템 프롬프트 | ✓ 자동 재로드 |
| 자동 메모리 (`MEMORY.md`) | ✓ 자동 재로드 |
| MCP 도구 정의 | ✓ 자동 재로드 |
| **루트 `CLAUDE.md`, `paths:` 없는 rules** | ✓ 디스크에서 재주입 |
| **호출된 Skill 본문** | ✓ 재주입 (스킬당 5K, 합계 25K 토큰 캡, 오래된 것 먼저 드롭) |
| **하위 폴더 `CLAUDE.md`** | ⚠️ 사라짐 (해당 폴더 파일 다시 읽힐 때까지) |
| **`paths:` 있는 Rules** | ⚠️ 사라짐 (트리거 파일 다시 읽으면 재로드) |
| **Skill 인덱스 (description 목록)** | ⚠️ 재주입 안 됨! 호출했던 스킬만 살아남음 |
| 대화 기록 | 구조화된 요약으로 대체 |

**숨은 함정**:
- 압축 후엔 **호출 안 한 스킬 description이 사라짐** → Claude가 자동 호출 못하게 됨
- **하위 폴더 CLAUDE.md는 사라짐** → 중요한 지침은 루트 CLAUDE.md에 두거나 `paths:` 없이 작성
- Skill 본문은 절단 시 **앞부분이 보존됨** → 가장 중요한 지침은 SKILL.md 위쪽에 배치!

---

### 4. Subagent의 컨텍스트는 메인과 어떻게 다른가

| 항목 | 메인 세션 | Subagent |
|------|----------|----------|
| 시스템 프롬프트 | 전체 | 더 짧음 (custom agent는 자체 정의) |
| CLAUDE.md | 로드 | **별도 로드** (subagent 컨텍스트에 카운트) |
| 자동 메모리 | 메인 MEMORY.md | **상속 안 됨** (`memory:` 있으면 별도 MEMORY.md) |
| 부모 대화 기록 | - | **없음** (작업 지시만 받음) |
| Plan 모드 도구 | 사용 가능 | 제외 |
| 백그라운드 태스크 도구 | 사용 가능 | 제외 |
| Agent 도구 자체 | 사용 가능 | **기본적으로 제외** (재귀 방지) |
| 결과 반환 | - | **최종 텍스트 응답만** + 토큰 메타데이터 |

**컨텍스트 절약 효과**: 공식 docs 예시 — Subagent가 6,100토큰 파일 읽음 → 메인은 420토큰 요약만 받음 = **약 14배 절약**

**Built-in Explore와 Plan 에이전트는 CLAUDE.md조차 안 읽음** (더 작은 컨텍스트로 빠른 탐색).

**함의**: subagent에게 작업을 위임할 때 **메인 세션에서 알게 된 맥락은 별도로 prompt에 넣어줘야 함**. "지금까지 작업 내용을 알아서 가져가" 같은 가정은 작동 안 함.

---

### 5. Hooks가 "셸 스크립트 트리거" → "이벤트 워크플로우 엔진"으로 진화

**Hook handler 5종류**:
| Type | 용도 |
|------|------|
| Shell command | 가장 일반적 (포맷터, 린터) |
| HTTP endpoint | 외부 서비스에 POST (Slack, 로깅) |
| MCP tool | 등록된 MCP 도구 호출 |
| Prompt | Claude에게 추가 프롬프트 주입 |
| Agent | 서브에이전트 실행 |

**활용 예**:
- `UserPromptSubmit` + Prompt handler → 모든 사용자 입력에 자동 컨텍스트 주입
- `PreCompact` + Agent handler → 압축 직전 서브에이전트로 핵심 정보 보존
- `FileChanged` + MCP tool → 특정 파일 변경 시 Slack 알림

---

### 6. Hook Exit Code 2 — "차단"이 아닌 "Claude 가르치기" 채널

| Exit Code | 동작 |
|-----------|------|
| `0` + stdout | **Claude 안 봄** (debug log만) |
| `0` + `additionalContext` JSON | 컨텍스트에 정상 주입 |
| **`2` + stderr** | **stderr이 Claude에게 직접 전달** (프롬프트 일부처럼) |

**활용 예**:
```bash
# PreToolUse hook
if echo "$TOOL_INPUT" | grep -q "rm -rf /"; then
  echo "이 명령은 너무 위험합니다. 대신 명시적 경로 지정을 고려하세요." >&2
  exit 2
fi
```
Claude는 이 메시지를 받아 **다른 접근을 시도**합니다. "차단" 대신 "지도"로 작용.

---

### 7. `auto` 모드 = "사용자 신뢰 + 분류기 안전망" 패러다임

```
default            : 모든 행동 사용자 승인 (피로 누적)
bypassPermissions  : 모두 허용 (위험)
auto               : 분류기가 위험만 차단, 나머진 자동 진행 ⭐
```

- 차단 횟수 임계치(연속 3회 / 누적 20회) 초과 시 자동 default 폴백
- `claude auto-mode defaults` 로 분류기 룰 확인
- Sonnet 4.6 / Opus 4.6 / Opus 4.7만 지원 (Haiku 미지원)
- 장시간 작업 (`/loop`, `/batch`) 과 결합하면 위력적

---

### 8. Plugins = "확장의 패키지 매니저"

```
.claude/ 직접 설정     : 단일 프로젝트만 적용
Plugin                 : 여러 프로젝트에 재사용 + 마켓플레이스 배포
```

**Plugin 구조**:
```
my-plugin/
├── .claude-plugin/plugin.json   # 매니페스트 (이름/버전/설명)
├── commands/  agents/  skills/  hooks/
├── bin/                          # 활성화 시 PATH에 추가
└── settings.json                 # agent, subagentStatusLine 키 지원
```

**숨은 강력 기능**:
- `bin/` → 활성화 시 PATH에 추가 → **CLI 도구도 번들** 가능
- `settings.json`의 `agent` 키 → 플러그인의 에이전트를 **메인 스레드로 활성화** → 시스템 프롬프트/도구/모델 변경 가능
- 호출 시 `/plugin-name:command` 네임스페이스로 충돌 방지

**Plugin 만들 시기**: 본인이 자주 쓰는 워크플로우가 **여러 프로젝트에서 반복**되거나 **팀에 공유**할 가치가 있으면 승격.

---

### 9. 컨텍스트 비용 계층 — 한 번 정리해두면 평생 유용

```
[항상 로드 — 비싸다]      CLAUDE.md, MCP 도구 정의
                          ↓
[조건부 로드 — 보통]      paths: 있는 Rules, Skill description
                          ↓
[호출 시만 — 효율적]     Skill body, Subagent (요약만 반환)
                          ↓
[비용 0 — 외부 실행]     Hook (Claude 컨텍스트 외부)
```

**3가지 법칙**:
1. **CLAUDE.md ≤ 200줄** — 넘치면 Rules 또는 Skill로 이동
2. **사용 안 하는 MCP 서버는 연결 해제** — 도구 정의가 모든 요청에 로드됨
3. **많은 파일 탐색이 필요하면 Subagent로 격리** — 메인 컨텍스트 보호

---

### 10. `!` 명령어 — 권한 승인 없이 Claude grounding

```
!git log --oneline -10
이 커밋들 중 인증 관련된 것만 골라줘
```

- `!`로 시작한 줄은 **사용자 셸에서 직접 실행** (Claude가 실행 안 함)
- 출력이 다음 메시지 앞에 자동 prefix
- **권한 승인 불필요** — 사용자가 직접 실행한 것이므로
- Claude가 명령 결과를 사실로 받아들이게 만드는 가장 빠른 방법

**언제 유용한가**:
- "현재 git 상태 알아둔 채로 작업해줘" — `!git status` 후 질문
- "이 에러 로그 보고 분석해줘" — `!cat error.log` 후 질문
- 파이프(`cat ... | claude -p`)와 비슷하지만 **인터랙티브 세션 안에서** 자연스럽게 활용

---

### 11. 자주 놓치는 "숨은 보석" 기능들

| 기능 | 효과 |
|------|------|
| `<!-- 주석 -->` in CLAUDE.md | **컨텍스트 진입 전 제거됨** → 사람용 메모를 토큰 낭비 없이 작성 |
| `CLAUDE.local.md` | 개인용 비공개 지침 (gitignore 자동). 같은 폴더 CLAUDE.md보다 **뒤에 추가**되어 충돌 시 우선 |
| `--exclude-dynamic-system-prompt-sections` | 머신별 정보를 시스템 프롬프트에서 분리 → **여러 사용자가 prompt cache 공유** (CI 비용 절감) |
| `isolation: worktree` | 서브에이전트가 저장소의 격리된 복사본에서 작업 → 위험한 실험 안전 |
| `/fewer-permission-prompts` | 트랜스크립트 분석으로 자주 승인한 명령 자동 allowlist 추가 |
| Live change detection | 기존 skills 디렉토리에 SKILL.md 추가/수정은 **세션 재시작 없이** 즉시 반영 |
| `--bare` 모드 | hooks/skills/MCP/auto memory/CLAUDE.md 자동 발견 모두 건너뛰고 빠른 시작 |
| `/rewind` | 모든 파일 편집은 자동 체크포인트. 코드만/대화만/둘 다 선택 복원 → **과감한 시도가 가능** |

---

## 🎯 한 문장 요약

> **Claude Code의 가장 중요한 자원은 컨텍스트 윈도우다.**
> CLAUDE.md, Skill, Subagent, Hook, Plugin은 모두 **컨텍스트를 어떻게 채우고 어떻게 비울지**의 다른 답이다.
> 무엇이 항상 로드되고, 무엇이 압축에서 살아남고, 무엇이 0비용으로 실행되는지를 이해하면 — 같은 작업을 5배 빠르게, 10배 싸게 할 수 있다.

---

## 📚 관련 문서

- [메인 학습 가이드](claude-code-guide.md) — Phase 1~6 실습 체크리스트
- [작동 방식](claude-code-how-it-works.md) — 에이전트 루프, 컨텍스트 윈도우, 권한
- [확장하기](claude-code-extensions.md) — Skills, Subagents, Hooks, MCP, Plugins
- [지침 및 메모리](claude-code-memory.md) — CLAUDE.md, 자동 메모리, Rules
- [워크플로우](claude-code-workflows.md) — 실전 패턴
- [모범 사례](claude-code-best-practices.md) — 5대 핵심 + 최신 기능 활용 팁
