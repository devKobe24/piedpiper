---
name: piedpiper-wrap
description: 한 기능의 마지막 Phase가 끝났을 때 완료 보고서를 작성한다. 레포 전체 감사 결과와 미해결 항목을 정리해 .progress-report에 남긴다.
argument-hint: [기능 번호 또는 이름]
disable-model-invocation: true
allowed-tools: Read, Glob
---

## 계획

@CLAUDE.md

## 진행 상태

@.progress-report/state.md

## 대상 기능

$ARGUMENTS

---

## 지시

대상 기능의 모든 Phase가 끝났다는 전제로 완료 보고서를 작성한다.
인자가 없으면 상태 파일에서 가장 최근에 완료된 기능을 대상으로 삼는다.

아직 끝나지 않은 Phase가 있으면 멈추고 어느 Phase가 남았는지 알린다.

### 1. 완료 조건 검증

각 Phase의 완료 조건을 `CLAUDE.md`에서 가져와, 실제로 충족되었는지
코드와 테스트로 확인한다. 충족되지 않은 것은 숨기지 말고 그대로 적는다.

### 2. 감사 결과 수집

사용자에게 `/ponytail-audit`를 먼저 돌렸는지 확인한다.
안 돌렸으면 돌리고 오라고 안내하고 멈춘다.

`/ponytail-review`가 diff만 보는 것과 달리 `/ponytail-audit`는 레포 전체를 본다.
Phase 단위로는 안 보이다가 기능 단위로 누적되면 드러나는 중복
(여러 Phase에 걸쳐 비슷한 유틸이 반복 생성된 경우 등)을 여기서 잡는다.

### 3. 보고서 작성

`.progress-report/feature-<번호>-report.md`에 저장한다.

```markdown
# 기능 <번호>: <이름> 완료 보고서

작성일: <날짜>

## Phase별 완료 조건 충족 여부
| Phase | 완료 조건 | 충족 |
|---|---|---|

## 감사 결과
### 중복·과잉으로 지목된 항목
### 조치한 것 / 남긴 것과 그 이유

## 미해결 debt
(.progress-report/debt.md 에서 이 기능과 관련된 항목)

## 다음 기능에 영향을 주는 결정
<뒤이을 기능이 전제로 삼아야 할 것들>

## 발견된 문제
<완료 조건은 충족했지만 찜찜한 것들. 없으면 '없음'>
```

### 4. 상태 갱신

`.progress-report/state.md`에 기능 완료를 기록하고,
다음 기능의 첫 Phase를 "현재 위치"로 옮긴다.

### 5. 다음 안내

다음 기능의 첫 Phase 완료 조건을 넣은 `/goal` 명령을 완성해서 제시한다.
마지막 기능이었다면 최종 점검 절차를 안내한다.

- `.progress-report/` 전체 보고서 전수 조사
- `/ponytail-debt` 로 미해결 장부 최종 확인
- `/code-review --fix` 로 정확성 버그 점검
- 원본 명세와 대조해 요구사항 누락 확인
