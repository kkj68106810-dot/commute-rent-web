# 🚆 commute-rent-web — Commute Simulator Tokyo Web Frontend

> 수도권(도쿄권) 이주자를 위한 월세·통근 시뮬레이터 (B2C Web UI + B2B 관리 콘솔)

[한국어](README.ko.md) | [日本語](README.md) · 백엔드·DB: [commute-rent-api][backend-repo]

> [!NOTE]
> 현재 상태: **프론트엔드 프로토타입 v0.1.0** 입니다. API는 내장 **목업 API(`/api/*`, Next.js Route Handlers + 인메모리 DB)** 로 동작하며, Spring Boot 백엔드는 예정입니다.
> 월세·통근 시간은 **포트폴리오용 샘플 데이터**입니다 (목업: 41개 역, 근무지 6개 역). 월세는 실데이터 확보가 어려워 임시 생성 데이터를 쓰며, 노선·역 데이터는 백엔드에서 HeartRails Express API로 생성할 예정입니다 ([데이터 출처][backend-data]).

상태 표기: ✅ 구현됨 · 🟡 일부 구현 · 🔜 예정

---

## 1. 프로젝트 개요 (Project Overview)
- **타깃 (Target):** 도쿄권(도쿄·가나가와·사이타마·지바)으로 이주를 검토하는 IT 엔지니어 및 회사원
- **주요 기능 (Key Features):** 근무지와 희망 월세를 입력하면 최적의 거주 지역(역)을 지도에 추천하고, 월세 시세와 통근 시간을 시뮬레이션합니다.
- **구성:**
  - **B2C (SSR):** 검색·지도 화면, 역 상세 팝업, 로그인·회원가입 — 한국어·일본어
  - **B2B 관리 콘솔 (CSR):** 관리자 로그인(JWT), 대시보드, 역 마스터·월세 통계 관리(CRUD)

## 2. 기술 스택과 채택 이유 (Tech Stack & Why This Tech?)
| 기술 | 버전 | 채택 이유 |
|---|---|---|
| **Next.js (App Router)** | 16.3.5 | 서버 사이드 렌더링(SSR)으로 초기 로딩 속도와 SEO를 확보하고, 관리 화면은 CSR로 분리 |
| **React** | 19.2.8 | `useTransition` 으로 필터 변경 시 화면 전환 중 상태(로딩 바)를 처리 |
| **TypeScript** | 5.x | DB 스키마를 반영한 타입(`src/lib/types.ts`)으로 백엔드 API(Spring Boot)와의 통신 시 런타임 오류를 사전에 차단 |
| **Tailwind CSS** | 4.x | 별도 UI 라이브러리 없이 B2C·관리 UI 모두 Tailwind로 구현 (`@tailwindcss/postcss`) |
| **axios** | 1.18.0 | 브라우저용(인터셉터로 Bearer JWT 자동 부여, 타임아웃 8초)과 SSR용(타임아웃 5초) 인스턴스를 분리 |
| **react-leaflet + Leaflet** | 5.0.0 + 1.9.4 | OpenStreetMap 타일 기반 지도. API 키 불필요 |
| ESLint | 9 (`eslint-config-next` 16.3.5) | 정적 검사 |

## 3. 주요 기능 (Features)

### B2C — 검색·지도 (`/ko`, `/ja`)
| 기능 | 상태 |
|---|---|
| 근무지 역 / 최대 월세(50,000〜250,000엔) / 방 구조(1R·1K·1DK·1LDK·2LDK) / 최대 환승(0〜2회) / 최대 통근 시간(10〜90분) 필터 | ✅ |
| 서버 사이드 필터링(SSR)으로 결과를 HTML로 렌더링, 평균 월세 오름차순(동액은 통근 시간 순) | ✅ |
| 월세 구간 4색 마커(~7만 / 7〜9만 / 9〜11만 / 11만~엔), 근무지 마커, 자동 범위 맞춤, 범례 | ✅ |
| 리스트 ↔ 지도 연동 (행 클릭 시 지도 이동·팝업, 마커 클릭 시 해당 행 선택) | ✅ |
| 역 상세 팝업: 역명(일·한·가나·영), 노선 배지, 평균 월세·범위, 통근 시간, 환승 횟수 | ✅ |
| 즐겨찾기 토글 (낙관적 업데이트 + localStorage. 로그인 체크 없음, 목업 user_id=1 고정) | 🟡 |
| 로그인·회원가입 (`/{locale}/login`, 목업 JWT, 비밀번호 8자 이상) | ✅ (목업) |
| 마이페이지 (`/{locale}/mypage`) | 🔜 |

