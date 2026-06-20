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

### 자동화 방식

수동 Export 방식은 "Postman에서 수정했는데 레포에 반영을 깜빡"하면 CI가 옛날 컬렉션으로 테스트하는 문제가 있습니다. 이를 없애는 방향이 **Postman을 단일 원본(single source of truth)으로 두고, CI가 실행할 때마다 최신 컬렉션을 직접 받아오는 것**입니다.

| 방식 | 동기화 시점 | 사람 개입 | 단점 |
|------|-------------|-----------|------|
| 수동 Export | 사람이 export/commit 할 때 | 매번 필요 | 반영 누락 위험 |
| Postman API 연동 | CI 실행 시점에 자동 | 불필요 | API Key 관리 필요 |

### Postman API로 CI가 알아서 최신 컬렉션을 가져오기

CI가 실행될 때 Postman API로 최신 컬렉션을 내려받아 그대로 Newman에 넘기는 방식입니다. 레포에 컬렉션 JSON을 커밋해 두지 않아도 됩니다.

**1. 준비물**

- **Postman API Key**: Postman → 우상단 프로필 → Settings → API keys 에서 발급
- **Collection UID**: 컬렉션 → 우측 패널 또는 공유 링크에서 확인 (`uid` 형태, 예: `12345678-abcd-...`)

**2. GitHub Secrets 등록**

레포 Settings → Secrets and variables → Actions 에서 추가합니다.

| 이름 | 값 |
|------|-----|
| `POSTMAN_API_KEY` | 발급받은 API Key |
| `POSTMAN_COLLECTION_UID` | 컬렉션 UID |

**3. 워크플로우에서 최신 컬렉션 실행**

Newman은 Postman API URL을 직접 실행할 수 있습니다.

```yaml
      - name: Run API tests (Postman 최신 컬렉션)
        env:
          POSTMAN_API_KEY: ${{ secrets.POSTMAN_API_KEY }}
          POSTMAN_COLLECTION_UID: ${{ secrets.POSTMAN_COLLECTION_UID }}
        run: |
          newman run "https://api.getpostman.com/collections/${POSTMAN_COLLECTION_UID}?apikey=${POSTMAN_API_KEY}"
```

또는 먼저 파일로 내려받은 뒤 실행할 수도 있습니다. (로그/아티팩트로 남기고 싶을 때)

```yaml
      - name: Fetch latest collection
        env:
          POSTMAN_API_KEY: ${{ secrets.POSTMAN_API_KEY }}
          POSTMAN_COLLECTION_UID: ${{ secrets.POSTMAN_COLLECTION_UID }}
        run: |
          curl -sS -H "X-Api-Key: ${POSTMAN_API_KEY}" \
            "https://api.getpostman.com/collections/${POSTMAN_COLLECTION_UID}" \
            | jq '.collection' > postman/collection.json

      - name: Run API tests
        run: newman run postman/collection.json
```

> [!IMPORTANT]
> API Key는 절대 레포에 평문으로 커밋하지 말고 반드시 GitHub Secrets로 관리하세요. URL 방식은 키가 명령줄에 노출되므로, 민감하게 다룬다면 헤더(`X-Api-Key`) 방식을 권장합니다.

> [!NOTE]
> 이 방식으로 가면 레포의 컬렉션 JSON은 더 이상 원본이 아닙니다. 버전 기록을 위해 레포에도 남기고 싶다면, 환경(environment)은 Postman API로 별도 조회해 `-e` 옵션으로 함께 넘길 수 있습니다.

---

## Documentation

- [스크립트 작성 (Tests)](https://learning.postman.com/docs/tests-and-scripts)
- [Collection Runner](https://learning.postman.com/docs/running-collections)
- [Monitor (스케줄 자동화)](https://learning.postman.com/docs/monitoring)
- [Newman (CLI 실행)](https://learning.postman.com/docs/newman)
