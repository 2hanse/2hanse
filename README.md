# 이한세 | Hanse Lee

**반복되는 배포·검증 문제를 자동화와 게이트로 해결하는 개발자입니다.**

쓰리더블유에서 업무 협업 서비스 WITIM(웹·Flutter 앱·데스크톱 모노레포)과 신규 웹 서비스를 개발하고 있습니다(2026.01 ~ 재직 중).  
구현은 AI 코딩 에이전트에게 맡기고, 문제 정의·설계 결정·검증은 직접 합니다.

> 회사 저장소는 비공개라, 코드 대신 한 일과 판단을 적었습니다.

<br/>

## 🏢 쓰리더블유 실무 (2026.01 ~)

### CI/CD와 배포 자동화

- **릴리즈 노트 자동 발행** — 루트 VERSION 하나에 섞여 있던 앱 버전을 앱별로 나누고, 저장소에 쌓이던 릴리즈 노트를 병합 때 GitHub Releases로 자동 발행하게 바꿨습니다. 발행된 태그는 건너뛰고 모든 트랙이 성공해야 원본을 지워, 부분 실패는 다음 실행이 재시도합니다. 3개월 넘게 다른 개발자의 병합에서도 동작했습니다.
- **모바일 CI/CD** — iOS·Android 버전을 VERSION 파일 하나에서 Xcode Run Script·Gradle로 빌드 시점에 주입하고, 병합마다 돌며 등재된 빌드 번호와 부딪히던 자동 배포를 수동 실행 전용으로 바꿨습니다.
- **자체 호스팅 배포 장애** — 배포마다 CSS가 깨지던 문제를 `root로 실행된 서비스 → 캐시 파일 root 소유 → 다음 배포 rm -rf 실패 → && 단락 평가로 복사 누락 → 새 CSS 404` 원인 체인으로 규명하고, 서비스 실행 사용자 지정과 복사 실패 시 단계 실패로 고쳤습니다.

### AI 개발 도구와 품질 게이트

- **QA 봇** — QA 이슈의 간단한 UI 수정을 헤드리스 코딩 에이전트에 맡기되, 결과물은 브랜치 커밋까지로 제한하고 반영은 사람이 정하게 설계해 시험 운영했습니다. 게이트 결과는 에이전트의 자기 보고 대신 스크립트 실측값으로 판정합니다.
- **개발 하네스(개인)** — 완료 조건 → 수정 전 e2e 재현 → 커밋 전 e2e 통과·다른 모델 교차 검증을 Claude Code 훅으로 강제합니다.

### 오류 감지와 장애 진단

- **운영에서만 깨지던 배포 결함** — 신규 웹 서비스를 운영과 같은 구성으로 실기동해 CI가 놓친 배포 결함을 찾아 고치고, 반복되던 환경변수 누락은 코드가 읽는 키와 배포 설정을 대조하는 테스트로 막았습니다.
- **푸시 탭 이동 회귀** — 알림함과 달리 푸시 경로에만 링크 번역이 없다는 차이에서 원인을 찾아 서버 알림 링크를 앱 라우트로 번역하고, 수리를 되돌리면 실패하는 통합 테스트를 두었습니다.

<br/>

## 🐾 사이드 프로젝트

- **[댕로드](https://github.com/After-Daeng-Road/After-Daeng-Road)** · 2026 관광데이터 활용 공모전 출품 · 2026.05 ~ 진행 중 · [데모](https://after-daeng-road-web-pi.vercel.app) — 퇴근 후 반려견과 다녀올 한적한 근교를 추천하는 서비스. 한국관광공사 TourAPI·반려동물 동반여행 API 연동, 추천 API(시간 예산·운영시간 반영·페이지네이션), 관광 데이터 실측 기반 한적도 적재, 회원 탈퇴·개인정보 파기 로직을 맡았습니다.

<br/>

## 📦 입사 전 프로젝트 (AI 코딩 도구 없이 작성)

- **[싹싹커밋](https://github.com/ssak-three/ssakssak-commit)** · 3인 팀 · 2025.08 ~ 11 — 커밋 분석 리포트 서비스에서 커밋 수집과 GitHub API 한도 대응, 리포트·분석 진행 화면 담당
- **[QuickWise](https://github.com/2hanse/Quick-wise)** · 1인 · 2025.09 ~ 12 — Google Calendar 일정 맞춤 준비 카드 Android 앱. Kotlin 알림 모듈을 RN 브릿지에서 Expo Modules API 로컬 모듈로 이전

<br/>

## 💻 Tech Stack

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=TypeScript&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=Next.js&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=React&logoColor=black)
![React Native](https://img.shields.io/badge/React_Native-61DAFB?style=flat-square&logo=React&logoColor=black)
![Flutter](https://img.shields.io/badge/Flutter-02569B?style=flat-square&logo=Flutter&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-7F52FF?style=flat-square&logo=Kotlin&logoColor=white)
![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat-square&logo=NestJS&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=PostgreSQL&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=Prisma&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=GitHubActions&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=Docker&logoColor=white)
![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=Vitest&logoColor=white)
![Claude Code](https://img.shields.io/badge/Claude_Code-D97757?style=flat-square&logo=Anthropic&logoColor=white)

<br/>

## 📫 Contact

- **Email:** leehanse.dev@gmail.com
