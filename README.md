<div align="center">

# API Test Automation with Postman + Newman

Postman 컬렉션으로 REST API를 검증하고, GitHub Actions에서 Newman으로 자동 실행하는 API 테스트 자동화 레포지토리

[![API Test (Newman)](https://github.com/JangHwanPark/test-automation-api-postman/actions/workflows/newman.yml/badge.svg)](https://github.com/JangHwanPark/test-automation-api-postman/actions/workflows/newman.yml)
[![Postman](https://img.shields.io/badge/Postman-Collection-FF6C37?logo=postman&logoColor=white)](https://www.postman.com/)
[![Newman](https://img.shields.io/badge/Newman-CLI-FF6C37?logo=postman&logoColor=white)](https://github.com/postmanlabs/newman)
[![Node.js](https://img.shields.io/badge/Node.js-20-339933?logo=node.js&logoColor=white)](https://nodejs.org/)

</div>

---

## 목차

- [소개](#소개)
- [API Data](#api-data)
- [디렉토리 구조](#디렉토리-구조)
- [테스트 커버리지](#테스트-커버리지)
- [로컬에서 실행하기](#로컬에서-실행하기)
- [CI 자동화 (GitHub Actions)](#ci-자동화-github-actions)
- [Postman과 깃허브 연결하기](#postman과-깃허브-연결하기)
- [Documentation](#documentation)

---

## 소개

이 레포지토리는 [restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html) API를 대상으로 한 테스트 자동화 예제입니다.

- **Postman**: 요청과 검증 스크립트(Tests)를 컬렉션으로 관리
- **Newman**: 컬렉션을 CLI에서 실행하는 러너 (Node.js 기반)
- **GitHub Actions**: 컬렉션이 변경되거나 PR이 열릴 때 테스트를 자동 실행

정상 응답뿐 아니라 잘못된 입력, 없는 리소스 등 실패 케이스까지 함께 검증합니다.

---

## API Data

- [restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html)

---

## 디렉토리 구조

```text
test-automation-api-postman/
├─ .github/
│  └─ workflows/
│     └─ newman.yml                       # Newman 실행 CI 워크플로우
├─ postman/
│  └─ API Test.postman_collection.json    # Postman 컬렉션
└─ README.md
```

---

## 테스트 커버리지

| 그룹 | 메서드 | 케이스 | 기대 결과 |
|------|--------|--------|-----------|
| Get Booking | `GET` | 전체 조회 | `200` |
| Get Booking | `GET` | 이름 필터, 결과 없음 | `200` |
| Get Booking | `GET` | 정상 조회 | `200` |
| Get Booking | `GET` | 없는 ID | `404` |
| Get Booking | `GET` | 형식 오류 | `4xx` |
| Create Booking | `POST` | 정상 생성 | `200` |
| Create Booking | `POST` | 필드 누락 | 오류 |
| Create Booking | `POST` | 잘못된 타입 | 오류 |
| Create Booking | `POST` | 잘못된 날짜 형식 | 오류 |
| Update Booking | `POST` | 전체 수정 (UpdateBooking) | - |
| Update Booking | `PATCH` | 부분 수정 (PartialUpdateBooking) | - |
| Delete Booking | `DELETE` | 삭제 (DeleteBooking) | - |
| Auth | `POST` | 토큰 발급 | - |
| HealthCheck | `GET` | Ping | - |

---

## 로컬에서 실행하기

### 사전 준비

- [Node.js](https://nodejs.org/) 20 이상

### Newman 설치

```bash
npm install -g newman
```

### 컬렉션 실행

```bash
newman run "postman/API Test.postman_collection.json"
```

> [!TIP]
> HTML 리포트가 필요하면 `newman-reporter-htmlextra`를 설치한 뒤 `-r htmlextra` 옵션을 추가하세요.
> ```bash
> npm install -g newman-reporter-htmlextra
> newman run "postman/API Test.postman_collection.json" -r htmlextra
> ```

---

## CI 자동화 (GitHub Actions)

`.github/workflows/newman.yml` 워크플로우가 다음 시점에 테스트를 실행합니다.

- **push**: `main` 브랜치에서 `postman/**` 또는 워크플로우 파일이 변경될 때
- **pull_request**: `main`, `release/**`, `feat/*` 브랜치를 대상으로 한 PR
- **수동 실행**: Actions 탭의 `workflow_dispatch` 버튼

실행 단계는 레포 체크아웃 → Node.js 20 설치 → Newman 설치 → 컬렉션 실행 순으로 진행됩니다.

> [!NOTE]
> 현재 워크플로우는 `postman/API_test.postman_collection.json` 경로를 참조하지만 실제 파일명은 `postman/API Test.postman_collection.json`입니다. CI가 정상 동작하려면 두 경로를 일치시켜야 합니다.

---

## Postman과 깃허브 연결하기

### 수동으로 Export

```text
Postman에서 수정 → Export → 레포 파일 덮어쓰기 → git commit & push
```

- 컬렉션이 바뀔때 마다 위 파이프라인을 수행해야함
- 자주 변경되지 않으면 부담이 적지만 반대면 유지보수가 힘듬

---

## Documentation

- [스크립트 작성 (Tests)](https://learning.postman.com/docs/tests-and-scripts)
- [Collection Runner](https://learning.postman.com/docs/running-collections)
- [Monitor (스케줄 자동화)](https://learning.postman.com/docs/monitoring)
- [Newman (CLI 실행)](https://learning.postman.com/docs/newman)
