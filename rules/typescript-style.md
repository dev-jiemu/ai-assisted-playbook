---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# TypeScript 스타일 (typescript-eslint recommended + Airbnb/Google TS)

TypeScript 코드를 작성·수정할 때 적용하는 기준. 리뷰 절차·중요도·출력 양식은 `ts-reviewer` 서브에이전트에 있다.

## 스타일
`tsconfig`의 **strict 모드**를 전제로, **typescript-eslint recommended** 룰과 범용 관용구(Airbnb / Google TypeScript Style)를 따른다. 특히:

- **타입 안전성:** `any` 금지(필요 시 `unknown` 후 좁히기). 타입 단언(`as`)과 비-널 단언(`!`)을 남용하지 않는다. 암묵적 `any`가 새어나오지 않게 한다.
- **null / undefined:** 옵셔널 체이닝(`?.`)과 nullish 병합(`??`)을 쓴다. `||`로 인한 falsy 오작동(`0`, `""`)을 피한다.
- **비동기(Promise):** **floating promise 금지** — 모든 Promise는 `await` 하거나 명시적으로 `void` 처리한다. `async` 함수의 에러 처리(try/catch)를 빠뜨리지 않고, 병렬화 가능한 구간은 `Promise.all`을 쓴다.
- **불변성:** `let`보다 `const`를 우선하고, 변경되지 않는 구조는 `readonly` / `as const`를 적용한다.
- **타입 설계:** `interface`/`type`을 일관되게 쓴다. 상태 구분이 필요한 곳은 **판별 유니온(discriminated union)**을 쓰고, 무분별한 `enum`보다 union literal(`'A' | 'B'`)을 고려한다. `switch`는 `never`로 exhaustiveness를 보장한다.
- **동등성/문법:** `===`/`!==`를 쓴다(`==` 금지). 사용하지 않는 변수·임포트를 남기지 않고, 순환 의존(circular import)을 만들지 않는다.
- **(React인 경우):** hook 의존성 배열을 정확히 하고, 불필요한 리렌더와 key 누락을 피한다.

## 네이밍 (공통 규약의 TS 적용)
CLAUDE.md 공통 네이밍 규칙을 TS 관용구에 맞춰 적용한다:
- **코드 내부**의 변수·함수·프로퍼티는 **camelCase**, 타입/클래스/인터페이스/enum은 PascalCase, 상수는 `UPPER_SNAKE_CASE` 또는 camelCase.
- **외부와 데이터를 주고받는 필드**(API 요청/응답 DTO 등)는 백엔드 계약에 맞춰 **snake_case**를 쓴다. 권장 패턴:
  - 외부 DTO 타입(`interface UserResponse { user_id: number }`)과 **내부 도메인 모델(camelCase)을 분리**하고, 매핑/어댑터 레이어에서 변환.
  - 또는 직렬화 경계(axios interceptor, fetch wrapper 등)에서 camel ↔ snake 자동 변환.
- 내부 도메인 로직에서 snake_case 프로퍼티를 직접 들고 다니지 않는다(경계 레이어 제외).
