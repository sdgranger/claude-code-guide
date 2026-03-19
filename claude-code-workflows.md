# Claude Code 일반적인 워크플로우 가이드

> 출처: [일반적인 워크플로우](https://code.claude.com/docs/ko/common-workflows)
> 코드베이스 탐색, 버그 수정, 리팩토링, 테스트, PR 생성 등 실전 워크플로우입니다.

---

## 1. 새로운 코드베이스 이해하기

### 1.1 코드베이스 빠른 개요

```bash
cd /path/to/project
claude
```

```
1단계: "이 코드베이스의 개요를 알려줘"
2단계: "주요 아키텍처 패턴을 설명해줘"
3단계: "핵심 데이터 모델이 뭐야?"
4단계: "인증은 어떻게 처리해?"
```

> **Tip**: 광범위한 질문 → 특정 영역으로 좁히기

### 1.2 관련 코드 찾기

```
"사용자 인증을 처리하는 파일을 찾아줘"
→ "이 인증 파일들이 어떻게 연동되는지 설명해줘"
→ "로그인 프로세스를 프론트엔드부터 DB까지 추적해줘"
```

> **코드 인텔리전스 플러그인** 설치 시 "정의로 이동", "참조 찾기" 기능 사용 가능

- [ ] Claude에게 현재 프로젝트의 개요 요청해보기
- [ ] 특정 기능의 코드 흐름 추적 요청해보기

---

## 2. 버그 수정

```
1단계: "npm test 실행하면 오류가 나와"
       또는 오류 메시지 직접 붙여넣기
       또는 `cat error.log | claude`

2단계: "user.ts의 @ts-ignore 수정 방법을 제안해줘"

3단계: "제안한 null 체크를 user.ts에 적용해줘"
```

### 효과적인 버그 리포트 프롬프트

| 나쁜 예 | 좋은 예 |
|---------|---------|
| "로그인이 안 돼" | "세션 타임아웃 후 로그인 실패. src/auth/ 토큰 리프레시 확인. 실패 테스트 작성 후 수정해" |
| "빌드 에러" | "빌드가 이 오류로 실패: [오류 붙여넣기]. 수정하고 빌드 성공 확인해. 근본 원인 해결해" |
| "느려" | "UserService.getAll()이 1000건 이상에서 10초 걸림. 쿼리 최적화해줘" |

- [ ] 오류 메시지와 함께 구체적인 버그 수정 요청해보기
- [ ] "수정 후 테스트 실행해서 확인해" 패턴 사용해보기

---

## 3. 코드 리팩토링

```
1단계: "코드베이스에서 deprecated API 사용을 찾아줘"

2단계: "utils.js를 최신 JavaScript로 리팩토링하는 방법 제안해"

3단계: "utils.js를 ES2024 기능으로 리팩토링해. 동일한 동작 유지해"

4단계: "리팩토링된 코드의 테스트를 실행해"
```

> **Tip**: 작고 테스트 가능한 단위로 리팩토링하세요.

- [ ] 리팩토링 요청 시 "동일한 동작 유지" 조건 명시하기
- [ ] 리팩토링 후 테스트 실행 요청하기

---

## 4. Plan Mode로 안전한 코드 분석

### Plan Mode 진입 방법

| 방법 | 명령 |
|------|------|
| **세션 중 전환** | `Shift+Tab` × 2 (기본값 → Auto-Accept → Plan Mode) |
| **새 세션 시작** | `claude --permission-mode plan` |
| **비대화형 분석** | `claude --permission-mode plan -p "인증 시스템 분석"` |

### Plan Mode 워크플로우

```
[Plan Mode] "인증 시스템을 분석하고 OAuth2 마이그레이션 계획을 세워줘"
    ↓ Claude가 분석 및 계획 작성

[Ctrl+G] 계획을 에디터에서 직접 편집
    ↓

"하위 호환성은 어떻게 해?"
    ↓ Claude가 계획 보완

[Normal Mode로 전환] "계획대로 구현해줘"
```

### Plan Mode가 적합한 경우

| 적합 | 부적합 |
|------|--------|
| 다단계 구현 계획 | 오타 수정 |
| 코드베이스 탐색/조사 | 로그 한 줄 추가 |
| Claude와 방향 반복 논의 | 변수명 변경 |

> **간단한 작업은 Plan Mode를 건너뛰세요.** diff를 한 문장으로 설명할 수 있으면 바로 실행!

- [ ] `Shift+Tab`으로 Plan Mode 진입해보기
- [ ] Plan Mode에서 코드 분석 → Normal Mode에서 구현 해보기
- [ ] `Ctrl+G`로 계획을 에디터에서 편집해보기

---

## 5. 테스트 작업

```
1단계: "NotificationsService에서 테스트되지 않은 함수를 찾아줘"

2단계: "notification service에 대한 테스트를 추가해"

3단계: "notification service의 엣지 케이스 테스트를 추가해"

4단계: "새 테스트를 실행하고 실패하는 것을 수정해"
```

> **Tip**: Claude에게 놓쳤을 엣지 케이스를 식별하도록 요청하세요.
> 오류 조건, 경계값, 예상치 못한 입력을 분석합니다.

- [ ] 특정 서비스에 대한 테스트 추가 요청해보기
- [ ] "엣지 케이스 테스트도 추가해" 요청해보기

---

## 6. PR 생성

### 간단한 방법
```
"내 변경사항으로 PR 만들어줘"
```

### 단계별 방법
```
1단계: "인증 모듈 변경사항을 요약해줘"
2단계: "PR 만들어줘"
3단계: "PR 설명에 보안 개선 관련 컨텍스트를 추가해"
```

> `gh pr create`로 PR을 만들면 세션이 자동으로 해당 PR에 연결됩니다.
> 나중에 `claude --from-pr 123`으로 재개할 수 있습니다.

- [ ] Claude에게 PR 생성 요청해보기
- [ ] `claude --from-pr <number>`로 PR 기반 세션 시작해보기

---

## 7. Subagent 활용

### 자동 위임
```
"최근 코드 변경사항을 보안 이슈에 대해 검토해"
→ Claude가 자동으로 적절한 subagent에게 위임
```

### 명시적 사용
```
"subagent를 사용하여 auth 모듈을 보안 검토해"
"debugger subagent로 로그인 실패 원인을 조사해"
```

### 커스텀 Subagent 생성
```
/agents → "Create New subagent" 선택
```

- [ ] "subagent를 사용하여 조사해" 패턴 사용해보기
- [ ] `/agents`로 사용 가능한 subagent 확인

---

## 8. 세션 관리

### 세션 이름 지정 & 재개

```bash
# 세션 중 이름 지정
/rename auth-refactor

# 나중에 이름으로 재개
claude --resume auth-refactor

# 가장 최근 대화 재개
claude --continue

# 대화형 선택기
claude --resume
```

### 세션 선택기 단축키

| 단축키 | 작업 |
|--------|------|
| `↑/↓` | 세션 간 이동 |
| `Enter` | 세션 선택 |
| `P` | 미리보기 |
| `R` | 이름 바꾸기 |
| `/` | 검색 필터 |
| `B` | 현재 브랜치 필터 |
| `A` | 모든 프로젝트 보기 |

- [ ] `/rename`으로 현재 세션에 이름 지정해보기
- [ ] `claude --resume`으로 세션 선택기 사용해보기

---

## 9. Git Worktree로 병렬 세션

### Worktree 사용법

```bash
# 격리된 worktree에서 Claude 시작
claude --worktree feature-auth

# 다른 worktree에서 병렬 세션
claude --worktree bugfix-123

# 자동 이름 생성
claude --worktree
```

> Worktree = 같은 저장소의 **별도 작업 디렉토리** (브랜치별 격리)
> `.claude/worktrees/<name>/`에 생성됩니다.

### Worktree 정리
- **변경사항 없음** → 자동 제거
- **변경사항 있음** → Claude가 유지/제거 여부 질문

> `.gitignore`에 `.claude/worktrees/` 추가하세요.

- [ ] `claude --worktree feature-name`으로 격리 세션 시작해보기
- [ ] 두 터미널에서 다른 worktree로 병렬 작업 해보기

---

## 10. 알림 설정

Claude가 완료되거나 입력 필요 시 데스크톱 알림을 받을 수 있습니다.

### 설정 방법

```
/hooks → Notification 선택 → + Match all → hook 명령 추가
```

### macOS 알림 명령
```bash
osascript -e 'display notification "Claude Code needs your attention" with title "Claude Code"'
```

### 알림 유형

| Matcher | 발생 시기 |
|---------|---------|
| `permission_prompt` | 도구 승인 요청 시 |
| `idle_prompt` | 작업 완료, 다음 입력 대기 |
| `auth_success` | 인증 완료 |
| `elicitation_dialog` | Claude가 질문할 때 |

- [ ] `/hooks`로 알림 Hook 설정해보기

---

## 11. Extended Thinking (확장된 사고)

### 기본 설정

| 설정 | 방법 | 용도 |
|------|------|------|
| 노력 수준 | `/model`에서 조정 | low / medium / high |
| `ultrathink` | 프롬프트에 포함 | 한 번만 깊은 추론 |
| 토글 | `Option+T` / `Alt+T` | 현재 세션 on/off |
| 전역 기본값 | `/config` | 모든 프로젝트 기본값 |

### 언제 사용?
- 복잡한 아키텍처 결정
- 어려운 버그 분석
- 다단계 구현 계획
- 트레이드오프 평가

```
"ultrathink: 이 아키텍처의 트레이드오프를 분석해줘"
```

> `Ctrl+O`로 사고 과정을 확인할 수 있습니다 (회색 이탤릭).

- [ ] `Option+T`로 Thinking 토글해보기
- [ ] `Ctrl+O`로 사고 과정 확인해보기
- [ ] "ultrathink" 키워드로 깊은 분석 요청해보기

---

## 12. 이미지 & 파일 참조

### 이미지 사용
```
1. 드래그 앤 드롭으로 이미지 추가
2. Ctrl+V로 클립보드에서 붙여넣기
3. 경로 지정: "이 이미지 분석해: /path/to/image.png"
```

### 파일/디렉토리 참조
```
"@src/utils/auth.js 의 로직을 설명해"     # 파일 내용 포함
"@src/components 의 구조를 보여줘"          # 디렉토리 목록
```

- [ ] `@파일명`으로 파일 참조하여 질문해보기
- [ ] 이미지 붙여넣기로 스크린샷 분석 요청해보기

---

## 13. Unix 유틸리티로 사용

### 파이프 입출력
```bash
# 빌드 오류 분석
cat build-error.txt | claude -p '이 빌드 오류의 근본 원인을 설명해' > output.txt

# 로그 파싱
cat server.log | claude -p '에러 패턴을 분석해' --output-format json
```

### CI/CD 연동
```json
{
  "scripts": {
    "lint:claude": "claude -p 'main 대비 변경사항의 타이포를 리포트해. 파일명:줄번호 형식으로.'"
  }
}
```

### 출력 형식

| 형식 | 명령 | 용도 |
|------|------|------|
| **텍스트** (기본값) | `--output-format text` | 간단한 결과 |
| **JSON** | `--output-format json` | 구조화된 데이터 |
| **스트리밍 JSON** | `--output-format stream-json` | 실시간 처리 |

- [ ] `cat file | claude -p "분석해"` 파이프 사용해보기
- [ ] `--output-format json`으로 구조화된 출력 받아보기

---

> **실전 워크플로우 요약**
>
> | 작업 | 핵심 패턴 |
> |------|---------|
> | 코드 이해 | 광범위 질문 → 구체적 영역으로 좁히기 |
> | 버그 수정 | 오류 공유 → 수정 제안 → 적용 → 테스트 |
> | 리팩토링 | deprecated 찾기 → 제안 → 적용 → 검증 |
> | 안전 분석 | Plan Mode → 분석 → 계획 → Normal Mode 구현 |
> | 테스트 | 미커버 코드 식별 → 테스트 생성 → 엣지 케이스 추가 |
> | PR 생성 | "PR 만들어줘" (Claude가 요약 + 생성) |
> | 병렬 작업 | `--worktree` 또는 여러 터미널 |
