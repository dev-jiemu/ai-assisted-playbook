---
name: edge-case-analysis
description: 특정 코드, 신규 기능, 또는 아키텍처 변경 시 발생할 수 있는 잠재적 엣지 케이스, 동시성 이슈, 런타임 에러, 장애 시나리오를 사전에 예측하고 검증합니다.
---

# Pre-mortem & Edge Case Analysis Skill

새로 구현하되거나 변경된 코드가 **특정 릴리즈/운영 상황에서 어떻게 터질 수 있는지(Pre-mortem)** 사전에 분석하고, 이에 대응하는 예방책 및 검증 테스트를 제안합니다.

## 분석 체크리스트 (6대 장애 위험 요소)

분석 시 다음 6가지 관점에서 잠재적 위험을 송곳처럼 파헤칩니다:

1. **동시성 & 레이스 조건 (Concurrency & Race Conditions)**:
    - 동일 자원에 대한 동시 요청, 트랜잭션 격리 수준 문제, 데드락 발생 가능성
2. **분산 시스템 & 네트워크 장애 (Network & Partial Failures)**:
    - 외부 API 타임아웃, 재시도(Retry) 폭풍, 멱등성(Idempotency) 깨짐, 부분 성공/실패 상태
3. **경계값 & 데이터 비정상 (Edge Values & Malformed Data)**:
    - Null/Empty, 극단적으로 큰 문자열/숫자, DB Precision 오버플로우, 비정상 인코딩
4. **리포스 고갈 & 스케일 (Resource Exhaustion)**:
    - 메모리 누수, OOM, 커넥션 풀 고갈, N+1 쿼리로 인한 DB CPU 100%
5. **예외 처리 누락 (Unhandled Exceptions)**:
    - Silent Failure(에러가 먹히는 현상), Checked/Unchecked Exception 누락, Rollback 안 됨
6. **상태 불일치 (State Inconsistency)**:
    - 캐시(Redis)와 DB 간의 데이터 불일치, 분산 트랜잭션 미보장

---

## 작성 문서 구조 (템플릿)

결과물은 프로젝트 루트 기준 `docs/edgecases/` 하위에 `YYYYMMDD-[기능명]-edge-cases.md`로 생성합니다. (`/tmp` 금지)

### 1. 시뮬레이션 대상
- 분석 대상: [파일/메서드/기능]

### 2. 치명적 장애 예시 시나리오 (Top Potential Failures)

#### 🚨 시나리오 A: [장애 이름 (예: 동시 결제 요청 시 이중 차감)]
- **발생 조건**: [예: 동일 유저가 0.01초 간격으로 결제 버튼을 2번 누를 때]
- **원인 코드 위치**: `[파일 경로:라인 번호]`
- **상세 메커니즘**:
    1. A 요청이 DB에서 잔액 조회 (잔액 10,000원)
    2. B 요청이 DB에서 잔액 조회 (잔액 10,000원)
    3. A, B 모두 차감 로직 실행 -> 잔액이 -10,000원이 됨
- **심각도**: 🔴 CRITICAL / 🟡 WARNING

### 3. 장애 검증용 단위/통합 테스트 코드 제안
- 위 예외 상황이 실제로 발생하는지 검증할 수 있는 실패 테스트 코드(Fail Test) 작성

```[language]
// 예: 동시성을 검증하는 CountDownLatch 기반 테스트 코드