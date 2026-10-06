# 제품 요구사항 정의서 (PRD: Product Requirement Document)

## 1. 프로젝트 개요 (Overview)

* **프로젝트명:** 전국 학원·교습소 위치 및 경로 안내 웹 서비스 (가칭: EduMap)

* **목적:** 나이스(NEIS) 교육행정정보시스템 오픈 API의 학원교습소정보와 티맵(TMAP) API를 연동하여, 사용자가 주변 학원/교습소를 검색하고, 관심 학원을 즐겨찾기 등록하며, 지도시각화 및 길안내(경로 탐색) 정보를 한눈에 확인할 수 있는 웹 애플리케이션을 구축합니다.

* **주요 기술 스택:** Next.js (App Router), TypeScript, Tailwind CSS, TMAP JavaScript API v2 / TMAP Route API, 나이스 오픈 API (`acaInsTiInfo`), LocalStorage (즐겨찾기 저장용).

## 2. 사용자 페르소나 및 핵심 가치 (User Persona & Core Value)

* **주요 타깃:** 학원/교습소를 찾고자 하는 학부모, 학생 및 취업 준비생.

* **핵심 가치:**

  * 나이스(NEIS) 공공데이터 기반의 신뢰성 높은 학원·교습소 등록 데이터 조회.

  * 위치 기반 지도 탐색 및 시각적 이해도 향상.

  * 자주 찾는 학원/교습소 즐겨찾기(북마크)를 통한 빠른 조회 및 비교.

  * TMAP 연동을 통한 최적 도보 및 차량 길안내 경로 제공.

## 3. 핵심 기능 요구사항 (Key Features)

### 3.1 학원 및 교습소 정보 조회 (NEIS Open API)

* **오픈 API 연동:** 나이스 학원교습소 정보 API (`https://open.neis.go.kr/hub/acaInsTiInfo`) 연동.

* **지역별/관할별 검색:** 시도교육청코드 (`ATPT_OFCDC_SC_CODE`), 행정구역명 (`ADMST_ZONE_NM`) 기반 조회.

* **키워드 검색 및 페이징:** 학원명 검색 및 요청 페이지 위치 (`pIndex`), 페이지 당 요청 범주 (`pSize`) 옵션 지원.

* **상세 정보 제공:** 학원명 (`ACA_NM`), 도로명 주소 (`FA_RDNMA`), 분야명 (`REALM_SC_NM`), 레슨/교습과목, 연락처, 수강료 정보 등.

### 3.2 즐겨찾기(북마크) 기능

* **학원 즐겨찾기 추가/해제:** 학원 카드, 상세 모달, 지도 인포윈도우 내 별(★) 아이콘 클릭으로 즐겨찾기 토글.

* **즐겨찾기 데이터 영속성:** 로그인 없이 브라우저의 `localStorage`를 통해 저장 및 관리 (`FAVOIRTE_ACADEMIES_KEY`).

* **즐겨찾기 목록 필터링:** 좌측 사이드바에서 '전체 학원' / '즐겨찾기한 학원만 보기' 탭 전환 지원.

* **즐겨찾기 모아보기:** 즐겨찾기 등록된 학원들만 지도 위에 마커로 시각화하는 기능 제공.

### 3.3 TMAP 지도 시각화 (TMAP Map SDK)

* **현재 위치 기반 지도 렌더링:** 사용자의 HTML5 Geolocation 좌표를 기반으로 중심 위치 설정.

* **지오코딩 (Geocoding):** 나이스 API의 주소 정보(도로명 주소)를 TMAP Geocoding API를 통해 위경도 좌표로 변환하여 지도 상에 시각화.

* **마커 시각화:** 검색된 학원 위치에 마커 표시 및 클러스터링(다수 마커 존재 시) 처리 (즐겨찾기 학원은 커스텀 마커 아이콘 적용).

* **인포윈도우(Info Window):** 마커 클릭 시 간단한 정보(학원명, 교습분야, 즐겨찾기 버튼, 길안내 버튼) 팝업.

### 3.4 경로 안내 및 길안내 기능 (TMAP Route & Navigation)

* **도보/차량 경로 탐색:** 출발지(사용자 위치) $\rightarrow$ 도착지(선택한 학원 좌표) 최적 경로 지도 상 Polyline으로 시각화.

* **거리 및 소요시간 안내:** 총 거리(m/km) 및 예상 소요 시간(분) 표출.

