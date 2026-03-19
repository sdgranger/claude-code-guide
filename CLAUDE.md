# claude code 학습과 잘 사용하기 위한 샘플 프로젝트

claude-code-guide는 틀린 답을 낼 때가 있다. 사용자가 질문을 하면 https://code.claude.com/docs/en/overview 에서 md 파일 을 curl 로 참조해서 답해.
그 후에는 AskUserQuestion 으로 퀴즈를 내서 직접 따라하게 안내해.

## 학습 스킬 사용법

이 프로젝트를 clone 한 후 아래 명령어로 학습을 시작합니다:

- `/learn` — 다음 미체크 항목부터 자동으로 이어서 학습
- "이어서 학습하자" 또는 "claude-code-guide.md 이어서 학습하자" — 동일하게 동작

학습 진행 상황은 claude-code-guide.md 의 체크박스(`[ ]` → `[x]`)로 추적됩니다.
