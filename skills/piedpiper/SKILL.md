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

#### 전제: 기록과 실제 상태 대조

분기 판단에 들어가기 전에 먼저 확인한다.

`state.md`가 완료로 기록한 Phase의 완료 조건이 실제로 충족되는지,
해당 소스 파일과 테스트 파일의 존재로 대조한다.
`state.md`는 사람이 손으로도 고치는 파일이므로 기록을 그대로 믿지 않는다.

기록과 실제가 어긋나면 그 사실을 먼저 알리고,
**기록이 아니라 실제 상태를 기준으로** 다음 명령어를 제시한다.
규칙상 걸리는 분기와 실제로 해야 할 일이 다르면 둘 다 밝힌다.

#### 분기

아래 순서로 먼저 걸리는 조건을 따른다.

1. `CLAUDE.md`가 없다 →
   - 기존 명세 파일이 있는지 먼저 묻고, 있으면 그 경로를 받아 `CLAUDE.md` 초안을 만든다.
   - 없으면 만들려는 것을 설명해 달라고 요청해 `CLAUDE.md` 초안을 만든다.
   - 초안이 생긴 **뒤에야** `/piedpiper-clarify`를 안내한다.
     `/piedpiper-clarify`는 `CLAUDE.md`를 전제로 하므로, 없는 상태에서
     그쪽으로 보내면 다시 막힌다.
2. `CLAUDE.md`에 "확정된 결정" 섹션이 없다 → `/piedpiper-clarify`.
   이때 `.progress-report/clarify-*.md`가 이미 있으면,
   질문은 이미 만들어져 있으니 **답변만 하면 된다**는 점을 함께 알리고
   미결 항목을 나열한다.
3. `.progress-report/state.md`가 없다 → `/piedpiper init`
4. `.progress-report/research-*.md`가 없거나, 있어도 제목 줄만 있고
   실질 내용이 없다 → `/deep-research <질문>` 형태로 질문 초안까지
   완성해서 제시한다. 파일이 껍데기뿐인 경우에는 그 사실을 함께 알린다.
5. `CLAUDE.md`에 Phase 구조가 없다 → `/piedpiper-plan`
6. 진행 중인 Phase가 있다 → 그 Phase의 완료 조건을 **한 글자도 바꾸지 않고**
   그대로 넣은 `/goal <조건>` 명령을 완성해서 제시한다.
   이번 Phase의 범위 밖인 것(다음 Phase 몫)이 있으면 함께 짚는다.
7. Phase가 끝났는데 `state.md`에 리뷰 기록이 없다 → `/ponytail-review`.
   리뷰 후 이어질 다음 명령(`/goal` 또는 `/piedpiper-wrap`)도 함께 미리 보여준다.
   **마지막 Phase여도 리뷰가 먼저다.** 8번으로 건너뛰지 않는다.
   리뷰는 별도 세션이라 이전 Phase의 결정 근거를 보지 못한다.
   `state.md`에 "의도적으로 남긴 것" 항목이 있으면 함께 보여주어,
   리뷰가 지적하더라도 대조해 판단할 수 있게 한다.
8. 기능의 마지막 Phase가 끝났고 리뷰도 마쳤다 → `/piedpiper-wrap`
9. 모든 기능이 끝났다 → 최종 점검 절차 안내

#### 출력 규칙

`/goal`, `/deep-research`, `/ponytail` 계열은 직접 실행하지 않는다.
복사해서 붙여넣을 수 있는 완성된 문자열로 출력만 한다.

이유는 두 가지다. 빌트인 명령어는 스킬 안에서 호출할 수 없고,
`/goal` 조건은 사용자가 눈으로 확인하고 승인해야 하는 지점이다.

어느 분기에서 걸렸는지 판단 근거를 함께 보여준다.

### init

`.progress-report/` 디렉터리와 `state.md`를 만든다.
`CLAUDE.md`가 없으면 분기 1의 절차를 먼저 따른다.

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
