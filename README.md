# Claude Code 학습 가이드

Claude Code를 단계별로 학습할 수 있는 인터랙티브 가이드입니다.
체크리스트 기반으로 진행 상황을 추적하며, 퀴즈와 실습으로 학습합니다.

## 학습 로드맵

```
Phase 1  기본 사용법          ←── Deep Dive 1. 작동 방식
Phase 2  CLAUDE.md 설정       ←── Deep Dive 3. 지침 및 메모리
Phase 3  권한 및 보안
Phase 4  효율적인 작업 패턴    ←── Deep Dive 4. 일반적인 워크플로우
                              ←── Deep Dive 5. 모범 사례
Phase 5  커스터마이징          ←── Deep Dive 2. 확장하기
Phase 6  고급 기능
```

---

## 시작하기

### 1. 저장소 클론

```bash
git clone https://github.com/sdgranger/claude-code-guide.git
cd claude-code-guide
```

### 2. Claude Code 실행

```bash
claude
```

---

## 사용법

### 방법 1: `/learn` 스킬 (권장)

처음부터 순서대로 학습하며, 자동으로 다음 미체크 항목을 찾아 이어서 진행합니다.

```
/learn
```

- 첫 실행 시 Phase 1의 첫 항목부터 시작
- 다음에 돌아와서 `/learn` 하면 마지막 체크한 곳 이후부터 이어서 진행
- 진행 상황은 `claude-code-guide.md`의 체크박스(`[ ]` → `[x]`)로 추적

### 방법 2: 직접 프롬프트

스킬 없이도 Claude에게 직접 말하면 됩니다.

#### 이어서 학습하기

```
claude-code-guide.md 이어서 학습하자
```

#### 특정 Phase 학습

```
claude-code-guide.md의 Phase 1부터 학습하자
```

```
claude-code-guide.md의 Phase 4: 효율적인 작업 패턴을 학습하자
```

#### 특정 섹션만 학습

```
claude-code-guide.md의 1.2 슬래시 명령어 부분을 학습하자
```

```
claude-code-guide.md의 5.3 Hooks 부분을 학습하자
```

#### 심화 학습 (Deep Dive)

```
claude-code-how-it-works.md 읽고 학습하자
```

```
claude-code-best-practices.md의 실패 패턴 부분을 학습하자
```

#### 진행 상황 확인

```
claude-code-guide.md에서 내 진행 상황을 알려줘
```

---

## 학습 파일 구조

| 파일 | 내용 |
|------|------|
| `claude-code-guide.md` | 메인 학습 체크리스트 (Phase 1~6) |
| `claude-code-how-it-works.md` | Deep Dive 1: 에이전트 루프, 모델, 도구, 컨텍스트 |
| `claude-code-memory.md` | Deep Dive 3: CLAUDE.md 작성법, 자동 메모리 |
| `claude-code-workflows.md` | Deep Dive 4: 코드 탐색, 버그 수정, PR, Git |
| `claude-code-extensions.md` | Deep Dive 2: Skill, Subagent, Hook, MCP |
| `claude-code-best-practices.md` | Deep Dive 5: 프롬프트 전략, 세션 관리, 실패 패턴 |

---

## 학습 진행 레벨

### 입문 (일상 업무에 충분)
- Phase 1~2 + Deep Dive 1, 3

### 중급 (효율 향상)
- Phase 3~4 + Deep Dive 4, 5

### 고급 (자동화 & 확장)
- Phase 5~6 + Deep Dive 2

---

## 참고

- 학습 내용은 [Claude Code 공식 문서](https://code.claude.com/docs/ko/overview)를 기반으로 합니다.
- 진행 상황은 `claude-code-guide.md`의 체크박스에 저장되므로, 개인 브랜치에서 학습하는 것을 권장합니다.

```bash
git checkout -b my-learning
```
