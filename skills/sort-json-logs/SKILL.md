---
name: sort-json-logs
description: 사용자가 "로그 시간순으로 정리해줘", "로그 타임라인 정렬해줘", "JSON 로그 정렬" 등을 요청하거나, 긁어온 JSON/JSONL 로그 데이터를 전달했을 때 실행하여 타임라인(시간 오름차순) 순으로 정렬합니다.
---

# JSON Logs Timeline Sorter

사용자가 전달한 무작위 순서의 JSON 로그 데이터를 시간 순서대로 재정렬하여 출력합니다.

## 처리 지침

1. **로그 형식 자동 인식:**
    - JSON 배열 형태(`[...]`) 및 줄바꿈 구분 JSONL 형태를 모두 파싱합니다.

2. **타임스탬프 필드 감지:**
    - 각 로그 객체에서 시간 정보를 나타내는 키를 우선적으로 탐색합니다.
    - 대상 필드 예시: `@timestamp`, `timestamp`, `time`, `created_at`, `date`, `datetime` 등

3. **타임라인 정렬:**
    - 추출한 시간 필드를 기준(ISO 8601, Epoch Timestamp 등)으로 **과거 -> 최신(오름차순)** 순으로 정렬합니다.

4. **시각화 및 결과 출력:**
    - 기본적으로 정렬된 **JSON 데이터**를 반환합니다.
    - 사용자가 **"시각화해줘"**, **"그래프/표로 보여줘"**라고 요청하거나 로그 흐름을 한눈에 파악할 필요가 있을 때 아래 형식 중 적절한 방법으로 시각화를 추가합니다


### 시각화 서식 예시
**1) ASCII 타임라인 흐름도**
```text
──( INFO )──> User login success
──( WARN )──> High CPU usage detected (85%)
──(ERROR)──> Database connection timeout
```

**2) Markdown 타임라인 테이블**

| 시각 | 레벨 | 이벤트 / 메시지 | 주요 상세 |
| :--- | :---: | :--- | :--- |
| 10:00:02 | INFO | User login | user_id: 42 |
| 10:01:15 | WARN | High CPU | usage: 85% |
| 10:02:40 | ERROR | DB Timeout | retry: 3 |
