# 이한세 | Hanse Lee

**서버 API를 설계하고, 그 API를 쓰는 모바일 앱까지 만드는 개발자입니다.**

쓰리더블유에서 WITIM(위팀)의 API 서버(Next.js·Prisma·PostgreSQL·Redis)와 Flutter 앱을 개발하고 있습니다(2026.01 ~ 재직 중).  
위팀은 근태·채팅·프로젝트·휴가 관리를 한곳에서 하는 팀용 업무 협업 서비스입니다.

> 회사 저장소는 비공개라, 코드 대신 해결한 문제와 설계 판단을 적었습니다.

<br/>

## 🛠 서버에서 해결한 문제

### 출근 API — 멱등성 키와 서버 판정으로 중복·오탐 막기

모바일과 웹이 동시에 출근을 찍으면 중복 행이 생기고, 핫스팟으로 회사에 온 사용자는 IP 화이트리스트에 막혔습니다.

- 중복 방지를 두 겹으로 걸었습니다. 라우트는 `Idempotency-Key`로 기존 행이 있으면 409를 돌려주고, 조회를 동시에 통과한 요청은 DB 유니크 제약이 막습니다. 키는 `sha256(사용자|워크스페이스|영업일)`로 만들어 영업일 경계(오전 6시)를 서버 크론과 맞췄습니다.
- 위치 검증은 GPS·IP를 AND가 아니라 OR로 두고, 422 응답에 `gps_pass`·`ip_pass`·거리·권장 조치를 실어 앱이 분기하게 했습니다.
- 자동 출퇴근은 앱이 좌표를 서버로 보내던 구조를 버리고 **거리 구간과 신호만 받아 서버 한 곳에서 판정**하게 바꿨습니다. 위치 개인정보가 서버에 남지 않고 판단 로직이 앱마다 흩어지지 않습니다. 운영 DB의 이벤트 기록으로 "클라이언트가 친 퇴근"이 원인임을 확정하고, 진단 신호로 "OS가 등록을 버림"과 "등록됐지만 발화 안 함"을 갈라 계측했습니다.

### 모바일 단일 활성 세션 — 이중 푸시를 서버 한 관문에서 막기

출근 완료 푸시가 사용자의 모든 기기와 좀비 토큰에 팬아웃됐고, 단일 세션을 배포한 뒤에도 재발했습니다.

- 데스크톱의 Redis 활성 토큰 포인터 패턴을 모바일에 옮기되, Centrifugo 이벤트 필드를 분리해 데스크톱·웹에는 영향이 없게 했습니다.
- 재발은 앱 배포 없이 서버만으로 끝냈습니다. 모든 푸시가 지나는 `sendPushToUsers` 한 관문에 기기 가드를 넣어 전 유형을 한 번에 덮었습니다.
- 기기 등록을 active·inactive·unknown 3상태로 나눴습니다. inactive는 401이 아니라 200 soft-reject로 거절했습니다. 401은 앱이 사유 없이 강제 로그아웃하는 경로라서입니다. Redis 장애(unknown)는 fail-open으로 뒀습니다.

### 첨부 저장 경합 — 유니크 충돌을 작업 성격으로 가르기

신규 워크스페이스의 첫 다중 업로드가 항상 500이었고, 손상 영상 하나가 썸네일 큐를 무한 재시도로 점유했습니다.

- 유니크 위반(P2002)을 한 가지로 처리하지 않았습니다. 읽기 목적 get-or-create는 "상대가 이미 만들었다"는 뜻이라 재조회하고, 값이 누적되는 upsert는 재조회하면 증가분이 사라지므로 트랜잭션 재시도로 갈랐습니다. 재시도는 opt-in으로 둬 진짜 중복인 경로가 백오프를 낭비하지 않게 했습니다.
- 워커는 추출 실패도 잡 성공으로 처리해 큐 점유를 막고, 웹 요청 처리 중에는 ffmpeg를 돌리지 않는다는 전제를 테스트로 고정했습니다.

### 알림 배지 — 4곳에서 다르던 셈 규칙을 서버 한 곳으로

앱 아이콘 배지는 248인데 알림 탭은 18개였습니다. 아이콘·푸시 배지·탭 배지·목록마다 개수 제한·기간·워크스페이스 범위가 달랐고, DM만 저장 카운터라 한 번 틀어지면 스스로 고쳐지지 않았습니다.

