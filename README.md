# ai-assisted-playbook

Claude Code 전역 프롬프트/지침 백업. 스킬·서브에이전트는 **플러그인(`playbook`)**으로, `CLAUDE.md`·`rules/`는 `~/.claude/`에 심링크해서 사용한다.

## 구조

```
.
├── .claude-plugin/
│   ├── plugin.json                # 플러그인 매니페스트 (name: playbook)
│   └── marketplace.json           # 이 레포 자체를 마켓플레이스로 등록
├── skills/<name>/SKILL.md         # [플러그인] 메인 대화 안에서 실행되는 작업 절차 → /playbook:<name>
├── agents/                        # [플러그인] 서브에이전트 (별도 컨텍스트에서 실행 후 결과만 반환)
│   ├── go-reviewer.md             #  - Go 전체 리뷰 + Diff/커밋/PR 리뷰
│   ├── java-reviewer.md           #  - Java 전체 리뷰 + Diff/커밋/PR 리뷰
│   ├── ts-reviewer.md             #  - TypeScript 전체 리뷰 + Diff/커밋/PR 리뷰
│   ├── legacy-analysis.md         #  - 레거시 코드베이스 구조·배포 분석 → docs/analysis/
│   └── version-upgrade-history.md #  - As-Is → To-Be 릴리즈 노트·Breaking Changes 조사
├── CLAUDE.md                      # [심링크] 매 세션 자동 로드되는 핵심 지침
└── rules/                         # [심링크] 언어별 스타일 (paths 매칭 시 자동 로드)
    ├── go-style.md
    ├── java-style.md
    └── typescript-style.md
```

### 자동 로딩 메커니즘

| 위치 | 연동 방식 | Claude Code 동작 |
| --- | --- | --- |
| `CLAUDE.md` | 심링크 | 세션 시작 시 항상 로드 (플러그인은 CLAUDE.md를 로드하지 않음) |
| `rules/` | 심링크 | user-level rule. 각 파일의 `paths` frontmatter에 맞는 파일을 다룰 때 로드 (플러그인은 rules를 지원하지 않음). 서브에이전트도 해당 파일을 읽으면 로드됨 |
| `skills/*/SKILL.md` | 플러그인 | `description`에 맞는 요청이면 자동 호출. 직접 호출은 `/playbook:<name>` |
| `agents/*.md` | 플러그인 | `description`에 맞는 요청이면 자동 위임. 직접 호출은 `@agent-playbook:<name>` |

### 스킬 vs 에이전트 기준

| | 스킬 (`skills/`) | 에이전트 (`agents/`) |
| --- | --- | --- |
| 실행 위치 | 메인 대화 컨텍스트 | 별도 컨텍스트 (메인 대화를 볼 수 없음) |
| 맞는 작업 | **지금까지의 대화 맥락**이 필요한 작업 (예: `pr-writer`, `dev-summary`) | 입력이 명확하고, 탐색·조사량이 많아 **결과만 받으면 되는** 작업 (리뷰, 레거시 분석, 릴리즈 노트 조사) |
| 넘길 정보 | 없음 (대화를 그대로 봄) | 위임 프롬프트에 필요한 입력을 명시해야 함 (아래 "사용법") |

## 사용법

### 에이전트

자연어로 요청하면 `description`을 보고 자동 위임된다. 확실히 특정 에이전트로 보내려면 `@agent-playbook:<name>`으로 멘션한다.

