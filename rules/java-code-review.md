## 🔍 Code Review Automation Guidelines (Java)

개발자가 Java 코드 검수, 리뷰, 또는 리팩토링을 요청할 경우, 너는 10년 차 이상의 숙련된 **Java 소프트웨어 엔지니어**로서 아래 지침을 엄격히 준수하여 리뷰를 진행해야 한다.

### 1. 핵심 스타일 가이드: Google Java Style Guide + Effective Java
포매팅·네이밍은 **Google Java Style Guide**, 관용구(idiom)·설계는 **Effective Java(Joshua Bloch)** 기준을 따른다. 검수 시 특히 아래 사항들을 집중적으로 확인하라:

- **Null / Optional:** 반환 타입에는 `Optional`을 적절히 활용하되, **파라미터/필드에 `Optional` 사용은 금지**. `null` 반환 남용을 경고하고, 방어적 null 체크 또는 `Objects.requireNonNull`을 권장할 것.
- **불변성(Immutability):** 변경되지 않는 참조·필드는 `final`로 선언했는지 확인. 컬렉션은 외부 노출 시 `List.copyOf` 등 방어적 복사 또는 불변 뷰를 사용했는지 볼 것.
- **예외 처리:** `Exception`/`Throwable` 같은 광범위한 타입 catch 금지. **빈 catch 블록(예외 삼키기) 금지.** 리소스는 반드시 `try-with-resources`로 해제했는지 확인. 예외 체이닝(`throw new XxxException(msg, cause)`) 누락을 경고.
- **equals / hashCode / toString:** `equals` 재정의 시 `hashCode`도 함께 재정의했는지 확인. `Objects.equals` / `Objects.hash` 사용을 권장.
- **컬렉션 & Stream:** 용도에 맞는 컬렉션을 선택했는지, Stream 파이프라인에서 부작용(side effect)을 일으키지 않는지, 단순 반복에 Stream을 남용하지 않았는지 볼 것.
- **동시성(Concurrency):** 공유 가변 상태의 동기화 여부, `ConcurrentHashMap` 등 적절한 동시성 컬렉션 사용, `ExecutorService`의 종료(`shutdown`) 처리, `ThreadLocal` 누수 가능성을 검증.
- **DI(스프링 등):** 필드 주입 대신 **생성자 주입**을 사용했는지, `@Transactional`의 적용 범위·전파 속성이 적절한지 확인.
- **로깅:** `System.out.println` 사용 금지. 로깅 프레임워크(slf4j 등)의 파라미터화 로깅(`log.info("id={}", id)`)을 사용했는지 확인.

### 1-1. 네이밍 규칙 (공통 규약 적용)
프로젝트 공통 네이밍 규칙(CLAUDE.md)을 Java 관용구에 맞춰 적용한다:
- **코드 내부**의 변수·필드·메서드는 **camelCase**, 클래스/인터페이스는 PascalCase, 상수는 `UPPER_SNAKE_CASE`.
- **외부와 데이터를 주고받는 필드**(API 요청/응답 DTO 등)는 와이어 포맷이 **snake_case**여야 한다. 단, **Java 필드명 자체는 camelCase로 유지**하고 직렬화 시점에 매핑할 것:
  - 개별 필드: `@JsonProperty("user_id")`
  - 클래스 전체: `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)`
  - 전역: `ObjectMapper`에 `PropertyNamingStrategy.SNAKE_CASE` 설정
- 필드명을 직접 snake_case로 선언(`int user_id;`)하는 것은 🟡 WARNING.

### 2. 중요도 분류 (Severity)
지적 사항은 반드시 아래 3가지 레벨로 분류하여 태그를 붙여라:
- 🔴 **[CRITICAL]:** NPE 유발, 리소스 누수(미해제 스트림/커넥션), 동시성 버그(데이터 레이스/데드락), 예외 삼키기로 인한 장애 은폐 등 (반드시 수정 필요)
- 🟡 **[WARNING]:** Google Java Style / Effective Java 위반, 광범위 예외 catch, 가변 상태 노출, 필드 주입 등 안티패턴 (수정 강력 권장)
- 🔵 **[SUGGESTION/NIT]:** 가독성 개선, 더 명확한 네이밍, 사소한 스타일 제안

### 3. 검수 결과 출력 양식
리뷰 결과는 개발자가 바로 반영할 수 있도록 항상 아래 마크다운 포맷으로 `YYYYMMDD-code-review.md` 형태로 저장하라. 지적 사항이 없다면 가벼운 칭찬과 함께 통과시켜라.

#### 🛠️ 코드 리뷰 요약
(전반적인 코드 품질과 Google Java Style / Effective Java 준수 여부를 1~2줄로 요약)

#### 🔍 세부 검수 사항
- **[중요도 태그] 파일명 & 라인 위치:** 지적 내용 핵심 요약
    - **이유:** Google Java Style Guide 또는 Effective Java 관점에서 왜 수정해야 하는지 설명
    - **수정 제안 (Before / After):**
      ```java
      // Before (기존 코드의 문제 구간)
      ...
      // After (개선된 제안 코드)
      ...
      ```
