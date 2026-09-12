---
name: piedpiper
description: 명세 기반 Phase 워크플로의 현재 상태를 보고 다음에 할 일을 알려준다. 인자 없이 실행하면 진행 상황을 요약하고, next를 주면 다음에 입력할 명령어를 제시한다.
argument-hint: [status|next|init|reset]
disable-model-invocation: true
allowed-tools: Read, Glob
---

## 계획

@CLAUDE.md

## 진행 상태

@.progress-report/state.md

## 요청

$ARGUMENTS

---

## 지시

요청 인자에 따라 아래 중 하나를 수행한다. 인자가 없으면 `status`로 간주한다.

참조한 두 파일 중 하나라도 존재하지 않으면 그 사실을 먼저 알리고,
`init` 안내로 넘어간다. 없는 파일을 있는 것처럼 다루지 않는다.

### status

현재 진행 상황을 표로 보여준다.

| 기능 | Phase | 상태 |
|---|---|---|

마지막에 한 줄로 요약한다: 완료 Phase 수 / 전체, 그리고 지금 어느 지점인지.
미해결 debt 항목이 `.progress-report/debt.md`에 있으면 개수만 덧붙인다.

### next

상태를 판단해, 사용자가 **직접 입력할 명령어 한 줄**을 제시한다.
아래 순서로 먼저 걸리는 조건을 따른다.

1. `CLAUDE.md`가 없다 → `/piedpiper init` 안내
2. `.progress-report/state.md`가 없다 → `/piedpiper-clarify`
3. `.progress-report/research-*.md`가 하나도 없다 →
   `/deep-research <질문>` 형태로, 질문 초안까지 완성해서 제시
4. `CLAUDE.md`에 Phase 구조가 없다 → `/piedpiper-plan`
5. 진행 중인 Phase가 있다 → 그 Phase의 완료 조건을 그대로 넣은
   `/goal <조건>` 명령을 완성해서 제시
6. Phase가 방금 끝났고 리뷰 전이다 → `/ponytail-review`
7. 기능의 마지막 Phase가 끝났다 → `/piedpiper-wrap`
8. 모든 기능이 끝났다 → 최종 점검 절차 안내

**중요:** `/goal`, `/deep-research`, `/ponytail` 계열은 직접 실행하지 않는다.
복사해서 붙여넣을 수 있는 완성된 문자열로 출력만 한다.
이유는 두 가지다. 빌트인 명령어는 스킬 안에서 호출할 수 없고,
`/goal` 조건은 사용자가 눈으로 확인하고 승인해야 하는 지점이다.

### init

`.progress-report/` 디렉터리와 `state.md`를 만든다.
`CLAUDE.md`가 없으면 명세 파일 위치를 먼저 물어본다.

`state.md` 초기 형태:

```markdown
# 진행 상태

## 현재 위치
아직 시작 전

## 완료된 Phase
(없음)

## 이월 항목
(없음)
```

### reset

상태 파일을 초기화한다. 실행 전에 무엇이 지워지는지 보여주고 확인을 받는다.
확인 없이 지우지 않는다.
