# piedpiper

명세 문서 하나에서 기능별 Phase 단위 구현까지 끌고 가는 Claude Code 워크플로.

조사는 `/deep-research`, 완주는 `/goal`, 과잉 설계 억제는
[ponytail](https://github.com/DietrichGebert/ponytail)에 맡기고,
piedpiper는 그 사이를 잇는 절차와 상태 관리를 담당한다.

**실행하는 대신 다음에 칠 명령어를 완성해서 제시한다.**
왜 그런지는 [아래](#왜-자동으로-다-안-하나)에 적었다.

## 요구사항

- **Claude Code v2.1.236**에서 동작을 확인했다. 그 이전 버전은 확인하지 않았다.
- [ponytail](https://github.com/DietrichGebert/ponytail) 플러그인
- `/deep-research`와 `/goal`은 Claude Code 기본 내장이라 별도 설치가 없다.
  세션에서 `/deep`, `/go`를 입력해 자동완성에 뜨는지 확인한다.

```bash
claude --version
```

## 설치

```
/plugin marketplace add devKobe24/piedpiper
/plugin install piedpiper@piedpiper
```

두 명령을 각각 따로 보내야 한다.

ponytail도 함께 설치한다.

```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

설치 후 `/pied`를 입력해 명령어 네 개가 뜨는지 확인한다.

## 명령어

| 명령어 | 하는 일 |
|---|---|
| `/piedpiper` | 현재 진행 상황 요약 |
| `/piedpiper next` | 지금 상태를 판단해 다음에 입력할 명령어를 제시 |
| `/piedpiper init` | `.progress-report/`와 상태 파일 생성 |
| `/piedpiper reset` | 상태 파일 초기화 (지울 내용을 먼저 보여주고 확인을 받는다) |
| `/piedpiper-clarify` | 명세를 읽고 미결 사항을 질문 리스트로 만들어 파일에 저장 |
| `/piedpiper-plan` | 조사 결과로 `CLAUDE.md`를 Phase 구조로 재작성 |
| `/piedpiper-wrap` | 기능 완료 보고서 작성 |

막히면 언제든 `/piedpiper next`를 치면 된다.
어느 단계인지 판단해서 그다음에 칠 명령어를 완성해 준다.

## 사용 흐름

```
# 조사 (ponytail off)
/ponytail off
/piedpiper next          → CLAUDE.md 초안 생성
/piedpiper-clarify       → 질문에 답변
/deep-research <clarify가 제시한 질문>
/piedpiper-plan

# 구현 (ponytail on) — Phase 수만큼 반복
/ponytail full
/goal <plan이 제시한 완료 조건>
/ponytail-review
   → state.md 갱신 (직접)
/piedpiper next

# 기능 마감
/ponytail-audit
/piedpiper-wrap
```

## 무엇을 해 주는가

### 완료 조건을 판정 가능하게 만든다

`/piedpiper-plan`은 각 Phase에 **기계적으로 판정 가능한** 완료 조건을 붙인다.
`/goal`이 그 조건을 그대로 받아 판정하므로, 서술형이면 일찍 종료될 여지가 생긴다.

| 쓰지 않는 것 | 쓰는 것 |
|---|---|
| 로그인을 구현할 것 | `AuthControllerTest`가 전부 통과하고 `POST /login`이 유효 자격증명에 200과 토큰을 반환할 것 |
| API를 개선할 것 | `ProductServiceTest`가 통과하고 조회 경로에서 N+1 쿼리가 발생하지 않을 것 |

`/piedpiper next`는 이 조건을 **한 글자도 바꾸지 않고** `/goal` 뒤에 붙여 제시한다.

### 기록과 실제를 대조한다

`state.md`는 사람이 손으로도 고치는 파일이라 실제 코드와 어긋날 수 있다.
`/piedpiper next`는 분기 판단 전에 완료로 기록된 Phase가 정말 끝났는지
소스와 테스트 파일로 대조하고, 어긋나면 **기록이 아니라 실제를 기준으로** 판단한다.

### 결정 근거를 다음 단계로 넘긴다

`/ponytail-review`는 별도 세션이라 이전 Phase의 결정 근거를 보지 못한다.
그래서 의도적으로 남긴 코드를 반복해서 지적한다.

`state.md`에 `## 의도적으로 남긴 것` 섹션을 두면
`/piedpiper next`가 리뷰를 안내할 때 그 내용을 함께 보여주고,
`/piedpiper-wrap`이 감사 결과와 대조해 "적용하지 말 것"으로 분류한다.

## 왜 자동으로 다 안 하나

세 가지 구조적 제약 때문이다.

- `/goal`은 빌트인 명령어라 스킬 안에서 호출할 수 없다.
  Skill 도구로 접근 가능한 빌트인은 `/init`, `/security-review` 정도로 한정된다.
- `/deep-research`는 사용자가 직접 호출할 때만 실행된다.
  이전 버전에서는 모델이 스스로 시작할 수도 있었다.
- 한 메시지에 명령어는 맨 앞 하나만 인식된다.
  스킬은 최대 6개까지 이어붙일 수 있지만, 포크된 서브에이전트로 도는
  스킬이 나오면 거기서 확장이 멈춘다.

한 번 더 치는 수고가 들지만, `/goal` 조건은 어차피 사용자가 눈으로
확인하고 승인해야 하는 지점이라 여기서 손이 가는 게 오히려 맞다.

## 파일 레이아웃

```
project/
├── CLAUDE.md                   # Phase 계획 (변하지 않는 것만)
├── specs/
│   └── <명세>.md               # 사용자가 쓴 원본 명세
└── .progress-report/
    ├── state.md                # 진행 상태
    ├── clarify-01.md           # clarify 질문지 + 확정된 답변
    ├── research-01.md          # /deep-research 결과
    ├── debt.md                 # ponytail: 로 미뤄둔 항목
    └── feature-1-report.md     # 기능 완료 보고서
```

`CLAUDE.md`에는 **계획**만, `.progress-report/`에는 **상태**를 둔다.
`CLAUDE.md`는 매 턴 컨텍스트에 올라가므로 진행 기록을 넣으면
매 요청마다 그 토큰을 내게 된다.

### state.md 구조

```markdown
# 진행 상태

## 현재 위치
Phase 1-2 진행 예정

## Phase
| Phase | 상태 | 날짜 | 메모 |
|---|---|---|---|

## 이월 항목
(다음 Phase에서 함께 처리할 것)

## 의도적으로 남긴 것
(리뷰가 지적하더라도 유지하기로 한 것과 그 근거)
```

`## 의도적으로 남긴 것`은 선택이지만, 있으면 리뷰 왕복이 줄어든다.

## ponytail 모드 전환

Phase 경계에서 직접 치는 게 가장 확실하다.
세션 단위로 고정하려면 터미널을 나눠도 된다.

```bash
PONYTAIL_DEFAULT_MODE=off claude    # 조사용
PONYTAIL_DEFAULT_MODE=full claude   # 구현용
```

`ultra`는 권하지 않는다. ponytail 벤치마크는 FastAPI + React 레포 기준이고,
큰 감소가 난 사례는 네이티브 기능으로 대체 가능한 프론트엔드 과잉 설계였다.
Spring처럼 프레임워크가 이미 많은 것을 해주는 영역에서는 그 이득이 적고,
대신 컨벤션상 필요한 계층까지 깎일 위험이 커진다.

## 알려진 한계

**상태 파일은 사용자가 관리한다.**
`/piedpiper next`는 `state.md`를 읽기만 하고 고치지 않는다.
반면 `/goal`은 빌트인이라 piedpiper가 행동을 제약할 수 없어서,
구현 중에 `state.md`를 건드릴 수 있다. Phase마다 내용을 확인하는 편이 안전하다.

**Phase 사이의 맥락은 파일로만 넘어간다.**
`/goal`, `/ponytail-review`, `/piedpiper-wrap`은 서로 다른 세션에서 돈다.
대화에만 남긴 판단은 다음 단계로 넘어가지 않으므로,
유지할 이유가 있는 결정은 `state.md`에 적어야 한다.

**조사 단계는 대화형이 아니다.**
`/deep-research`는 질문 하나를 받아 단발 보고서를 내는 백그라운드 워크플로다.
질문을 만드는 일은 `/piedpiper-clarify`가 맡는다.

## 검증

실제 Spring Boot 프로젝트(Java 21 + Gradle, 할 일 목록 API)로
명세 작성부터 기능 하나 완료까지 한 바퀴 돌려 각 스킬의 동작을 확인했다.
그 과정에서 발견한 문제는 수정해 반영했다.

## 의존성

ponytail은 룰을 복사해 넣지 않고 별도 플러그인으로 두었다.
MIT라 복사 자체는 가능하지만, 본가가 업데이트되면 룰이 갈라지고
`/ponytail-review` 같은 명령은 어차피 원본이 있어야 동작한다.

## 문제 신고

[Issues](https://github.com/devKobe24/piedpiper/issues)로 남겨주세요.

## 라이선스

MIT
