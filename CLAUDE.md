# 전역 지침 (Global Instructions)

매 세션 자동 로드되는 핵심 지침. 상세 절차는 `rules/`와 `skills/`로 분리되어 있다.

## 기본 동작

- 응답은 **한국어**로 한다. 기술 용어/식별자는 원문(영어) 유지.
- 코드를 다룰 때는 해당 언어의 **10년 차 이상 숙련 엔지니어** 관점으로 임한다.
- 추측하지 않는다. 모든 서술은 실제 코드/설정 파일 근거 기반으로 하고, 근거 파일 경로를 명시한다.
- 불확실하거나 죽은 코드로 의심되는 부분은 단정하지 말고 `⚠️ 확인 필요`로 표시한다.

## 공통 네이밍 규칙 (모든 언어 공통)

모든 언어의 코드 리뷰·작성 시 아래 네이밍 규칙을 적용한다. 언어별 구체적 적용법은 각 `rules/*-code-review.md`에 명시되어 있다.

- **코드 내부에서 쓰는 변수·필드** → **camelCase** (각 언어 관례 우선. 예: Go의 exported 식별자는 PascalCase)
- **API 연동 등 외부와 데이터를 송수신하는 필드** → **snake_case**
  - 단, 언어 식별자 규칙을 깨면서 필드명을 직접 snake_case로 쓰지 말고 **직렬화 경계에서 매핑**한다 (Go struct tag, Java `@JsonProperty`/`@JsonNaming`, TS 외부 DTO 타입 분리 등).

## 룰 라우팅 (Rules)

`rules/` 폴더는 Claude Code가 자동 탐색하지 않으므로, 아래 조건에 해당하면 **해당 파일을 직접 읽고 그 지침을 따른다.**

- **Go 코드 전체 리뷰/검수/리팩토링** 요청 → `rules/go-code-review.md` (Uber Go Style Guide)
- **Java 코드 전체 리뷰/검수/리팩토링** 요청 → `rules/java-code-review.md` (Google Java Style + Effective Java)
- **TypeScript 코드 전체 리뷰/검수/리팩토링** 요청 → `rules/typescript-code-review.md` (typescript-eslint + Airbnb/Google TS)
- **Git Diff / 특정 커밋 / PR 변경분 리뷰** 요청 → `rules/go-diff-review.md`
  - 변경된 라인(`+`)과 인접 컨텍스트만 검수. 전체 파일 분석 금지. (현재 Go 전용)

모든 리뷰는 지적을 🔴 CRITICAL / 🟡 WARNING / 🔵 SUGGESTION 으로 분류하고, 위 공통 네이밍 규칙을 함께 적용한다.

## 스킬 (Skills)

`skills/`는 Claude Code가 `description`을 보고 필요할 때 자동 호출한다. 수동 안내가 필요한 경우 목록:

- `legacy-analysis` — 레거시 코드베이스 구조·배포 분석 문서화 (Mermaid)
- `pr-writer` — GitHub PR/Issue 본문 작성
- `dev-summary` — 대화·작업 내용 개발 문서형 요약본
- `troubleshooting-report` — 에러 디버깅 과정을 트러블슈팅 리포트로 정리

## 산출물 규칙

- 코드 리뷰 결과는 `YYYYMMDD-code-review.md` 형태로 저장한다.
- 레거시 분석 산출물은 `docs/analysis/` 하위에 생성한다.
- 다이어그램은 마크다운에서 바로 렌더링되도록 **Mermaid**로 작성한다.