| 에이전트 | 요청 예시 | 함께 줄 정보 | 결과 |
| --- | --- | --- | --- |
| `go-reviewer` / `java-reviewer` / `ts-reviewer` | "현재 diff 리뷰해줘", "`abc123` 커밋 리뷰", "`main` 대비 이 브랜치 리뷰", "`<패키지 경로>` 전체 리뷰" | 리뷰 대상: 파일·패키지 경로, 또는 커밋 SHA / base 브랜치 | `docs/reviews/YYYYMMDD-code-review.md` 저장 + 중요도별 개수·🔴 요약 반환 |
| `legacy-analysis` | "이 프로젝트 분석해줘", "온보딩 문서 만들어줘" | (선택) 분석 대상 하위 경로, 특히 볼 영역 | `docs/analysis/00~06-*.md` 생성 + 구조 요약·"여기부터 보세요"·`⚠️ 확인 필요` 목록 반환 |
| `version-upgrade-history` | "Wowza 4.8.18 → 4.9.7 변경 이력 정리해줘" | 대상 소프트웨어, As-Is / To-Be 버전 (As-Is 생략 시 의존성 파일에서 탐색) | 버전별 Changelog·Breaking Changes 리포트 반환 (출처 URL 포함) |

- 리뷰 에이전트는 **대상 코드를 수정하지 않는다** (`Edit` 도구 없음).
- 여러 언어가 섞인 변경이면 언어별 리뷰 에이전트에 각각 위임된다 (CLAUDE.md "코드 리뷰 위임").

### 스킬

`/playbook:<name>`으로 호출하거나 자연어로 요청한다. 입력창에 `/playbook:`을 치면 목록이 나온다.

- `pr-writer`는 대화 맥락(문제 현상·논의 내용)을 근거로 쓰므로 **작업한 대화 안에서** 호출한다.

### 설치 확인

- `/agents` → 플러그인 에이전트 5개가 `playbook:` 이름으로 보이는지
- 입력창에 `@agent-playbook:` → 에이전트만, `/playbook:` → 스킬만 자동완성됨
- 셸에서 `claude plugin list` → `playbook@ai-assisted-playbook`이 `enabled`, `Read from:`이 이 레포 경로인지

## 설치

```bash
git clone https://github.com/dev-jiemu/ai-assisted-playbook.git ~/ai-assisted-playbook
cd ~/ai-assisted-playbook

# 1) CLAUDE.md, rules 심링크 (기존 파일 있으면 백업 먼저)
ln -sfn "$PWD/CLAUDE.md" ~/.claude/CLAUDE.md
ln -sfn "$PWD/rules"     ~/.claude/rules

# 2) 스킬·에이전트는 플러그인으로 설치
claude plugin marketplace add "$PWD"
claude plugin install playbook@ai-assisted-playbook
```

> `ln -sfn`의 `-n`: 대상(`~/.claude/rules`)이 이미 디렉터리 심링크면 그 안에 링크를 또 만들지 않고 링크 자체를 교체한다.
> (`-n` 없이 재실행하면 `rules/rules` 같은 자기참조 링크가 생긴다.)

> 로컬 경로로 추가한 마켓플레이스의 플러그인은 **이 레포에서 바로 읽힌다** (캐시 복사본이 아님).
> 레포를 옮기거나 지우면 플러그인이 깨지므로, 옮겼다면 `marketplace add`부터 다시 한다.

### 기존 심링크 방식에서 전환하는 경우

예전처럼 `~/.claude/skills`가 이 레포의 `skills/`를 가리키는 심링크라면, 스킬이 `/pr-writer`와 `/playbook:pr-writer`로 두 번 뜬다. 심링크를 실제 폴더로 바꾼다:

```bash
rm ~/.claude/skills        # 심링크만 삭제 (끝에 / 붙이거나 -r 쓰지 말 것)
mkdir ~/.claude/skills     # claude.ai 동기화 스킬(synced/)이 여기에 생성됨
```

## 수정 후 반영

| 수정 대상 | 반영 시점 |
| --- | --- |
| `skills/`, `agents/` | 새 세션 시작 또는 세션 안에서 `/reload-plugins`. commit·버전 변경·`plugin update` 불필요 |
| `CLAUDE.md`, `rules/` | 새 세션 시작 시 |

새 스킬·에이전트를 추가했거나 frontmatter가 의심되면 먼저 검증한다:

```bash
claude plugin validate ~/ai-assisted-playbook
```

> `version` 미지정, 루트 `CLAUDE.md` 미로드, LICENSE/icon 누락 경고는 이 구성에서 의도된 것이라 무시해도 된다.
