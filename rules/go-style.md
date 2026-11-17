---
paths:
  - "**/*.go"
---

# Go 스타일 (Uber Go Style Guide)

Go 코드를 작성·수정할 때 적용하는 기준. 리뷰 절차·중요도·출력 양식은 `go-reviewer` 서브에이전트에 있다.

## 빌드 & 테스트 기본값
표준 Go 모듈 기준 기본값이다. 대상 프로젝트에 Makefile이나 커스텀 스크립트가 있다면 그쪽을 우선한다.
- Build: `go build ./...`
- Test: `go test -v -race ./...`
- Lint: `golangci-lint run`

## 스타일
모든 Go 코드는 **Uber Go Style Guide**를 따른다. 특히:

- **Pointers to Interfaces:** 인터페이스를 포인터(`*Interface`)로 전달하지 않는다.
- **Receivers and Types:** 메서드 리시버(`Value` vs `Pointer`)를 일관되게 쓴다. 구조체에 포인터 리시버가 하나라도 있으면 모두 포인터 리시버로 통일한다.
- **Zero-value Mutexes:** `sync.Mutex`/`sync.RWMutex`는 포인터가 아닌 값으로 선언하고, 복사되지 않게 한다.
- **Error Handling:**
    - 에러 문자열은 소문자로 시작하고 마침표(`.`)로 끝내지 않는다.
    - 에러는 `fmt.Errorf("...: %w", err)` 형식으로 체이닝한다.
- **Slice/Map Initialization:** 크기를 알 수 있으면 `make(map[T]T, hint)` / `make([]T, 0, cap)`으로 용량을 미리 할당한다.
- **Goroutine Lifetimes:** 고루틴(`go func()`)을 만들 때 언제·어떻게 종료되는지(Context 등) 명시해 누수를 막는다.

## 네이밍 (공통 규약의 Go 적용)
CLAUDE.md 공통 네이밍 규칙을 Go 관용구에 맞춰 적용한다:
- **코드 내부** 식별자는 Go 관례를 따른다: unexported는 camelCase, exported는 PascalCase. ("내부 변수=camelCase" 규칙은 unexported 스코프에 해당)
- **외부와 데이터를 주고받는 필드**(JSON/메시지 등)는 와이어 포맷이 **snake_case**여야 한다. Go 필드명 자체는 snake_case로 쓰지 말고 **struct tag로 매핑**한다:
  ```go
  type User struct {
      UserID    int    `json:"user_id"`
      CreatedAt string `json:"created_at"`
  }
  ```
- 필드를 `User_ID`처럼 snake/혼합 케이스로 선언하지 않는다. 태그 없이 exported 필드를 내보내 와이어에 PascalCase가 노출되지 않게 한다.
