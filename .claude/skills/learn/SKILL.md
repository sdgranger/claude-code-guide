---
name: learn
description: Claude Code 학습 가이드 - 이어서 학습하기. 미체크 항목을 자동으로 찾아 순서대로 진행합니다.
---

# Claude Code 학습 가이드 — 이어서 학습하기

## 동작 방식

1. @claude-code-guide.md 를 읽습니다.
2. 파일에서 첫 번째 미체크 항목 `- [ ]`을 찾습니다.
3. 해당 항목이 속한 Phase/섹션의 제목을 함께 표시하여 현재 위치를 알려줍니다.
4. 해당 항목을 학습시킵니다.
5. 학습이 끝나면 체크 `- [x]`로 업데이트합니다.
6. 다음 항목으로 이어서 진행할지 AskUserQuestion으로 물어봅니다.

## 학습 진행 규칙

### 항목 유형별 처리

**CLI 명령어/단축키 항목** (예: `` `claude -c` — 가장 최근 대화 이어서 계속 ``):
1. 해당 명령어/단축키가 무엇인지 간단히 설명합니다.
2. 공식 문서에서 정확한 정보를 확인합니다 (아래 문서 참조 규칙).
3. 실제로 따라해볼 수 있는 예시를 제시합니다.
4. AskUserQuestion으로 이해를 확인하는 퀴즈를 냅니다.
5. 정답이면 체크하고 다음으로 넘어갑니다. 오답이면 다시 설명합니다.

**설정/작성 항목** (예: `프로젝트용 CLAUDE.md 작성`):
1. 무엇을 해야 하는지 설명합니다.
2. 구체적인 예시와 템플릿을 제시합니다.
3. AskUserQuestion으로 직접 해보았는지 확인합니다.
4. 완료 확인 후 체크합니다.

**심화 학습 항목** (예: `[작동 방식](claude-code-how-it-works.md) 1장: ...`):
1. 해당 가이드 파일을 `@`로 참조하여 관련 섹션을 읽습니다.
2. 핵심 내용을 요약하여 설명합니다.
3. AskUserQuestion으로 퀴즈를 냅니다.
4. 정답이면 체크합니다.

### 문서 참조 규칙

답변의 정확성을 보장하기 위해 공식 문서를 curl로 참조합니다:

| 주제 | 공식 문서 URL |
|------|--------------|
| CLI 사용법 | https://code.claude.com/docs/en/cli-usage |
| 작동 방식 | https://code.claude.com/docs/en/how-claude-code-works |
| 메모리/CLAUDE.md | https://code.claude.com/docs/en/memory |
| 권한 | https://code.claude.com/docs/en/permissions |
| 워크플로우 | https://code.claude.com/docs/en/common-workflows |
| 모범 사례 | https://code.claude.com/docs/en/best-practices |
| 확장 기능 | https://code.claude.com/docs/en/features-overview |
| 스킬 | https://code.claude.com/docs/en/skills |
| 서브에이전트 | https://code.claude.com/docs/en/sub-agents |
| Hooks | https://code.claude.com/docs/en/hooks |
| MCP | https://code.claude.com/docs/en/mcp |

### 진행 상황 표시

학습 시작 시 현재 진행률을 간단히 보여줍니다:

```
📍 현재 위치: Phase 1 > 1.2 세션 내 슬래시 명령어 > `/compact`
📊 진행률: 8/67 완료 (12%)
```

### 세션 종료 시

사용자가 그만하겠다고 하면:
1. 현재까지 완료한 항목을 모두 `[x]`로 저장합니다.
2. 다음에 `/learn` 또는 "이어서 학습하자"로 돌아오면 이어서 진행된다고 안내합니다.

## 심화 가이드 파일 참조

학습 중 심화 내용이 필요할 때 아래 파일을 `@`로 참조합니다:
- @claude-code-how-it-works.md — 에이전트 루프, 모델, 도구, 컨텍스트
- @claude-code-memory.md — CLAUDE.md 작성법, 자동 메모리
- @claude-code-workflows.md — 실전 워크플로우
- @claude-code-extensions.md — Skill, Subagent, Hook, MCP
- @claude-code-best-practices.md — 프롬프트 전략, 실패 패턴
