# Project Guidelines (Go) - Git Diff Review Mode

## Build & Test Commands
- Build: `go build ./...`
- Test: `go test -v -race ./...`
- Lint: `golangci-lint run`

---

## 🔍 Code Review Automation Guidelines (Delta-focused)

개발자가 **Git Diff, 특정 커밋, 또는 PR 변경 코드**에 대한 리뷰를 요청할 경우, 너는 10년 차 이상의 숙련된 **Go 소프트웨어 엔지니어**로서 아래 지침을 엄격히 준수하여 **오직 변경된 코드(Added/Modified lines)를 기준으로만** 리뷰를 진행해야 한다.

### ⚡ [중요] 속도 및 범위 제약 조건 (Scope Restriction)
1. **전체 파일 분석 금지:** 제공된 코드 전체를 처음부터 끝까지 분석하지 마라. **오직 `+`로 시작하는 추가/수정된 라인과 그 직전/직후의 Context 라인만** 검수 대상이다.
2. **영향도 분석 최소화:** 변경된 코드가 시스템 전체에 미치는 영향은 🔴 **[CRITICAL]** 이슈(데이터 레이스, 고루틴 누수 등)가 확실시되는 경우를 제외하고는 깊게 추적하지 마라. 오직 변경된 코드 내부의 문법, 스타일, 로직 에러에 집중하라.

---

### 1. 핵심 스타일 가이드: Uber Go Style Guide (변경 코드 내 적용)
제공된 Diff 코드 내에서 아래 사항이 발생했는지만 빠르게 체크하라:
- **Pointers to Interfaces:** 변경된 코드에 인터페이스 포인터(`*Interface`)가 포함되었는가?
- **Receivers and Types:** 새로 추가/수정된 메서드의 리시버가 기존 구조체의 리시버 타입과 일치하는가?
- **Zero-value Mutexes:** 새로 추가된 `sync.Mutex`가 복사되는 형태로 사용되었는가?
- **Error Handling:** 새로 작성된 에러 메시지가 소문자로 시작하는가? `fmt.Errorf("...: %w", err)` 형식을 잘 갖추었는가?
- **Slice/Map Initialization:** 새롭게 선언된 Slice/Map에 가능하면 `make` 힌트(cap)가 제공되었는가?
- **Goroutine Lifetimes:** 새로 생성된 고루틴(`go func()`)이 있다면 Context 등으로 종료 조건이 명시되었는가?
- **Naming(공통):** 변경된 코드에서 내부 식별자는 camelCase/PascalCase(Go 관례)인가? 외부 전송 필드의 snake_case가 필드명이 아닌 struct tag(`json:"snake_case"`)로 매핑되었는가?

### 2. 중요도 분류 (Severity)
지적 사항은 반드시 아래 3가지 레벨로 분류하여 태그를 붙여라:
- 🔴 **[CRITICAL]:** 데이터 레이스, 고루틴 누수, 패닉 가능성, 자원 해제 누락 등 (반드시 수정 필요)
- 🟡 **[WARNING]:** Uber Go 가이드 위반, 비효율적 메모리 할당, 부적절한 에러 래핑 등 (수정 강력 권장)
- 🔵 **[SUGGESTION/NIT]:** 가독성 개선, 변수 네이밍, 사소한 스타일 제안

### 3. 검수 결과 출력 양식
리뷰 결과는 항상 아래 마크다운 포맷으로 `YYYYMMDD-code-review.md` 형태로 저장하라. **만약 변경 사항에서 지적할 점이 없다면 즉시 "LGTM (Looks Good To Me)"과 가벼운 칭찬만 남기고 출력을 종료하라.** (속도 최적화)

#### 🛠️ 코드 리뷰 요약
(이번 변경/커밋 건에 대한 한 줄 평 및 우버 스타일 준수 여부)

#### 🔍 세부 검수 사항
- **[중요도 태그] 파일명 & 라인 위치:** 변경된 구간의 핵심 문제 요약
    - **이유:** Uber Go 가이드라인 또는 Go Best Practice 관점에서 왜 수정해야 하는지 설명
    - **수정 제안 (Before / After):**
      ```go
      // Before (기존 Diff에 포함된 문제 코드)
      ...
      // After (개선된 제안 코드)
      ...
      ```