### B2B — 관리 콘솔 (`/admin`)
| 기능 | 상태 |
|---|---|
| 관리자 로그인, JWT(role=ADMIN, 만료 8시간) 클라이언트 가드 — 토큰 없음·만료 시 로그인 화면으로 이동 | ✅ (목업 JWT) |
| 대시보드: 역·노선·월세 통계·통근 데이터 건수, 1K 평균 월세, 도도부현별 1K 평균, 1K 상위/하위 5개 역 | ✅ |
| 데이터 그리드 (역 마스터 / 월세 통계): 서버 페이징·정렬·검색(디바운스)·필터 | ✅ |
| 등록·수정 모달, 삭제 확인 다이얼로그, `(station_id, room_layout)` 중복 시 409 오류 표시 | ✅ |

## 4. UI/UX & 구현 포인트 (UI/UX & Implementation)
프론트엔드 고유의 기술 과제를 다음과 같이 해결했습니다.

- **URL Query Parameters를 이용한 상태 관리:**
  복잡한 검색 조건(최대 월세, 통근 시간, 환승 횟수 등)을 URL 쿼리(`dest`, `maxRent`, `layout`, `maxTransfers`, `maxMinutes`)와 동기화했습니다. 사용자가 페이지를 새로고침하거나 검색 결과 URL을 공유해도 상태가 그대로 유지됩니다. 잘못된 값은 오류가 아니라 기본값으로 보정하며, 링크 복사·초기화 버튼을 제공합니다.
  예) `/ko?dest=3&maxRent=110000&layout=1K&maxTransfers=1&maxMinutes=45`
- **i18n 다국어 대응:**
  `/ko`·`/ja` 로케일 경로로 한국어·일본어를 전환하며, 전환 시 검색 조건을 유지합니다. 로케일별 title·description과 hreflang(alternates)을 출력하고, 그 외 로케일은 404입니다. 사전은 `src/lib/i18n.ts` 에 직접 정의했습니다 (i18n 라이브러리 미사용).
- **SSR과 지도 CSR의 분리:**
  검색은 Server Component에서 필터링해 HTML로 렌더링하고, Leaflet 지도는 `next/dynamic`(`ssr: false`)으로 클라이언트에서만 로드합니다.
- **백엔드 전환 구조:**
  SSR(`serverApi.ts`)과 브라우저(`http.ts`)의 접속처를 환경 변수로 전환합니다. SSR은 백엔드 미설정·장애 시 목업 데이터로 폴백합니다.
- **반응형:** `lg` 미만에서는 리스트를 지도 아래에 배치하고, 관리 사이드바는 `md` 이상에서 표시합니다.

## 5. 목업 API와 환경 변수 (Mock API & Environment)
`.env.example` 을 `.env.local` 로 복사합니다. **비워 두면 내장 목업 API(`/api/*`)를 사용합니다.**

| 변수 | 사용처 | 예시 |
|---|---|---|
| `API_BASE_URL` | SSR (Server Component) | `http://localhost:8080/api` |
| `NEXT_PUBLIC_API_BASE_URL` | 브라우저 axios 인스턴스 | `http://localhost:8080/api` |

- 목업 API는 `src/app/api/**` (Route Handlers)와 `src/lib/store.ts`(인메모리 DB)로 구현했습니다. 서버를 재시작하면 데이터가 초기화됩니다.
- 관리 목록 API는 Spring Data Pageable 형식: `?page=0&size=10&sort=field,asc&q=…` → `{content, totalElements, totalPages, number, size}`
- 목업 JWT는 서명 검증이 없는 데모 전용입니다 (본 구현: Spring Security + 서명된 JWT 예정).
- 엔드포인트 목록(API-001〜017)과 DB 설계는 [commute-rent-api README][backend-repo]를 참고하세요.

## 6. 디렉터리 구조 (Directory Structure)
```text
commute-rent-web/
├── src/
│   ├── app/
│   │   ├── [locale]/          # B2C (SSR): /ko, /ja, /{locale}/login
│   │   ├── admin/             # B2B 관리 콘솔 (CSR): login, (console)/ 대시보드·stations·rent-stats
│   │   ├── api/               # 목업 REST API (Route Handlers)
│   │   └── page.tsx           # / → /ko 리다이렉트
│   ├── components/            # SearchFilters, ResultsExplorer, MapView, AuthForm, Header, admin/DataGrid …
│   └── lib/
│       ├── types.ts           # DB 스키마를 반영한 타입
│       ├── i18n.ts            # ko / ja 사전
│       ├── query.ts           # URL 쿼리 파싱·기본값, 월세 구간
│       ├── http.ts            # 브라우저 axios (JWT 인터셉터)
│       ├── serverApi.ts       # SSR axios (목업 폴백)
│       ├── auth.ts            # 목업 JWT
│       └── store.ts, mockData.ts  # 목업 인메모리 DB·샘플 데이터
├── screenshots/               # 화면 캡처
└── .env.example
```

