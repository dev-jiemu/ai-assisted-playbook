# ai-assisted-playbook

Claude Code 전역 프롬프트/지침 백업. `~/.claude/`에 연동해서 사용한다.

## 구조

```
.
├── CLAUDE.md                      # 매 세션 자동 로드되는 핵심 지침
├── rules/                         # 조건부로 적용되는 지침 (CLAUDE.md가 라우팅)
│   ├── go-code-review.md          #  - Go 코드 전체 리뷰
│   ├── go-diff-review.md          #  - Git Diff / PR 변경분 리뷰 (Go)
│   ├── java-code-review.md        #  - Java 코드 전체 리뷰
│   └── typescript-code-review.md  #  - TypeScript 코드 전체 리뷰
└── skills/                        # 필요할 때 불러오는 작업 절차
    ├── legacy-analysis/SKILL.md   #  - 레거시 코드베이스 분석 문서화
    ├── pr-writer/SKILL.md         #  - GitHub PR/Issue 본문 작성
    ├── dev-summary/SKILL.md       #  - 작업 내용 요약본
    └── troubleshooting-report/SKILL.md  # - 트러블슈팅 리포트
```

### 자동 로딩 메커니즘

| 위치 | Claude Code 동작 |
| --- | --- |
| `CLAUDE.md` | 세션 시작 시 항상 로드 |
| `skills/*/SKILL.md` | frontmatter의 `description`을 보고 관련 요청일 때 자동 호출 |
| `rules/` | **자동 탐색 안 됨.** `CLAUDE.md`의 라우팅에 따라 필요 시 직접 읽음 |

## 전역 폴더 연동

이 레포를 `~/.claude/`에 심링크한다.

```bash
git clone https://github.com/dev-jiemu/ai-assisted-playbook.git
cd ai-assisted-playbook

# CLAUDE.md (기존 파일 있으면 백업 먼저)
ln -sf "$PWD/CLAUDE.md"  ~/.claude/CLAUDE.md

# skills (개별 스킬을 ~/.claude/skills/ 아래로 연동)
mkdir -p ~/.claude/skills
ln -sf "$PWD"/skills/*   ~/.claude/skills/

# rules (CLAUDE.md가 상대경로로 참조하므로 함께 연동)
ln -sf "$PWD/rules"      ~/.claude/rules
```

> 참고: `CLAUDE.md`가 `rules/go-code-review.md`를 상대경로로 가리키므로,
> `rules/`도 `~/.claude/` 아래에 같이 올라와 있어야 라우팅이 동작한다.
