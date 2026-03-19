# Claude Code 지침 및 메모리 가이드

> 출처: [Claude가 프로젝트를 기억하는 방법](https://code.claude.com/docs/ko/memory)
> CLAUDE.md 파일과 자동 메모리를 통해 세션 간 지식을 전달하는 방법을 이해합니다.

---

## 1. 두 가지 메모리 시스템

Claude Code는 두 가지 메커니즘으로 세션 간 지식을 전달합니다:

| | CLAUDE.md 파일 | 자동 메모리 |
|---|---|---|
| **작성자** | 사용자 (당신) | Claude (자동) |
| **포함 내용** | 지침 및 규칙 | 학습 및 패턴 |
| **범위** | 프로젝트, 사용자 또는 조직 | 작업 트리당 |
| **로드 대상** | 모든 세션 (전체) | 모든 세션 (처음 200줄) |
| **용도** | 코딩 표준, 워크플로우, 아키텍처 | 빌드 명령, 디버깅 인사이트, 선호도 |

> Claude의 동작을 **의도적으로 안내**하려면 → **CLAUDE.md**
> Claude가 **자동으로 학습**하도록 하려면 → **자동 메모리**

- [ ] 두 메모리 시스템의 차이 이해하기
- [ ] `/memory` 명령으로 현재 로드된 메모리 확인해보기

---

## 2. CLAUDE.md 파일

### 2.1 위치와 범위

| 범위 | 위치 | 용도 | 공유 |
|------|------|------|------|
| **관리 정책** | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | 조직 전체 지침 | 조직 모든 사용자 |
| **프로젝트** | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 팀 공유 지침 | git으로 팀 멤버 |
| **사용자** | `~/.claude/CLAUDE.md` | 개인 선호도 | 본인만 |

### 로드 순서
```
~/.claude/CLAUDE.md          ← 개인 (항상 로드)
    ↓
../../CLAUDE.md              ← 상위 디렉토리 (올라가며 탐색)
    ↓
../CLAUDE.md                 ← 상위 디렉토리
    ↓
./CLAUDE.md                  ← 프로젝트 루트 (항상 로드)
    ↓
./sub-dir/CLAUDE.md          ← 하위 디렉토리 (해당 파일 작업 시 지연 로드)
```

- [ ] `/init` 실행하여 CLAUDE.md 자동 생성해보기
- [ ] `./CLAUDE.md`에 프로젝트 규칙 작성하기
- [ ] `~/.claude/CLAUDE.md`에 개인 선호도 작성하기

### 2.2 효과적인 지침 작성법

#### 좋은 지침의 3원칙

1. **구체적**: 검증 가능할 정도로 구체적으로
2. **간결**: 200줄 이하 목표
3. **일관적**: 충돌하는 규칙 없이

| 나쁜 예 | 좋은 예 |
|---------|---------|
| "코드를 제대로 포맷하세요" | "2칸 들여쓰기 사용" |
| "변경 사항을 테스트하세요" | "커밋 전 `npm test` 실행" |
| "파일을 정리하세요" | "API 핸들러는 `src/api/handlers/`에 위치" |

#### 핵심 질문
> *"이 줄을 제거하면 Claude가 실수를 할까?"*
> 아니라면 삭제하세요!

#### 포함할 것 / 제외할 것

| 포함 | 제외 |
|------|------|
| Claude가 추측 불가한 Bash 명령 | Claude가 코드를 읽어 파악 가능한 것 |
| 기본값과 다른 코드 스타일 | 표준 언어 규칙 |
| 테스트 지시사항 | 상세한 API 문서 (링크로 대체) |
| 아키텍처 결정 | 긴 튜토리얼 |
| 일반적인 함정 | 파일별 설명 |

- [ ] 현재 CLAUDE.md를 위 기준으로 점검하기
- [ ] 불필요한 내용 제거하기

### 2.3 파일 Import

```markdown
# CLAUDE.md 안에서
프로젝트 개요: @README.md
npm 명령: @package.json
git 규칙: @docs/git-instructions.md
개인 설정: @~/.claude/my-project-instructions.md
```

> 상대 경로는 CLAUDE.md 파일 위치 기준
> 재귀 import 최대 5단계

- [ ] `@파일명` 구문으로 외부 문서 import 해보기

---

## 3. `.claude/rules/` — 규칙 구성

### 3.1 기본 설정

```
.claude/
├── CLAUDE.md           # 주 프로젝트 지침
└── rules/
    ├── code-style.md   # 코드 스타일
    ├── testing.md      # 테스트 규칙
    └── security.md     # 보안 요구사항
```

> `paths:` frontmatter가 없는 규칙 = 모든 세션에 로드 (CLAUDE.md와 동일)

### 3.2 경로별 규칙 (핵심 기능!)

특정 파일 작업 시에만 로드되어 **컨텍스트를 절약**합니다:

```markdown
# .claude/rules/api-rules.md
---
paths:
  - "src/api/**/*.ts"
  - "src/api/**/*.java"
---

# API 개발 규칙
- 모든 API 엔드포인트에 입력 검증 포함
- 표준 오류 응답 형식 사용
- OpenAPI 문서 주석 포함
```

### Glob 패턴 예시

| 패턴 | 매칭 |
|------|------|
| `**/*.ts` | 모든 TypeScript 파일 |
| `src/**/*` | src/ 아래 모든 파일 |
| `*.md` | 루트의 마크다운 파일 |
| `src/**/*.{ts,tsx}` | 중괄호 확장 (여러 확장자) |

- [ ] `.claude/rules/` 디렉토리 생성하기
- [ ] `paths:` frontmatter를 사용한 경로별 규칙 만들어보기

### 3.3 사용자 수준 규칙

```
~/.claude/rules/
├── preferences.md    # 개인 코딩 선호도
└── workflows.md      # 선호 워크플로우
```

> 사용자 수준 규칙은 프로젝트 규칙보다 낮은 우선순위

---

## 4. 자동 메모리

### 4.1 작동 방식

Claude가 작업하면서 자동으로 노트를 저장합니다:
- 빌드 명령
- 디버깅 인사이트
- 아키텍처 노트
- 코드 스타일 선호도
- 워크플로우 습관

> 모든 세션마다 저장하지 않습니다. 향후 대화에서 유용한지 Claude가 판단합니다.

### 4.2 저장 위치

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 간결한 인덱스 (처음 200줄만 세션 시작 시 로드)
├── debugging.md       # 디버깅 패턴
├── api-conventions.md # API 설계 결정
└── ...                # 기타 주제 파일
```

> **MEMORY.md의 처음 200줄만** 모든 세션에 로드됩니다.
> 200줄 초과 시 Claude가 상세 내용을 별도 주제 파일로 이동합니다.
> 주제 파일은 Claude가 필요할 때 읽습니다.

### 4.3 활성화/비활성화

```bash
# 세션 내에서
/memory    # 토글 가능

# 설정으로
# .claude/settings.json
{
  "autoMemoryEnabled": false
}

# 환경 변수로
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1
```

### 4.4 메모리 편집

자동 메모리 파일은 **일반 마크다운**입니다. 언제든 편집/삭제 가능합니다.

```bash
/memory    # 세션 내에서 파일 찾아보기 및 편집
```

Claude에게 직접 기억하라고 요청할 수도 있습니다:
```
"항상 pnpm을 사용하도록 기억해"
"API 테스트에 로컬 Redis가 필요하다는 것을 기억해"
```

- [ ] `/memory`로 자동 메모리 상태 확인하기
- [ ] Claude에게 "~를 기억해" 요청해보기
- [ ] `~/.claude/projects/` 아래 MEMORY.md 파일 직접 확인해보기

---

## 5. CLAUDE.md vs 자동 메모리 vs Skill — 언제 어떤 것을?

```
항상 필요한 프로젝트 규칙     → CLAUDE.md
특정 파일 작업 시만 필요       → .claude/rules/ (paths: 지정)
가끔 필요한 참조 자료         → Skill
Claude가 자동으로 학습        → 자동 메모리
```

### CLAUDE.md 크기 관리 전략

```
CLAUDE.md (< 200줄)
├── 핵심 빌드 명령
├── 코드 스타일 (기본값과 다른 것만)
├── 워크플로우 규칙
└── @docs/detailed-guide.md (import로 참조)

.claude/rules/
├── api-rules.md (paths: src/api/**)
├── test-rules.md (paths: **/*.test.*)
└── frontend-rules.md (paths: src/components/**)

.claude/skills/
└── deploy/SKILL.md (온디맨드 워크플로우)
```

- [ ] CLAUDE.md 크기가 200줄 초과하는지 확인
- [ ] 초과 시 Rules나 Skill로 분리 계획 세우기

---

## 6. 문제 해결

| 문제 | 원인 | 해결 |
|------|------|------|
| Claude가 CLAUDE.md를 따르지 않음 | 지침이 모호하거나 충돌 | `/memory`로 로드 확인, 지침 구체화, 충돌 제거 |
| `/compact` 후 지침 손실 | 대화에서만 전달 (CLAUDE.md에 미작성) | CLAUDE.md에 지속적 규칙 추가 |
| CLAUDE.md가 너무 큼 | 참조 콘텐츠가 본문에 포함 | `@import` 또는 `.claude/rules/`로 분리 |
| 자동 메모리 내용 모름 | 파일 미확인 | `/memory`로 확인 및 편집 |
| 다른 팀의 CLAUDE.md 간섭 (모노레포) | 상위 CLAUDE.md 자동 로드 | `claudeMdExcludes` 설정 |

### `claudeMdExcludes` 설정 (모노레포용)

```json
// .claude/settings.local.json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

- [ ] `/memory`로 현재 로드된 모든 지침 파일 확인하기
- [ ] 충돌하는 지침이 있는지 점검하기

---

> **핵심 정리**:
> - **CLAUDE.md** = 당신이 Claude에게 주는 지침 (항상 로드, 간결하게)
> - **자동 메모리** = Claude가 스스로 저장하는 학습 (자동, 편집 가능)
> - **Rules** = 파일 경로별로 범위 지정 가능한 규칙 (컨텍스트 절약)
> - 지속적 규칙은 반드시 **CLAUDE.md에 작성** (대화 중 전달만 하면 압축 시 손실)
