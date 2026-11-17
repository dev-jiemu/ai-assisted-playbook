---
paths:
  - "**/*.java"
---

# Java 스타일 (Google Java Style Guide + Effective Java)

Java 코드를 작성·수정할 때 적용하는 기준. 리뷰 절차·중요도·출력 양식은 `java-reviewer` 서브에이전트에 있다.

## 스타일
포매팅·네이밍은 **Google Java Style Guide**, 관용구(idiom)·설계는 **Effective Java(Joshua Bloch)**를 따른다. 특히:

- **Null / Optional:** 반환 타입에는 `Optional`을 적절히 쓰되, **파라미터/필드에 `Optional`은 쓰지 않는다**. `null` 반환을 남용하지 않고, 방어적 null 체크 또는 `Objects.requireNonNull`을 쓴다.
- **불변성(Immutability):** 변경되지 않는 참조·필드는 `final`로 선언한다. 컬렉션을 외부에 노출할 때는 `List.copyOf` 등 방어적 복사 또는 불변 뷰를 쓴다.
- **예외 처리:** `Exception`/`Throwable` 같은 광범위한 타입을 catch하지 않는다. **빈 catch 블록(예외 삼키기) 금지.** 리소스는 `try-with-resources`로 해제한다. 예외는 체이닝(`throw new XxxException(msg, cause)`)한다.
- **equals / hashCode / toString:** `equals`를 재정의하면 `hashCode`도 함께 재정의한다. `Objects.equals` / `Objects.hash`를 쓴다.
- **컬렉션 & Stream:** 용도에 맞는 컬렉션을 고른다. Stream 파이프라인에서 부작용(side effect)을 일으키지 않고, 단순 반복에 Stream을 남용하지 않는다.
- **동시성(Concurrency):** 공유 가변 상태는 동기화하고, `ConcurrentHashMap` 등 적절한 동시성 컬렉션을 쓴다. `ExecutorService`는 `shutdown`하고, `ThreadLocal` 누수를 막는다.
- **DI(스프링 등):** 필드 주입 대신 **생성자 주입**을 쓴다. `@Transactional`의 적용 범위·전파 속성을 의도에 맞게 지정한다.
- **로깅:** `System.out.println` 금지. 로깅 프레임워크(slf4j 등)의 파라미터화 로깅(`log.info("id={}", id)`)을 쓴다.

## 네이밍 (공통 규약의 Java 적용)
CLAUDE.md 공통 네이밍 규칙을 Java 관용구에 맞춰 적용한다:
- **코드 내부**의 변수·필드·메서드는 **camelCase**, 클래스/인터페이스는 PascalCase, 상수는 `UPPER_SNAKE_CASE`.
- **외부와 데이터를 주고받는 필드**(API 요청/응답 DTO 등)는 와이어 포맷이 **snake_case**여야 한다. **Java 필드명 자체는 camelCase로 유지**하고 직렬화 시점에 매핑한다:
  - 개별 필드: `@JsonProperty("user_id")`
  - 클래스 전체: `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)`
  - 전역: `ObjectMapper`에 `PropertyNamingStrategy.SNAKE_CASE` 설정
- 필드명을 직접 snake_case로 선언(`int user_id;`)하지 않는다.