- 셈 규칙을 서버 한 곳으로 모으고, 목록과 배지가 같은 제외 상수를 공유하게 했습니다.
- 저장 카운터 대신 파생 계산으로 바꾸고, 백필은 배지가 늘어나는 방향으로는 틀어지지 않게 설계했습니다. 30일 정리는 삭제가 아니라 읽음 처리로, 캐시는 유저별 무효화 대신 자연 수렴으로 트레이드오프를 택했습니다.

### 어드민 푸시 발송 — 전체 사용자 대상 발송의 원자성

전체 사용자 대상이라 중복 발송·중도 실패·무인증 호출이 곧 사고였습니다.

- 조회 가드와 쓰기 사이에 원자성이 없으면 두 요청이 같은 초안을 통과해 전원에게 두 번 나가므로, `status IN (DRAFT, SCHEDULED)` 조건부 `updateMany`로 청구하고 count가 0이면 거부했습니다.
- 청구 뒤 예외는 SENDING에 고착되지 않게 FAILED로 낙착하고, 프로덕션에서 시크릿이 없으면 엔드포인트를 열지 않게 했습니다. FCM 크레덴셜과 발송 코드는 한 곳에만 뒀습니다.

<br/>

## 📱 모바일 앱에서 해결한 문제

### GPS 자동 출퇴근 — 폴링에서 OS 지오펜스로

- 위치 스트림 폴링은 Doze·강제 종료·재부팅 뒤 동작을 보장하지 못해, Android GeofencingClient·iOS CLCircularRegion을 쓰는 OS 지오펜스로 교체하고 앱이 꺼진 상태에서 콜백을 받는 백그라운드 isolate 진입점을 만들었습니다.
- 재구축이 만든 회귀(설정 반경 무시, 가짜 EXIT로 인한 잘못된 퇴근 푸시)를 순수 함수 상태 머신으로 다시 고쳤고, 기기에서만 드러나는 동작은 단위 테스트 통과를 완료로 보지 않게 됐습니다.

### 오프라인 대응 — 임시 큐에서 정책 결정표로

- 연결이 끊기면 요청을 로컬 큐에 저장했다가 지수 백오프로 재전송하게 했습니다. 일주일 뒤 4xx·5xx가 재시도 횟수를 올리지 않아 큐에 영구히 남는 결함과, **로그아웃 뒤 이전 계정의 큐가 새 토큰으로 실행될 수 있는 보안 문제**를 스스로 찾아 고쳤습니다.
- 두 달 뒤 결정 14개로 정책을 문서화했습니다. 메시지·첨부는 자동 재전송하지 않고 사용자가 재시도·취소하며, 읽음 처리는 메시지 큐와 분리한 가벼운 큐로 처리합니다.

<br/>

## 🐾 사이드 프로젝트

- **[댕로드](https://github.com/After-Daeng-Road/After-Daeng-Road)** · 2026 관광데이터 활용 공모전 출품 · 2026.05 ~ 진행 중 · [데모](https://after-daeng-road-web-pi.vercel.app) — 퇴근 후 반려견과 다녀올 한적한 근교를 추천하는 서비스. 한국관광공사 TourAPI·반려동물 동반여행 API 연동, 추천 API(시간 예산·운영시간 반영·페이지네이션), 관광 데이터 실측 기반 한적도 적재, 회원 탈퇴·개인정보 파기 로직을 맡았습니다.

<br/>

## 📦 입사 전 프로젝트

- **[싹싹커밋](https://github.com/ssak-three/ssakssak-commit)** · 3인 팀 · 2025.08 ~ 11 — 커밋 분석 리포트 서비스에서 커밋 수집과 GitHub API 한도 대응, 리포트·분석 진행 화면 담당
- **[QuickWise](https://github.com/2hanse/Quick-wise)** · 1인 · 2025.09 ~ 12 — Google Calendar 일정 맞춤 준비 카드 Android 앱. Express 서버와 Kotlin 알림 모듈(RN 브릿지 → Expo Modules API 이전)

<br/>

## 💻 Tech Stack

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=TypeScript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=Node.js&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=Next.js&logoColor=white)
![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat-square&logo=NestJS&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=Prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=PostgreSQL&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=Redis&logoColor=white)
![BullMQ](https://img.shields.io/badge/BullMQ-D62828?style=flat-square)
![Centrifugo](https://img.shields.io/badge/Centrifugo-1E90FF?style=flat-square)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=Firebase&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=Docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=GitHubActions&logoColor=white)
![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=Vitest&logoColor=white)
![Flutter](https://img.shields.io/badge/Flutter-02569B?style=flat-square&logo=Flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-0175C2?style=flat-square&logo=Dart&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=React&logoColor=black)

<br/>

## 📫 Contact

- **Email:** leehanse.dev@gmail.com
