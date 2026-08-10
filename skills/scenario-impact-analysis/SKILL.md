---
name: scenario-impact-analysis
description: 특정 함수, 엔드포인트, 또는 특정 필드 값이 들어왔을 때 시간순 데이터 흐름, 동시성 문제(Race Condition), 상태 전이, 엣지 케이스 시나리오 분석 보고서를 생성합니다.
---

# Scenario & Data Flow Impact Analysis Skill

특정 엔드포인트/함수에 특정 조건의 데이터가 유입되었을 때, **시간의 흐름(Time-sequence)에 따른 시스템 내부 데이터 처리 과정, race condition 발생 지점, 상태 변화 및 예외 시나리오**를 정밀 분석합니다.

## 분석 가이드라인

1. **시간순 흐름 추적 (Timeline-based)**:
    - 데이터가 들어온 순간부터(T0) DB 저장, 타 시스템 전파, 최종 응답까지(Tn) 시간 단위/순서별로 데이터 처리 과정을 분석합니다.

2. **동시성 및 레이스 조건(Race Condition) 심층 분석**:
    - 동일 조건 또는 인접한 시간(ms 단위)에 동일 요청이 중복 유입되거나, 연관 데이터가 동시에 수정될 때 발생하는 **Race Condition, Deadlock, Dirty Read, Phantom Read** 가능성을 타임라인별로 추적합니다.

3. **산출물 경로**:
    - 결과물은 프로젝트 루트 기준 `docs/scenario-analysis/` 하위에 `YYYYMMDD-[시나리오명]-impact.md` 형식으로 저장합니다. (`/tmp` 사용 금지)

---

## 작성 문서 구조 (템플릿)

문서 작성 시 다음 구조를 반드시 준수합니다:

### 1. 분석 대상 및 입력 시나리오 조건 (Scenario Context)
- **대상 엔드포인트 / 함수**: [예: POST /api/v1/coupons/issue 또는 issueCoupon()]
- **유입된 특정 조건/필드값**: [예: couponId: 501, userId: 1004, stock: 1 (마지막 1개 잔여)]
- **분석하려는 상황 (Situation)**: [예: 잔여 수량이 1개일 때 100명의 사용자가 동시에 요청을 보낸 상황]

### 2. 시간순 데이터 흐름 분석 (Time-Sequence Execution Flow)
데이터 유입 시점부터 처리 완료까지의 실행 순서를 시간 단계별로 서술합니다.

- **T0 (요청 수신)**: 입력 값 검증 및 메모리 로드 상태 (인자값, DTO)
- **T1 (비즈니스 로직 처리)**: 상태 판단 및 메모리 내 변수 변화
- **T2 (DB / 외부 연동)**: C/R/U/D 쿼리 실행 및 트랜잭션 범위
- **T3 (이벤트 발행 및 응답)**: 메시지 큐 전송, 최종 Return Value 반환

### 3. 동시성 / 레이스 조건 시나리오 분석 (Race Condition & Concurrency)

#### [시나리오 A: 동시 요청 발생 시 (Concurrent Requests)]
- **문제 발생 시점 (Critical Section)**: 어느 줄/어느 DB 작업에서 동시 접근이 문제가 되는가?
- **상태 오염 형태**: Race Condition 결과로 발생할 수 있는 데이터 불일치 (예: 초과 발급, 마이너스 수량 등)
- **해결 방안 제시**: 낙관적 락(Optimistic Lock), 비관적 락(Pessimistic Lock), Distributed Lock(Redis 등), DB Unique 제약 조건 중 권장 방식

### 4. 엣지 케이스 및 예외 처리 흐름 (Edge Cases)

| 번호 | 예외 / 엣지 상황 | 처리 결과 (Return/Status) | 변수 및 DB 상태 원복 여부 (Rollback) |
| :--- | :--- | :--- | :--- |
| **1** | [예: T2 지점 DB Timeout] | `500 Internal Error` | 트랜잭션 롤백, 수량 변수 원복 |
| **2** | [예: 중복 요청(Idempotency)]| `409 Conflict` 또는 기존 결과 반환 | 추가 DB 작업 없이 차단 |
| **3** | [예: 필드값 유효성 실패] | `400 Bad Request` | 비즈니스 로직 진입 전 차단 |

### 5. Mermaid 기반 시퀀스 다이어그램 가이드
- 시간 순서 및 Race Condition 발생 지점이 한눈에 보이도록 Mermaid 시퀀스 다이어그램을 그려주세요.
- **다이어그램에 꼭 들어가야 할 내용**:
    1) 각 주체(Client, API Server, DB, Redis 등) 간의 메시지 흐름
    2) Race Condition 발생 위험 구간 (`rect rgb(...)` 영역 또는 Note로 표시)
    3) 주요 함수 Return 값 및 DB State 변화