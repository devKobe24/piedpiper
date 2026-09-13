# piedpiper

**명세부터 구현·리뷰·완료 보고까지, 다음에 할 일을 안내하는 Claude Code 워크플로.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Release](https://img.shields.io/github/v/release/devKobe24/piedpiper)](https://github.com/devKobe24/piedpiper/releases)

piedpiper는 프로젝트의 명세, 조사 결과, 진행 기록을 읽고 **지금 실행할 다음 명령어**를 완성해 제시하는 오픈소스 플러그인입니다. 생성된 명령은 사용자가 내용을 확인한 뒤 직접 실행합니다.

작업 중 다음 단계가 궁금하면 Claude Code에서 입력하세요.

```text
/piedpiper next
```

[설치하기](#설치하기) · [첫 기능 진행하기](#첫-기능-진행하기) · [명령어](#명령어) · [파일과 진행 기록](#파일과-진행-기록) · [문제 해결](#문제-해결)

## 어떤 일을 도와주나요?

명세는 있지만 어디서부터 구현할지 막막하거나, 여러 세션에 걸친 작업에서 진행 상황과 결정 근거를 이어 가고 싶을 때 사용할 수 있습니다.

- **구현 전 질문 정리**: 미결 사항을 최대 7개의 질문으로 정리하고, 사용자 결정과 조사할 항목을 구분합니다.
- **단계별 구현 계획**: 명세를 테스트 가능한 크기의 Phase로 나누고, 각 단계에 확인 가능한 완료 조건을 붙입니다.
- **다음 명령 안내**: 계획과 진행 기록을 읽어 조사·구현·리뷰 중 지금 필요한 작업을 안내합니다.
- **결정과 결과 기록**: 의도적으로 유지한 설계의 근거, 미해결 항목, 기능 완료 결과를 파일에 남깁니다.

**Phase**는 한 기능을 나눈 구현 단계입니다. 하나의 Phase는 그 자체로 테스트할 수 있고, 한 세션 안에 마칠 수 있는 크기를 목표로 합니다.

| 도구 | 맡는 역할 |
| --- | --- |
| **piedpiper** | 질문 정리, Phase 계획, 다음 명령 안내, 완료 보고 |
| Claude Code **`/deep-research`** | 정리된 질문에 대한 조사 |
| Claude Code **`/goal`** | 지정한 완료 조건을 목표로 구현 진행 |
| **[ponytail](https://github.com/DietrichGebert/ponytail)** | 과잉 설계 억제, 변경 내용 리뷰, 저장소 전체 감사 |

## 설치하기

### 1. 실행 환경 확인

이 저장소에 기록된 검증 환경은 **Claude Code v2.1.236**입니다. 다른 버전의 동작은 확인되지 않았습니다.

터미널에서 버전을 확인합니다.

```bash
claude --version
```

이후 **작업할 프로젝트의 루트 디렉터리에서 Claude Code를 실행**하세요. 아래 `/` 명령은 Claude Code 입력창에 입력합니다.

### 2. 플러그인 설치

**아래 명령은 각각 따로 입력하고 실행이 끝난 뒤 다음 명령을 보내세요.**

piedpiper 마켓플레이스 등록:

```text
/plugin marketplace add devKobe24/piedpiper
```

piedpiper 설치:

```text
/plugin install piedpiper@piedpiper
```

함께 사용하는 ponytail 마켓플레이스 등록:

```text
/plugin marketplace add DietrichGebert/ponytail
```

ponytail 설치:

```text
/plugin install ponytail@ponytail
```

### 3. 명령 표시 확인

Claude Code 입력창에 `/pied`를 입력해 자동완성에서 다음 **4개 스킬**을 확인합니다.

- `piedpiper`
- `piedpiper-clarify`
- `piedpiper-plan`
- `piedpiper-wrap`

또한 `/deep`, `/go`, `/ponytail`을 입력해 필요한 명령이 표시되는지 확인하세요. 이 워크플로는 Claude Code의 `/deep-research`, `/goal`을 사용하므로, 현재 환경에서 두 명령을 사용할 수 있어야 합니다. 두 명령은 piedpiper에 포함된 스킬이 아닙니다.

## 첫 기능 진행하기

처음에는 작은 기능 하나로 시작하는 것을 권합니다. 아래는 새 워크플로를 시작하는 순서입니다. **기존 작업을 이어 갈 때는 `/piedpiper next`부터 실행**하세요.

### 1. 명세를 준비하고 초기화하기

만들 기능을 Markdown으로 작성합니다. 예를 들어 `specs/todo-api.md`에 저장할 수 있습니다. 이미 작성한 명세가 있다면 그 파일의 경로를 사용하면 됩니다.

<details>
<summary>명세 예시: 할 일 목록 API</summary>

```markdown
# 할 일 목록 API

## 목표

할 일을 등록하고 목록으로 조회한다.

## 요구사항

- 할 일은 id, title, completed 값을 가진다.
- 제목을 입력해 등록하며 completed의 초기값은 false다.
- 빈 제목으로 등록하면 HTTP 400을 반환한다.
- 목록은 등록한 순서로 반환한다.

## 기술

- Java 21, Spring Boot 3, Gradle
- 데이터 저장 방식은 구현 전에 결정한다.

## 이번 범위

인증과 사용자 화면은 포함하지 않는다.
```

</details>

조사 단계에서는 ponytail을 끕니다.

```text
/ponytail off
```

새 워크플로를 초기화합니다.

```text
/piedpiper init
```

`CLAUDE.md`가 없으면 명세 경로나 만들려는 기능을 묻습니다. 준비한 경로를 전달하세요. 안내에 따라 `CLAUDE.md` 초안과 `.progress-report/state.md`가 준비되었는지 확인합니다.

### 2. 미결 사항에 답하기

```text
/piedpiper-clarify
```

명세에 없는 결정 사항을 질문으로 정리합니다. 질문을 확인한 뒤 `Q1: A, Q2: B`처럼 답할 수 있습니다. 선택지는 실제로 제시된 질문을 기준으로 고르세요.

답변은 `CLAUDE.md`의 **확정된 결정** 섹션과 `.progress-report/clarify-01.md`에 기록됩니다. 조사가 필요한 항목은 `/deep-research` 명령 초안으로 안내합니다.

### 3. 조사하고 결과를 파일로 저장하기

clarify가 제시한 `/deep-research` 명령을 확인한 뒤 직접 실행합니다. 아래의 `<조사 질문>`은 안내받은 실제 질문으로 바꿉니다.

```text
/deep-research <조사 질문>
```

**조사가 끝나면 보고서 본문을 `.progress-report/research-01.md`에 저장하세요.** 후속 조사는 `research-02.md`처럼 번호를 올립니다. 다음 단계가 조사 결과를 읽을 수 있도록, 파일에 실제 보고서 내용이 들어 있는지 확인합니다.

`research-*.md`가 없거나 제목만 있으면 `/piedpiper next`는 조사가 필요한 상태로 판단합니다. `/piedpiper-plan`도 조사 보고서를 전제로 하므로, 저장을 마친 뒤 진행하세요.

### 4. Phase별 구현 계획 만들기

```text
/piedpiper-plan
```

조사 보고서와 확정된 결정을 바탕으로 `CLAUDE.md`를 기능별 Phase 계획으로 다시 작성합니다. 각 Phase에는 구현 범위와 완료 조건이 들어갑니다.

완료 조건은 다음처럼 결과를 확인할 수 있어야 합니다.

> `TodoControllerTest`가 통과하고, 빈 제목으로 `POST /api/todos`를 요청하면 HTTP 400을 반환할 것.
>
> 위 이름과 경로는 설명용 예시입니다. 실제 계획은 프로젝트에 맞게 생성됩니다.

계획 작성 후에는 Phase 개수와 첫 `/goal` 명령을 안내합니다. `CLAUDE.md`의 기존 프로젝트 규칙과 **확정된 결정**도 함께 확인하세요.

### 5. Phase마다 구현·리뷰·기록 반복하기

구현을 시작하기 전에 ponytail 모드를 전환합니다.

```text
/ponytail full
```

계획 또는 `/piedpiper next`가 제시한 `/goal` 명령을 실행합니다. 아래 자리표시자 대신 **해당 Phase의 완료 조건을 그대로** 사용하세요.

```text
/goal <현재 Phase의 완료 조건>
```

구현이 끝나면 완료 조건에 맞는 동작과 테스트 결과를 확인하고, 변경 내용을 리뷰합니다.

```text
/ponytail-review
```

리뷰 결과를 검토한 뒤 `.progress-report/state.md`에 다음을 기록합니다.

- 해당 Phase의 완료 여부와 확인한 결과
- 리뷰 완료 여부, 적용한 사항과 보류한 사항
- 다음 Phase와 이월할 작업
- 의도적으로 유지한 코드나 설계와 그 이유

그다음 안내를 받습니다.

```text
/piedpiper next
```

남은 Phase가 있으면 같은 과정을 반복합니다. **기능의 마지막 Phase도 리뷰를 마친 뒤** 아래 기능 마감 단계로 넘어갑니다.

### 6. 기능을 마감하고 완료 보고서 남기기

기능의 모든 Phase와 리뷰가 끝났으면 저장소 전체의 코드 감사를 진행합니다.

```text
/ponytail-audit
```

감사 결과를 확인한 뒤 완료 보고서를 작성합니다. 다음은 기능 1을 마감하는 예시입니다.

```text
/piedpiper-wrap 1
```

wrap이 물으면 감사 실행 여부와 결과를 전달하세요. `next`가 바로 wrap을 안내했더라도 **감사를 먼저 마쳐야 합니다.**

wrap은 다음을 수행합니다.

- Phase별 완료 조건과 검증 근거 확인
- 감사 지적, 조치 권고, 의도적으로 유지할 항목, 미해결 debt 정리
- `.progress-report/feature-1-report.md` 작성
- `state.md`에 기능 완료 기록 및 다음 기능의 첫 Phase 안내

감사 지적의 적용 여부는 사용자가 판단합니다. wrap의 작업 범위는 보고서 작성과 상태 갱신이며, 소스 코드 수정은 포함하지 않습니다.

마지막 기능까지 끝났다면 다음 항목을 최종 점검하세요.

- `.progress-report/`의 전체 보고서
- `/ponytail-debt`로 확인하는 미해결 장부
- 원본 명세와 비교한 요구사항 누락
- 코드 리뷰를 통한 정확성 문제

wrap의 최종 안내에는 `/code-review --fix`가 포함됩니다. 이 명령은 piedpiper에 포함되어 있지 않으므로 현재 환경에서 지원하는지 확인하세요. 지원하는 경우 수정 범위를 확인한 뒤 실행하고, 제공되지 않으면 사용하는 코드 리뷰 도구로 정확성을 점검합니다.

## 명령어

| 명령어 | 사용할 때 | 결과 |
| --- | --- | --- |
| `/piedpiper` 또는 `/piedpiper status` | 현재 위치를 확인할 때 | 기능·Phase별 진행 상황, 완료 수, 미해결 debt 개수 요약 |
| `/piedpiper next` | 다음 할 일을 확인할 때 | 판단 근거와 다음에 실행할 명령 안내 |
| `/piedpiper init` | 새 워크플로를 시작할 때 | 필요한 `CLAUDE.md` 초안, 진행 디렉터리와 초기 상태 준비 |
| `/piedpiper reset` | 진행 상태를 초기화할 때 | 지워질 내용을 먼저 보여주고 확인 후 상태 초기화 |
| `/piedpiper-clarify` | 명세의 미결 사항을 정리할 때 | 질문지, 확정된 답변, 조사 질문 초안 |
| `/piedpiper-plan` | 답변과 조사 결과가 준비됐을 때 | 완료 조건을 포함한 Phase 계획과 첫 `/goal` 명령 |
| `/piedpiper-wrap [기능 번호 또는 이름]` | 한 기능의 모든 Phase·리뷰·감사를 마쳤을 때 | 완료 보고서와 기능 단위 상태 갱신 |

`/piedpiper-wrap`의 인자를 생략하면 상태 파일에서 가장 최근에 완료된 기능을 대상으로 삼습니다. 대상을 명확히 지정하려면 번호나 이름을 함께 입력하세요.

`reset`은 진행 상태를 초기화할 때 사용하는 명령입니다. 실행 전 표시되는 삭제 범위를 확인하세요. 다음 단계를 이어 가는 용도로는 `next`를 사용합니다.

## 파일과 진행 기록

아래 경로는 **piedpiper를 사용하는 프로젝트의 루트**를 기준으로 합니다.

| 파일 | 담는 내용 | 작성·갱신 주체 |
| --- | --- | --- |
| `specs/todo-api.md` 등 원본 명세 | 만들 기능과 요구사항 | 사용자 |
| `CLAUDE.md` | 확정된 결정, 기능별 Phase 계획과 완료 조건 | 초기화·clarify·plan 단계 |
| `.progress-report/state.md` | 현재 위치, Phase·리뷰 상태, 이월 항목, 결정 근거 | init이 생성, 사용자가 Phase별 갱신, wrap이 기능 마감 시 갱신 |
| `.progress-report/clarify-01.md` | 질문지와 확정된 답변 | clarify |
| `.progress-report/research-01.md` | 조사 보고서 본문 | 사용자가 조사 결과 저장 |
| `.progress-report/debt.md` | 미뤄 둔 작업 | ponytail 연계 기록 |
| `.progress-report/feature-1-report.md` | 기능 완료 결과와 후속 권고 | wrap |

`CLAUDE.md`에는 계획과 확정된 결정을 두고, 자주 바뀌는 진행 기록은 `.progress-report/`에 모읍니다. 매 턴 읽히는 `CLAUDE.md`에 진행 이력이 계속 쌓이는 것을 줄이기 위한 구분입니다.

**`next`는 `state.md`를 갱신하지 않습니다.** 사용자가 Phase별 상태와 리뷰 결과를 기록해야 다음 단계 판단에 반영됩니다. `/goal`이 구현 중 상태 파일을 수정할 수도 있으므로 Phase가 끝날 때 내용을 확인하세요.

<details>
<summary>state.md 갱신 예시: Phase 1-1의 구현과 리뷰를 마친 경우</summary>

아래는 `init`의 최소 초기 형태를 확장한 기록 예시입니다. 실제 완료 여부와 결정에 맞게 작성하세요.

```markdown
# 진행 상태

## 현재 위치

Phase 1-2 진행 예정

## Phase

| Phase | 상태 | 리뷰 | 날짜 | 메모 |
| --- | --- | --- | --- | --- |
| Phase 1-1 | 완료 | 완료 | YYYY-MM-DD | 빈 제목 등록 시 400 반환과 관련 테스트 확인 |
| Phase 1-2 | 예정 | 미진행 | - | 목록 조회 구현 |

## 이월 항목

(없음)

## 의도적으로 남긴 것

- TodoService는 트랜잭션 경계로 사용하므로 유지한다.
```

</details>

대화에만 남긴 판단은 이후 작업에서 참조되지 않을 수 있습니다. **의도적으로 남긴 것**에 유지 이유를 적으면 `next`의 리뷰 안내와 wrap의 감사 결과 정리에 활용됩니다. wrap은 기존 기록을 보존하고, 해소된 항목은 해소 여부를 표시하도록 설계되어 있습니다.

진행 기록의 Git 포함 여부는 대상 프로젝트의 `.gitignore`에서 결정하세요. 이 플러그인 저장소의 `.gitignore`에는 `.progress-report/`가 제외되어 있지만, 대상 프로젝트의 설정은 별도로 관리됩니다.

## 문제 해결

| 상황 | 확인할 것과 다음 행동 |
| --- | --- |
| `/pied` 자동완성에 스킬이 없어요. | 설치가 완료됐는지 확인하고 Claude Code 세션을 다시 시작해 보세요. piedpiper 스킬은 4개이며, `next`·`init` 등은 그중 `piedpiper`의 인자입니다. |
| `/deep-research` 또는 `/goal`이 없어요. | Claude Code 버전과 현재 환경의 명령 제공 여부를 확인하세요. piedpiper 설치로 두 명령이 추가되지는 않습니다. |
| `next`가 초기화를 안내해요. | 현재 디렉터리의 `CLAUDE.md`와 `.progress-report/state.md`를 확인하세요. 새 워크플로라면 `init`을 실행하고 명세 경로를 전달합니다. |
| clarify가 기존 질문을 다시 보여줘요. | 질문지에 확정된 답변이 없으면 기존 질문의 답변을 기다립니다. 제시된 질문을 기준으로 답변하세요. |
| 조사했는데 다시 조사를 안내해요. | `.progress-report/research-*.md`에 보고서 본문이 저장됐는지 확인하세요. 파일명과 실제 내용이 필요합니다. |
| 리뷰했는데 다시 리뷰를 안내해요. | `state.md`에 해당 Phase의 리뷰 완료 기록이 있는지 확인하세요. `next`는 기록을 자동으로 고치지 않습니다. |
| wrap이 완료 보고서를 만들지 않아요. | 대상 기능에 미완료 Phase가 있는지, `/ponytail-audit`를 먼저 실행했는지 확인하세요. |
| 계획이나 보고서가 파일로 저장되지 않았어요. | 대상 경로와 Claude Code의 파일 쓰기 권한을 확인하고, 생성·갱신 대상 파일이 실제로 저장됐는지 확인하세요. |

## 동작 범위와 검증

- **다음 단계 판단**: `next`는 완료로 기록된 Phase와 관련 소스·테스트 파일의 존재를 대조합니다. 실제 테스트 실행 결과와 완료 조건 충족 여부는 구현·리뷰 단계에서 확인해야 합니다.
- **완료 보고**: wrap은 완료 조건이 코드와 테스트로 뒷받침되는지 확인하고, 충족되지 않은 조건도 보고서에 기록하도록 설계되어 있습니다.
- **실행 방식**: 사용자가 스킬을 호출하고, 안내받은 명령을 확인해 한 번에 하나씩 실행합니다. `/goal`, `/deep-research`, ponytail 계열 명령은 piedpiper가 직접 실행하지 않습니다.
- **ponytail 모드**: 조사에는 `off`, 구현에는 `full`을 권합니다. `ultra`는 필요한 프레임워크 계층까지 축소할 수 있어 기본 흐름에 포함하지 않습니다.

저장소에 기록된 검증 내용은 다음과 같습니다.

| 항목 | 확인한 범위 |
| --- | --- |
| Claude Code | v2.1.236 |
| 예제 프로젝트 | Java 21 + Spring Boot + Gradle 기반 할 일 목록 API |
| 사용 흐름 | 명세 작성부터 기능 하나 완료까지 4개 스킬 동작 확인 |
| 자동화된 테스트·CI | 현재 저장소에 포함되어 있지 않음 |

piedpiper는 Markdown 스킬 4개와 플러그인 설정으로 구성됩니다. Claude Code 및 ponytail의 동작은 설치된 환경과 버전에 영향을 받습니다.

## 피드백과 기여

설치 문제, 다음 단계 안내 오류, 문서 개선 제안은 [Issues](https://github.com/devKobe24/piedpiper/issues)에 남겨주세요. **Claude Code 버전, 실행한 명령, 진행 단계, 기대한 결과와 실제 결과**를 함께 적으면 재현에 도움이 됩니다.

스킬의 실제 지침은 아래 파일에서 확인할 수 있습니다.

- [상태 요약·다음 명령·초기화](skills/piedpiper/SKILL.md)
- [명세 질문 정리](skills/piedpiper-clarify/SKILL.md)
- [Phase 계획](skills/piedpiper-plan/SKILL.md)
- [기능 완료 보고](skills/piedpiper-wrap/SKILL.md)

ponytail은 별도로 설치하는 의존 플러그인이며, 원본 프로젝트의 규칙을 복사해 포함하지 않습니다. 배포 변경 사항은 [Releases](https://github.com/devKobe24/piedpiper/releases)에서 확인할 수 있습니다.

## 라이선스

[MIT](LICENSE)