* **외부 내비게이션 앱 연동:** 모바일 환경에서 TMAP 내비 앱 실행 Deep Link 지원.

## 4. 환경 변수 및 보안 (Environment Variables & Security)

* **API 키 보안:** Client 측에 인증키가 노출되는 것을 방지하기 위해 `.env.local` 환경변수 파일 활용.

* **환경변수 목록:**

  * `NEIS_API_KEY`: 나이스 교육정보 개방 포털 인증키

  * `NEXT_PUBLIC_TMAP_APPKEY`: TMAP JavaScript SDK 인증키

  * `TMAP_SERVER_API_KEY`: TMAP 경로 탐색 및 지오코딩용 서버 API 키

```
# .env.local
NEIS_API_KEY=your_neis_api_key_here
NEXT_PUBLIC_TMAP_APPKEY=your_tmap_appkey_here
TMAP_SERVER_API_KEY=your_tmap_server_api_key_here

```

## 5. 화면 구조 및 UI/UX (Information Architecture)

```
[메인 화면]
 ├── 상단 헤더 (앱 로고, 시도교육청/행정구 선택, 키워드 검색바)
 ├── 좌측/하단 사이드바
 │    ├── 탭 메뉴 ("전체 목록" | "⭐ 즐겨찾기 (N개)")
 │    └── 학원 리스트 카드 (즐겨찾기 토글 별 아이콘 포함)
 └── 메인 영역 (TMAP 지도 화면 + 위치 재설정 버튼)

[학원 상세 모달 / 인포윈도우]
 ├── 학원 기본정보 (이름, 주소, 연락처, 교습과목)
 ├── ⭐ 즐겨찾기 추가/해제 버튼
 └── 길안내 버튼 ("도보 경로", "차량 경로", "TMAP 앱으로 열기")

```

## 6. 데이터 흐름 및 시스템 구조 (Architecture)

```
[User Interface (Next.js Client)]
       │
       ├───► LocalStorage (즐겨찾기 학원 ID 및 정보 로컬 저장)
       │
       ├────► Next.js Server Route (/api/academies)
       │           │
       │           ├──► NEIS Open API (https://open.neis.go.kr/hub/acaInsTiInfo?KEY=${NEIS_API_KEY}&...)
       │           └──► TMAP Geocoding API (주소 ──► 좌표 변환)
       │
       └────► TMAP Web SDK v2 / Route API (TMAP APPKEY)

```

* **Next.js Server Route 역할:**

  * Client 측의 CORS 이슈 방지.

  * `NEIS_API_KEY` 보안 유지.

  * 나이스 API 응답의 주소를 TMAP Geocoding API와 연동해 위도/경도 정보(Latitude, Longitude)를 결합하여 Client에 전달.

## 7. 비기능적 요구사항 (Non-Functional Requirements)

* **반응형 웹:** 데스크톱, 태블릿, 모바일 디바이스 지원.

* **데이터 유지보수:** 즐겨찾기 데이터는 브라우저 용량 제한을 고려하여 필요한 학원 고유 식별자(`ACA_ASUT_NC`), 학원명, 좌표 정보 위주로 최적화하여 저장.

* **성능 및 캐싱:** API 호출 횟수 감소 및 로딩 속도 향상을 위해 검색 결과 SWR/React Query 또는 Next.js Data Caching 적용.

* **에러 핸들링:** Geolocation 거부 처리, TMAP API 로딩 실패 예외 처리, 주소 지오코딩 실패 대응.

## 8. 마일스톤 및 개발 단계 (Milestones)

| 단계 | 개발 내용 | 
| ----- | ----- | 
| **Phase 1** | Next.js 프로젝트 설정 및 `.env.local` 환경변수 구성, TMAP SDK Dynamic Loading 구현 | 
| **Phase 2** | NEIS API 연동 Route Handler 개발 (`/api/academies`) 및 주소-좌표 변환(Geocoding) 연동 | 
| **Phase 3** | 지도 마커 표출 및 학원 목록-지도 상호작용(클릭 연동) 구현 | 
| **Phase 4** | LocalStorage 기반 학원 즐겨찾기 기능 개발 (목록 필터링, 마커 스타일 분기) | 
| **Phase 5** | TMAP Route API를 활용한 길안내 경로 표출 및 소요시간 계산 | 
| **Phase 6** | 반응형 UI 정제, 예외 처리 및 최종 테스트 | 
