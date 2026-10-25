---
name: version-upgrade-history
description: 프로그래밍 언어, 프레임워크, 라이브러리, 도구 등의 지정된 대상 버전(As-Is에서 To-Be까지) 간의 릴리즈 노트, 변경 이력(Changelog), Breaking Changes 및 마이그레이션 이력을 추출합니다. 특정 버전 간의 업그레이드 변경 사항, Breaking Changes, 마이그레이션 로그 또는 릴리즈 히스토리 확인을 요청받았을 때 실행됩니다.
---

# Version Upgrade History Extractor

As-Is 버전부터 To-Be 버전까지의 변경 이력(Changelog), Breaking Changes, Migration Pointer 등을 검색 및 정리하는 스킬입니다.

## 워크플로우 (Workflow)

1. **입력 파라미터 확인 및 파싱**
    - 대상 소프트웨어 (예: Spring Boot, Node.js, React, Python, PostgreSQL 등)
    - `As-Is` 버전 (시작 버전) 및 `To-Be` 버전 (목표 버전)

2. **공식 변경 이력 및 릴리즈 노트 탐색**
    - GitHub Releases API / `CHANGELOG.md` / 공식 마이그레이션 가이드 탐색 및 크롤링
    - `curl` 또는 Web Search/Fetch 명령을 활용해 지정된 범위(As-Is < v <= To-Be) 내의 모든 릴리즈 내역 수집

3. **변경 사항 분류 및 요약**
    - 💥 **Breaking Changes / Deprecations** (하위 호환성 단절 및 폐지 예정)
    - 🚀 **Major New Features** (주요 신규 기능)
    - 🛠 **Fixes / Performance** (주요 버그 수정 및 성능 개선)
    - 🔒 **Security Fixes** (보안 패치)

4. **최종 리포트 출력**

---

## Output Template (출력 형식)

# 🚀 Version Upgrade History: [대상] ([As-Is] ➔ [To-Be])

## 📊 개요
- **대상:** `[Software/Library Name]`
- **업그레이드 범위:** `v[As-Is]` ➔ `v[To-Be]`
- **위험도 평가:** `[High / Medium / Low]` (Breaking Change 수에 따름)

## 💥 주요 Breaking Changes (필수 확인)
- **[vX.Y.Z]** [변경 내용 요약 및 대안/조치 방법]

## 📜 버전별 변경 이력 (Changelog History)

### `v[To-Be]`
- 💥 **Breaking:** ...
- 🚀 **Features:** ...
- 🛠 **Fixes/Perf:** ...

*(중간 릴리즈 버전 순차 표기)*

### `v[As-Is + 0.1]`
- 🚀 **Features:** ...

## 📋 마이그레이션 체크리스트
- [ ] 주요 의존성 패키지 버전 호환성 확인
- [ ] Deprecated 된 API/설정값 대체
- [ ] 테스트 코드 실행 및 검증