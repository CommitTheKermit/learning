---
name: android-study
description: learning 프로젝트에서 매니페스트 안드로이드 인터뷰 책의 다음 질문(또는 지정한 질문)을 골라 원문을 읽고, 답안 노트를 만든 뒤 소크라테스식으로 한 개념씩 학습을 진행한다. 사용자가 "/android-study", "학습 시작", "공부 시작", "다음 질문", "Q12 공부하자", "Compose Q3 하자" 같은 표현을 쓰면 발동.
---

# android-study

책 한 문항을 "면접에서 내 말로 설명하고 꼬리 질문을 버틸 수 있는" 상태까지 끌고 간다. 답은 사용자가 쓴다. 대신 써 주지 않는다.

## 기준 자료

- 교재: `docs/android-interview/manifest-android-interview-kr.pdf`
- 목차: PDF 3-8쪽. **PDF 쪽 = 책 쪽 + 8** (Q0 11→19, Q56 259→267, Compose Q0 348→356 확인)
- 일정과 판정 기준: `.claude/skills/jansori/SKILL.md` 의 `## 판정 기준`
- 학습 방식: `~/.claude/commands/socratic-learn.md` (사이클 규칙, 채점 신호 🟢🟡🔴)

## 1. 질문 고르기

- 사용자가 질문을 지정했으면 그 질문을 고른다. "Compose" 언급이 없으면 안드로이드 장으로 본다.
- 지정하지 않았으면 아래 순서에서 **답안이 아직 없는 첫 질문**을 고른다.
  1. 핵심 48문항: 안드로이드 Q0-Q19 → Q54 → Q56 → Compose Q0-Q25
  2. 나머지: 안드로이드 Q20-Q69 (Q54, Q56 제외) → Compose Q26-Q43
- 노트 파일명: 안드로이드 장 `docs/android-interview/q{N}-notes.md`, Compose 장 `docs/android-interview/c{N}-notes.md`
- "답안 없음" = 노트 파일이 없거나 `TODO(human)` 이 남아 있음.

```bash
cd /Users/ujeonghyeon/Desktop/dev/myDev/learning/docs/android-interview
for n in $(seq 0 19) 54 56; do f=q$n-notes.md; { [ ! -e $f ] || grep -q 'TODO(human)' $f; } && { echo $f; exit; }; done
for n in $(seq 0 25); do f=c$n-notes.md; { [ ! -e $f ] || grep -q 'TODO(human)' $f; } && { echo $f; exit; }; done
echo "핵심 48문항 완료"
```

어떤 질문을 골랐는지, 핵심 48문항 중 몇 번째인지 먼저 알린다.

## 2. 원문 읽기

- 목차(PDF 3-8쪽)에서 그 질문의 시작 쪽과 다음 질문의 시작 쪽을 찾아, PDF 쪽으로 바꿔 그 범위만 Read 한다(20쪽 단위).
- 질문 끝의 `실전 질문` 블록을 찾아 둔다.

## 3. 노트 만들기

노트 파일이 없을 때만 아래 형식으로 만든다. **이미 있으면 절대 덮어쓰지 않는다**(사용자의 답안이다).

```markdown
# Q{N}. {질문 제목} - 내 답안 노트

- 원문: `manifest-android-interview-kr.pdf` {책 시작}~{책 끝}쪽 (PDF {시작}~{끝}쪽)

## 실전 질문

> {원문 실전 질문을 그대로}

## 내 답안

TODO(human): 면접에서 말하듯 5~10줄로 작성
```

Compose 장이면 제목을 `# Compose Q{N}. ...` 으로 쓴다.

## 4. 소크라테스식 학습

`~/.claude/commands/socratic-learn.md` 의 사이클 규칙을 그대로 따른다. 이 프로젝트에서 덧붙이는 규칙은 다음과 같다.

- 한 사이클에 개념 하나만 다룬다. 원문을 한꺼번에 요약해 주지 않는다.
- 사용자가 어렵다고 하면 즉시 한 단계 위 기초로 후퇴한다(Q0 때 Framework·ART·HAL을 한꺼번에 다뤘다가 처음부터 다시 한 적이 있다).
- 깊이 상한은 "면접 설명 가능"이다. 원문이 그보다 깊은 내부 구현으로 들어가면 "면접 기준 밖"이라고 표시하고 건너뛴다.
- 설명은 원문 근거로 한다. 원문에 없는 내용을 보탤 때는 공식 문서 근거를 대거나 "원문 밖"이라고 밝힌다.

## 5. 답안 작성과 마무리

- 개념 사이클이 끝나면 사용자에게 노트의 `TODO(human)` 줄을 지우고 `## 내 답안` 에 직접 답을 쓰라고 요청한다.
- 답안을 받으면 실전 질문 기준으로 🟢🟡🔴 채점하고, 면접관처럼 **꼬리 질문 1-2개**를 던진다. 버티면 이 문항은 끝이다.
- 사용자가 답안을 써 달라고 해도 대신 쓰지 않는다. 막힌 지점을 질문으로 되돌려 준다.
- 할 일 체크는 하지 않는다. "책 학습" 항목은 114문항이 모두 끝났을 때 사용자에게 완료를 제안한다.
- 마지막에 다음 질문 번호와, 오늘 날짜 기준으로 jansori 판정상 기대치보다 앞서는지 뒤처지는지를 한 줄로 알린다.
