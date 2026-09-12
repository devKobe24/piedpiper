# piedpiper

명세 문서 하나에서 기능별 Phase 단위 구현까지 끌고 가는 Claude Code 워크플로.

조사는 `/deep-research`, 완주는 `/goal`, 과잉 설계 억제는
[ponytail](https://github.com/DietrichGebert/ponytail)에 맡기고,
piedpiper는 그 사이를 잇는 절차와 상태 관리를 담당한다.

## 요구사항

- Claude Code v2.1.236에서 동작을 확인했다. 그 이전 버전은 확인하지 않았다.
- [ponytail](https://github.com/DietrichGebert/ponytail) 플러그인
- `/deep-research`와 `/goal`은 Claude Code 기본 내장이라 별도 설치가 없다.
  세션에서 `/deep`, `/go`를 입력해 자동완성에 뜨는지 확인한다.

버전 확인:

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

`/deep-research`와 `/goal`은 Claude Code 기본 내장이라 별도 설치가 없다.
세션에서 `/deep`, `/go`를 입력해 자동완성에 뜨는지 확인한다.

## 명령어

| 명령어               | 하는 일                                     |
| -------------------- | ------------------------------------------- |
| `/piedpiper`         | 현재 진행 상황 요약                         |
| `/piedpiper next`    | 다음에 입력할 명령어를 제시                 |
| `/piedpiper init`    | 상태 파일 생성                              |
| `/piedpiper-clarify` | 명세를 읽고 미결 사항을 질문 리스트로       |
| `/piedpiper-plan`    | 조사 결과로 CLAUDE.md를 Phase 구조로 재작성 |
| `/piedpiper-wrap`    | 기능 완료 보고서 작성                       |

## 사용 흐름

```
# 조사 (ponytail off)
/ponytail off
/piedpiper-clarify
  → 질문에 답변
/deep-research <clarify가 제시한 질문>
/piedpiper-plan

# 구현 (ponytail on) — Phase 수만큼 반복
/ponytail full
/goal <plan이 제시한 완료 조건>
/ponytail-review
/piedpiper next

# 기능 마감
/ponytail-audit
/piedpiper-wrap
```

막히면 언제든 `/piedpiper next`를 치면 된다.

## 왜 자동으로 다 안 하나

세 가지 구조적 제약 때문이다.

- `/goal`은 빌트인 명령어라 스킬 안에서 호출할 수 없다.
  Skill 도구로 접근 가능한 빌트인은 `/init`, `/security-review` 정도로 한정된다.
- `/deep-research`는 사용자가 직접 호출할 때만 실행된다 (v2.1.236 부터).
- 한 메시지에 명령어는 맨 앞 하나만 인식된다.
  스킬은 최대 6개까지 이어붙일 수 있지만, 포크된 서브에이전트로 도는
  스킬이 나오면 거기서 확장이 멈춘다.

그래서 piedpiper는 **실행하는 대신 다음에 칠 명령어를 완성해서 제시한다.**
한 번 더 치는 수고가 들지만, `/goal` 조건은 어차피 사용자가 눈으로
확인하고 승인해야 하는 지점이라 여기서 손이 가는 게 오히려 맞다.

## 파일 레이아웃

```
project/
├── CLAUDE.md                   # Phase 계획 (변하지 않는 것만)
├── specs/
│   └── <명세>.md
└── .progress-report/
    ├── state.md                # 진행 상태
    ├── debt.md                 # ponytail: 로 미뤄둔 항목
    ├── research-01.md          # /deep-research 결과
    └── feature-01-report.md    # 기능 완료 보고서
```

`CLAUDE.md`에는 **계획**만, `.progress-report/`에는 **상태**를 둔다.
`CLAUDE.md`는 매 턴 컨텍스트에 올라가므로 진행 기록을 넣으면
매 요청마다 그 토큰을 내게 된다.

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

## 의존성

ponytail은 룰을 복사해 넣지 않고 별도 플러그인으로 두었다.
MIT라 복사 자체는 가능하지만, 본가가 업데이트되면 룰이 갈라지고
`/ponytail-review` 같은 명령은 어차피 원본이 있어야 동작한다.

## 문제 신고

[Issues](https://github.com/devKobe24/piedpiper/issues)로 남겨주세요.

## 라이선스

MIT