## 7. 실행 절차 (Getting Started)
로컬 환경 실행 방법입니다. 프론트엔드는 표준 포트 `3000` 을 사용합니다. 백엔드 없이 목업 API로 단독 실행할 수 있습니다.

```bash
# 0. 저장소 클론
git clone https://github.com/OWNER/commute-rent-web.git
cd commute-rent-web

# 1. 의존 패키지 설치
npm install

# 2. 로컬 개발 서버 실행 (Port: 3000)
npm run dev

# (선택) 프로덕션 빌드·실행 / 린트
npm run build && npm start
npm run lint
```

| URL | 화면 |
|---|---|
| `http://localhost:3000/ko`, `/ja` | B2C 검색·지도 (`/` 는 `/ko` 로 리다이렉트) |
| `/ko/login`, `/ko/login?mode=signup` | 로그인·회원가입 (목업) |
| `/admin/login` | 관리 콘솔 — 데모 계정 `admin@commute.dev` / `admin1234` |

## 8. 스크린샷 (Screenshots)
| B2C 검색·지도 (한국어) | B2C 검색·지도 (일본어) |
|---|---|
| ![B2C 검색·지도 (ko)](screenshots/01-b2c-search-map-ko.png) | ![B2C 검색·지도 (ja)](screenshots/02-b2c-search-map-ja.png) |
| **역 상세 팝업** | **회원가입** |
| ![역 상세 팝업](screenshots/03-b2c-marker-popup-ko.png) | ![회원가입](screenshots/04-b2c-signup.png) |
| **관리자 로그인** | **관리 대시보드** |
| ![관리자 로그인](screenshots/05-admin-login.png) | ![관리 대시보드](screenshots/06-admin-dashboard.png) |
| **월세 통계 그리드** | **역 마스터 그리드** |
| ![월세 통계 그리드](screenshots/07-admin-datagrid-rent-stats.png) | ![역 마스터 그리드](screenshots/08-admin-datagrid-stations.png) |
| **수정 모달** | |
| ![수정 모달](screenshots/09-admin-edit-modal.png) | |

## 9. 관련 문서 (Documents)
설계 문서(요구사항 정의서·화면설계서·API 명세서·테이블 정의서·테스트 케이스, 한·일)와 통근 시간 API 조사는 백엔드 저장소 [commute-rent-api][backend-repo]의 `docs/` 에 있습니다.

## 10. 로드맵 (Roadmap)
- [x] B2C 검색·지도 (SSR, URL 쿼리 상태, ko/ja)
- [x] B2B 관리 콘솔 (대시보드, 데이터 그리드 CRUD)
- [x] 목업 API·환경 변수로 백엔드 전환 구조
- [ ] 🔜 Spring Boot API 연동 (실 JWT 인증)
- [ ] 🔜 즐겨찾기 로그인 필수화 (비로그인 시 로그인 화면으로 이동)
- [ ] 🔜 마이페이지 (즐겨찾기 목록·삭제, 역 중심 지도 이동)
- [ ] 🔜 관리 화면 선택지를 목업 데이터 직접 import 대신 API(`/api/prefectures`)로 대체
- [ ] 🔜 HeartRails Express 데이터 사용 시 화면에 「HeartRails Express」 크레딧 표기 (무료 이용 조건)
- [ ] 🔜 통근 시간이 추정치임을 역 상세 팝업에 표기
- [ ] 🔜 DB와 타입 차이 해소 (`station_name_ko`, 노선 `company`/`color` 는 현재 DB에 없음)

---
※ 월세·통근 시간은 포트폴리오용 샘플 데이터입니다. 지도 © OpenStreetMap contributors · © Kim GwangJin

<!-- OWNER 를 GitHub 사용자명으로 바꾸세요 (기본 브랜치 main 가정). -->
[backend-repo]: https://github.com/OWNER/commute-rent-api/blob/main/README.ko.md
[backend-data]: https://github.com/OWNER/commute-rent-api/blob/main/README.ko.md#6-데이터-출처-data-sources
