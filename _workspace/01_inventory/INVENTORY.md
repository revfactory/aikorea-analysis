# 수집 인벤토리 — 국가인공지능전략위원회 (aikorea.go.kr)

- **대상**: https://www.aikorea.go.kr (대통령직속 국가인공지능전략위원회, NAIS)
- **수집 일시**: 2026-06-05 05:00 ~ 05:14 (KST)
- **수집 도구**: WebFetch(초기 구조 파악) + Chrome 브라우저 MCP(DOM/헤딩/alt/폼/네트워크/페이지텍스트) + headless Chrome(스크린샷 영속화) + curl/openssl(응답 헤더·성능·TLS·404)
- **수집자**: web-site-scout (정찰 단계)

## 수집 요약

| 항목 | 값 |
|------|-----|
| 한글판 페이지 (구조/콘텐츠/게시판/검색/사이트맵) | 18개 전수 수집 |
| 게시글 상세 | 대표 1건 샘플(정책자료 num=523) |
| 영문판 페이지 | 메인+정책 본문 수집 / 콘텐츠 4종 메뉴구조만 |
| 스크린샷 | 6장 (데스크톱 4 + 모바일 2) |
| 게시물 총량(목록 집계) | 약 565건 (개별 본문 미수집) |
| 발견 오류/이상 응답 | 404 외부 리다이렉트 1, soft-404 1 (상세 기록) |
| 외부 추적 스크립트 | 미발견 |
| 깨진 링크(방문 페이지 기준) | 0 (방문한 모든 페이지 200) |

## 페이지 수 / 샘플링 여부

- **구조·콘텐츠·검색 페이지**: 한글판 **전수 수집**(본문 텍스트 + HTML 구조: 헤딩/alt/폼/링크).
- **게시판 목록 11종**: **전수 방문**(목록 1페이지 + 총 건수 기록).
- **게시글 상세(약 565건)**: **샘플링** — 대표 1건만 구조 수집.
  - 샘플링 기준: 깊이보다 넓이 우선 + 정부 사이트(.go.kr) 부하 자제 원칙. 게시판별 대표 메뉴/목록은 전수 확보했고, 개별 글은 후속 분석가가 특정 글을 지정 요청 시 추가 수집.
- **영문판**: 메인+정책(Korea AI Action Plan) 본문 수집, 나머지 4개 콘텐츠 페이지는 메뉴 구조만(본문 미수집).

## 파일 색인

### 사이트맵
- `sitemap.md` — 한글판 18 + 영문판 7 + 외부/시스템 URL, 메뉴 경로·응답코드·게시물수·샘플링 정책

### 페이지별 (pages/)
- `pages/01_main.md` — 메인 페이지(히어로/GNB/AJAX 게시판/스킵네비)
- `pages/02_committee-intro.md` — 위원회 소개(4대 원칙·비전)
- `pages/03_members.md` — 위원 소개(위원장·정부위원·민간위원 명단 전수)
- `pages/04_organization.md` — 조직 소개(조직도 텍스트)
- `pages/05_mi-intro.md` — MI 소개(워드마크·색상·다운로드)
- `pages/06_boards.md` — 게시판 11종(건수·최신글·테이블 접근성)
- `pages/07_board-detail-sample.md` — 게시글 상세 대표 샘플(article 구조·PDF 첨부)
- `pages/08_search.md` — 통합검색(폼/입력 label 접근성)
- `pages/09_english-site.md` — 영문판(lang 불일치·명칭 혼재·미번역)

### 기술 메타데이터 (tech/)
- `tech/tech_metadata.md` — 응답 헤더, 보안 헤더 부재, 쿠키, TLS, 성능, 기술스택/외부리소스, 메타태그, robots/sitemap
- `tech/broken_links_and_errors.md` — 404 동작, soft-404, 리다이렉트, 미검증 범위

### 스크린샷 (screenshots/)
- `screenshots/main_desktop.png` (1440×900) / `screenshots/main_mobile.png` (390×844)
- `screenshots/about_desktop.png` / `screenshots/about_mobile.png`
- `screenshots/members_desktop.png`
- `screenshots/policy_desktop.png`

## 후속 분석가를 위한 주요 사실 포인터 (판단 아님, 사실)

- **접근성**: 빈 H1 + H1 중복(모든 페이지 공통 레이아웃), 위원회 소개 본문 이미지 4개 alt 부재, 조직도 이미지 alt가 로고 텍스트 재사용, 갤러리 썸네일 background-image(대체텍스트 없음), 검색 입력 일부 label 미연결, 영문판 `lang="ko"`, 다운로드가 JS 버튼. 단 스킵네비·게시판 table caption/th는 양호.
- **기술/보안**: HSTS/X-Frame-Options/X-Content-Type-Options/CSP/Referrer-Policy/Permissions-Policy 전부 부재. 쿠키는 Secure/HttpOnly/SameSite 적용. DV 인증서. cache-control 오타(`must-revalicate`). 자체 404 없음(외부 가비아 평문 리다이렉트).
- **SEO**: sitemap.xml 루트 1개뿐, robots.txt는 네이버봇만 명시·Sitemap 지시어 없음, description/og가 사이트명과 동일(페이지별 고유 메타 부재), soft-404, X-UA-Compatible IE=edge 잔재.
- **성능**: 메인 HTML TTFB 0.15~0.24s/24KB로 빠름. 정적 자산 max-age=300(짧음)+no-cache. 게시판 데이터 AJAX 후속 로드. Core Web Vitals 정량 미측정(기술 분석가 Lighthouse 권장).
- **UX/IA**: 모바일 반응형 정상 동작(스크린샷 확인). fullPage.js 풀스크린 메인. 한영 메뉴 비대칭. 통합검색 콘텐츠/게시물 이원화.
- **콘텐츠/정책**: 핵심 정책 원문이 PDF 첨부로만 제공(본문 HTML 텍스트 부재). 정책자료 11건·보도자료 74건·분과위 281건으로 활동 기록 활발(최신 2026-06-04). 위원 명단/약력 충실.

## 미수집 · 실패 항목

| 항목 | 사유 |
|------|------|
| 게시글 상세 약 564건 본문/첨부 PDF | 샘플링(깊이보다 넓이, 부하 자제). 대표 1건만 수집. |
| 영문판 About Us/Leadership/Authority/Org Chart/Photo News 본문 | 범위·시간 한계. 메뉴 구조만 확인. |
| Core Web Vitals(LCP/CLS/INP) 정량 | 정찰 범위 외. 기술 분석가 Lighthouse 등으로 측정 권장. |
| 콘솔 오류 상세 | 추적 시작 타이밍 한계로 빈 결과. 새로고침 후 재확인 권장. |
| 첨부 PDF 내용 | 다운로드하지 않음(권한·예의 원칙). 필요 시 분석가가 개별 요청. |
| 전수 링크 유효성 점검 | 방문 페이지(200)만 확인. 565건 개별 링크/갤러리 썸네일 전수 미검증. |

## 재수집 안내

- 본 인벤토리는 1회 정찰 결과다. 후속 분석가가 특정 게시글 본문, 영문 콘텐츠 본문, 추가 스크린샷(특정 페이지/뷰포트)이 필요하면 해당 URL을 지정해 추가 수집을 요청할 수 있다.
- 전체 재크롤이 필요한 경우가 아니면 이 인벤토리를 공통 입력으로 재사용한다.
