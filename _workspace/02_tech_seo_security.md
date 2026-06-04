# 기술·성능·SEO·보안 진단 — aikorea.go.kr (국가인공지능전략위원회, NAIS)

- 대상: https://www.aikorea.go.kr
- 진단일: 2026-06-05 (KST)
- 분석가: tech-seo-security (site-analysis-team)
- 입력: `_workspace/01_inventory/` (INVENTORY.md, tech/tech_metadata.md, tech/broken_links_and_errors.md, sitemap.md, pages/)
- 방법: 인벤토리 사실 + **비침투 라이브 재검증**(curl 응답 헤더·성능·압축, openssl TLS, WebFetch 수준 조회). 취약점 스캐닝·부하·우회 일절 미수행.
- 측정 한계: Core Web Vitals(LCP/CLS/INP) 브라우저 계측은 연결된 단일 브라우저 선택 확인 절차가 필요해 본 라운드에서 정량 미계측 → 리소스 구성·전송 지표로 정성 추정. 해당 항목은 "측정 불가/추정"으로 명시.

---

## 요약 (영역별 한 줄 + 종합 위험도)

| 영역 | 한 줄 진단 | 점수 |
|------|-----------|------|
| 기술 스택·구조 | nginx + GIOINFRA CMS(SSR+AJAX 하이브리드), self-host 자산·무추적으로 견고하나 `viewport=width=1600` 고정·소프트404 등 구조적 약점 | 68/100 |
| 성능 | TTFB 빠름(0.18~0.24s)·HTML gzip(94KB→17KB) 우수, 그러나 **CSS/JS 무압축**·정적자산 `max-age=300`·요청 61개로 캐싱/전송 최적화 미흡 | 62/100 |
| SEO | robots/sitemap 사실상 미관리·페이지별 고유 메타 부재·**og:image 깨짐**·canonical/hreflang/JSON-LD 전무·soft404 | 41/100 |
| 보안 | HTTPS·DV인증서·쿠키 3속성은 양호하나 **보안 응답 헤더 6종 전부 부재**·404 외부 평문 리다이렉트·느슨한 referrer | 48/100 |
| **종합** | 인프라 기본기와 무추적 개인정보 태세는 양호하나, **공공(.go.kr) 기준 권고/관행 대비 보안 헤더·SEO·검색 색인 관리가 전반적으로 미흡** | **54/100 (주의)** |

종합 위험도: **중(中) — 즉각적 침해 위험은 낮으나, 정부 사이트로서 신뢰·검색노출·클릭재킹/스니핑 방어 측면의 권고 미준수가 다수.**

---

## 1. 기술 스택·구조

| 항목 | 관찰값 | 근거 |
|------|--------|------|
| 웹서버 | nginx (버전 미노출 — 양호) | 라이브 `server: nginx`, X-Powered-By 등 버전 누출 헤더 없음 |
| 프로토콜 | HTTP/2 (HTTPS), 평문 HTTP는 302로 HTTPS 유도 | 라이브 헤더 |
| CMS/구축 | GIOINFRA corp. 구축 CMS (`copyright 2021 GIOINFRA corp.`) | tech_metadata.md §8 |
| 렌더링 | SSR(서버 HTML) + AJAX 하이브리드. 메인 게시판은 `mainBbsList.do`(POST) 6회 후속 로드 | tech_metadata.md §6, pages/01_main.md |
| 라우팅 | `menu_cd` 쿼리 기반 (`content.do` / `brdList.do` / `garList.do` / `brdDetail.do` / `search.do`) | sitemap.md |
| 콘텐츠 협상 | `Accept` 헤더에 따라 동일 URL이 text/html 또는 application/json 응답 | tech_metadata.md §1 |
| JS 라이브러리 | jQuery 3.7.1, fullPage.js 2.9.7, Swiper, nice-select, scrollbar (모두 self-host `/static/`) | tech_metadata.md §7 |
| CSS | style/main/common/nice-select/swiper/scrollbar/fullPage + **`tw.build.css`(Tailwind 빌드)** | tech_metadata.md §7 |
| 폰트 | **PretendardGOV**(정부 전용) + GmarketSans, self-host woff2, 외부 CDN 미사용 | tech_metadata.md §7 |
| 추적/분석 | **미발견**(GA/GTM/Pixel 없음) — 개인정보 측면 우수 | tech_metadata.md §7 |
| 외부 의존 | 정적자산 전부 자체 출처. 인라인 외부링크는 youtube.com·x.com뿐 | tech_metadata.md §7 |
| 도메인 정규화 | apex `aikorea.go.kr` → `www`로 302 정규화(양호) | 라이브: `https://aikorea.go.kr/` → 302 → www |

구조적 강점: self-hosting(외부 CDN/추적 의존 0), 정부 전용 폰트, 서버 버전 비노출, 무추적.
구조적 약점: (a) **렌더링된 정적 HTML이 항상 `<meta name="viewport" content="width=1600">`** 를 내보냄 — JS가 런타임에 `width=device-width`로 교체하는 구조(아래 발견 [주요] 참조). (b) 자체 404 페이지 부재로 오류 처리가 외부 가비아 도메인에 의존. (c) jQuery+fullPage.js+Swiper 등 레거시 라이브러리 다수 동시 로드.

---

## 2. 성능 (지표 + 병목 + 권고)

### 측정값 (라이브 curl, 메인 HTML)
| 측정 | run1 | run2 | run3 |
|------|------|------|------|
| TTFB | 0.235s | 0.227s | 0.180s |
| Total | 0.328s | 0.321s | 0.265s |
| HTML(원본, identity) | 94,059 B | 동일 | 동일 |
| HTML(gzip) | **17,399 B** (≈81.5% 절감) | — | — |

### 전송/캐싱 지표
| 항목 | 현황 | 판정 |
|------|------|------|
| HTML 압축 | **gzip 적용** (94KB→17.4KB) | 양호 |
| CSS/JS 압축 | **미적용** — `/static/css/main.css`·`/static/js/main.js` 응답에 `content-encoding` 없음(JS 15,342 B 원본 그대로 전송) | 미흡 |
| 정적자산 캐싱 | `cache-control: ... max-age=300, ... no-cache` (5분 + no-cache), 버전 쿼리(`?ver=`) 빈 값 | 미흡 |
| 조건부 요청 | ETag·Last-Modified 제공(재방문 304 가능) | 양호 |
| 요청 수 | 메인 약 61개(인벤토리 네트워크 기록) | 보통~다소 많음 |
| 게시판 데이터 | `mainBbsList.do` POST 6회 후속 로드 | LCP·체감 지연 요인 |

### Core Web Vitals (측정 불가 → 정성 추정)
- **LCP**: fullPage.js 풀스크린 히어로의 대형 인물 사진(webp)이 LCP 후보. HTML TTFB는 빠르나, 히어로 이미지 크기와 webp 디코드, fullPage 초기화가 LCP를 좌우. **정량 미계측**.
- **CLS**: fullPage.js 풀스크린 섹션 + 게시판 AJAX 후속 삽입 → 데이터 도착 시 카드 영역 레이아웃 이동 가능성. **정량 미계측**.
- **INP**: jQuery 기반 인터랙션. 정량 미계측.
- 권장: Lighthouse / PageSpeed Insights / CrUX로 실측(본 비침투 라운드 범위 외).

### 병목
1. **정적 CSS/JS 무압축** — 텍스트 자산 압축 미적용은 즉시 개선 가능한 전송 낭비.
2. **정적자산 단기 캐싱(max-age=300 + no-cache)** — 버전 해시 없는 자산을 5분 캐시 + no-cache로 매번 재검증, 재방문 비용 증가.
3. **게시판 데이터 AJAX 후속 로드(6회)** — 초기 화면의 핵심 콘텐츠가 2차 라운드트립에 묶임 → 체감 로딩·LCP·이탈 영향(→ ux-ia 공유).
4. **레거시 라이브러리 번들**(jQuery+fullPage+Swiper+nice-select+scrollbar 동시) — JS 실행/파싱 비용.

### 권고
- nginx `gzip`/`brotli`를 `text/css`·`application/javascript`에도 활성화.
- 정적자산에 **파일명/쿼리 버전 해시 + `Cache-Control: max-age=31536000, immutable`** 적용(불변 자산 장기 캐싱), HTML만 단기.
- `must-revalicate` 오타 수정(아래 발견 참조) — 캐시 지시어 신뢰성 회복.
- 히어로 이미지 `fetchpriority="high"`/프리로드, 게시판 초기 1건은 SSR로 인라인하여 LCP 콘텐츠 1차 전송에 포함 검토.

---

## 3. SEO (항목/현황/판정/권고 표)

| 항목 | 현황 (근거) | 판정 | 권고 |
|------|------------|------|------|
| title | **전 페이지 동일** "국가인공지능전략위원회" (라이브 교차확인: 위원회소개·정책자료·통합검색 모두 동일, 영문판만 "NAIS") — 페이지별 고유 없음 (ux-ia 교차확인 일치) | 미흡 | 페이지별 `{페이지명} | 국가인공지능전략위원회` 패턴 |
| meta description | "국가인공지능전략위원회"(사이트명과 동일, 모든 페이지) | 미흡 | 페이지별 60~120자 고유 설명 |
| meta keywords | 존재(레거시, 검색엔진 미사용) | 무영향 | 유지/제거 무관 |
| 헤딩 위계 | **H1 3개(빈 H1 1개 포함)·동일 H2 그룹 반복** (pages/01_main.md) | 불량 | H1 단일화·빈 H1 제거 (접근성과 동시 이슈 → accessibility 공유) |
| Open Graph | og:type/url/title/description/image 존재하나 title·description이 사이트명과 동일 | 보통 | 페이지별 og 값 |
| **og:image** | **깨짐** — `https://aikorea.go.kr/BB2EA8F120FB4DF89749696287F0170A.jpg` 라이브 HEAD = **302 → http://errdoc.gabia.io/404.html** | **불량** | 유효한 절대경로 og:image(1200×630) 교체 — SNS 공유 시 썸네일 미표시 |
| Twitter Card | 미발견 | 미흡 | `twitter:card=summary_large_image` 추가 |
| canonical | **부재** (라이브 HTML grep 결과 `rel=canonical` 없음) | 미흡 | 자기참조 canonical(쿼리 정규화) |
| hreflang | **부재** (한/영 2개 언어 운영하나 hreflang 상호 링크 없음) | 미흡 | ko/en hreflang 상호 지정 |
| 구조화 데이터(JSON-LD) | **0개** (라이브 grep `application/ld+json` 0) | 미흡 | GovernmentOrganization/BreadcrumbList JSON-LD |
| robots meta | `index` (색인 허용 — 정상) | 양호 | 유지 |
| robots.txt | `User-agent: Yeti`(네이버봇)만, `Sitemap:` 지시어 없음, `Allow:/` 공백 누락 | 미흡 | 전 크롤러 대상 규칙 + `Sitemap:` 절대 URL |
| sitemap.xml | **루트 URL 1개만 등록**, `<loc>`가 `http://`(평문), 외부 무료도구(Web-Site-Map.com) 산출 | **불량** | 전 페이지(한/영) 포함 https sitemap 생성·자동화 |
| 색인 가능성(soft-404) | 존재하지 않는 menu_cd가 **HTTP 200**(라이브: `menu_cd=999999` → 200, 1153 B) | 불량 | 부재 콘텐츠는 404 반환(검색엔진 빈/중복 색인 방지) → content-policy 공유 |
| 목록 콘텐츠 렌더링 | **게시판 목록·미리보기가 AJAX 후속 주입** — 서버 HTML에 게시물 행 없음(라이브: `brdList.do?menu_cd=000011` 서버 HTML에 알려진 글 제목 "행동계획"·"대한민국" 0회, `<tr>` 4개=헤더 골격뿐, ajax 참조 16회) | 불량 | 목록·미리보기를 SSR로 인라인하거나 동적 렌더링 제공 — 비-JS 크롤러 색인 누락 방지 (content-policy 검증과 일치) |
| 영문판 lang | **`<html lang="ko">`** (영문 페이지인데 한국어) (라이브 확인) | 불량 | 영문판 `lang="en"` (접근성과 동시 이슈) |
| X-UA-Compatible | `IE=edge`(구형 IE 잔재) | 경미 | 제거 |

SEO 종합: **검색엔진이 사이트를 발견·구분·미리보기하는 4대 기둥(sitemap·고유 메타·canonical·구조화데이터)이 모두 부실.** 정부 핵심 정책 사이트로서 검색 노출·SNS 공유 품질이 실제 트래픽 손실로 직결.

---

## 4. 보안 태세 (헤더·쿠키·HTTPS 표 + 발견)

### 보안 응답 헤더 (라이브 재검증 — 메인 페이지)
| 헤더 | 상태 | 영향(추정) |
|------|------|-----------|
| Strict-Transport-Security (HSTS) | **부재** | 평문 HTTP 다운그레이드/SSL스트립 방어 미보장 |
| X-Frame-Options | **부재** | 클릭재킹(iframe 삽입) 방어 없음 |
| X-Content-Type-Options | **부재** | MIME 스니핑 가능 |
| Content-Security-Policy | **부재** | XSS·인젝션 완화 계층 없음 |
| Referrer-Policy | **부재** (HTML meta는 `referrer=always` — 가장 느슨) | 외부로 전체 referrer 유출 |
| Permissions-Policy | **부재** | 브라우저 기능 권한 미제한 |
| X-XSS-Protection | 부재 | (현대 브라우저 영향 적음, 참고) |

> 라이브 재확인 결과 7종 모두 응답에 없음. (근거: `curl -D - | grep` 일치 없음, 2026-06-05.)

### 쿠키
| 쿠키 | 속성 | 판정 |
|------|------|------|
| JSESSIONID | `Path=/; Secure; HttpOnly; SameSite=Lax` | **양호**(3속성 적용). 단 무상태 GET마다 신규 발급(라이브: 캐시버스팅 GET 2회 모두 상이한 JSESSIONID Set-Cookie) → 세션 sprawl(경미) |

서드파티 추적 쿠키 미발견(개인정보 양호).

### HTTPS / 전송 보안
| 항목 | 관찰 | 판정 |
|------|------|------|
| 전 구간 HTTPS | 메인·정적자산 HTTPS, HTTP→302→HTTPS | 양호(단 HSTS 부재로 강제성 약함) |
| TLS 인증서 | 발급 Sectigo **DV**(도메인검증), CN=www.aikorea.go.kr, 유효 2025-09-24~2026-10-25 (라이브 openssl 확인) | 보통 — **정부 사이트로서 DV 등급은 낮음**(OV/EV 권고) |
| 혼합 콘텐츠 | 자산 자체 출처 https로 직접 mixed content 미관찰. 단 **404·og:image·sitemap `<loc>`가 http(평문)** 로 외부/등록됨 | 주의 |
| 404 처리 | 부재 경로 → **302 → http://errdoc.gabia.io/404.html** (외부 도메인·평문 다운그레이드, 라이브 확인) | 불량 |

### 정보 노출
- 서버 버전 비노출(nginx만), X-Powered-By 없음 — **양호**.
- 루트/오류 리다이렉트에 쿼리 파라미터(`editorType`, `screenTp`) 노출 — 민감정보 아님(경미).

보안 종합: **암호화·쿠키·버전 은닉의 기본기는 갖췄으나, "한 줄 추가로 되는" 보안 응답 헤더 6종을 전부 빠뜨려** 클릭재킹·MIME스니핑·referrer유출·다운그레이드에 대한 방어 계층이 비어 있음. 공공기관 권고(행안부/KISA 보안 가이드) 대비 미달.

---

## 발견 사항

### [치명] og:image 링크 깨짐 — SNS 공유 미리보기 실패 — (SEO)
- 근거: 라이브 HEAD `https://aikorea.go.kr/BB2EA8F120FB4DF89749696287F0170A.jpg` → **302 → http://errdoc.gabia.io/404.html**(존재하지 않음). og:image:width/height(1200×630)는 선언되어 있으나 실제 이미지 부재.
- 영향: 카카오톡·X·페이스북 등 모든 SNS/메신저 공유 시 대표 이미지가 표시되지 않음 → 정부 핵심 정책 홍보 확산력 저하.
- 권고: 실제 존재하는 대표 이미지(1200×630)로 og:image 절대 URL 교체 후 SNS 디버거로 캐시 갱신.

### [주요] 정적 HTML viewport가 `width=1600` 고정 (JS가 런타임 교체) — (기술/성능)
- 근거: 라이브 curl(모바일·데스크톱 UA 동일) 결과 서버 HTML은 항상 `<meta name="viewport" content="width=1600">`. 인벤토리 DOM 캡처는 `width=device-width...`(런타임 JS가 교체). 즉 **JS 실행 전/실패 시 모바일에서 1600px 데스크톱 레이아웃**으로 렌더.
- 영향: JS 차단·저사양·로드 실패 시 모바일 사용자에게 가로 스크롤/축소 레이아웃. 초기 페인트 단계 레이아웃 흔들림(CLS·체감) → ux-ia 모바일 사용성과 직결.
- 권고: 서버 HTML에 처음부터 `width=device-width, initial-scale=1`을 내보내고 JS 교체 제거. (※ 인벤토리에는 `maximum-scale=1.0`도 기록 — 확대 제한은 접근성 이슈로 accessibility와 교차.)

### [주요] 보안 응답 헤더 6종 전부 부재 — (보안)
- 근거: 라이브 재검증 — HSTS/X-Frame-Options/X-Content-Type-Options/CSP/Referrer-Policy/Permissions-Policy 모두 응답에 없음(grep 일치 0).
- 영향: 클릭재킹·MIME스니핑·XSS완화·HTTPS강제·referrer유출 방어 계층 부재. 정부 사이트 보안 권고 미준수.
- 권고: nginx에 단계적 추가 — 우선 `X-Frame-Options: SAMEORIGIN`(또는 CSP frame-ancestors), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Strict-Transport-Security: max-age=31536000; includeSubDomains`(평문 의존 정리 후), 이후 CSP를 report-only로 도입.

### [주요] sitemap.xml 미관리·robots.txt 부실 — 검색 색인 누락 — (SEO)
- 근거: 라이브 — sitemap.xml에 루트 1개만(외부 무료도구, `<loc>`가 http). robots.txt는 네이버봇(Yeti)만 명시, `Sitemap:` 지시어 없음.
- 영향: 565건 게시물·정책문서·영문판이 사이트맵 미등록 → 검색엔진 발견성 저하. Googlebot 등 명시적 관리 부재.
- 권고: 전 페이지(한/영) https sitemap 자동 생성, robots.txt에 `Sitemap:` 절대 URL + 전 크롤러 규칙. → content-policy 공유.

### [주요] 게시판 목록·미리보기가 AJAX 후속 렌더 — 비-JS 크롤러 색인 누락 — (SEO/기술)
- 근거: 라이브 — 게시판 목록 `brdList.do?menu_cd=000011`(정책자료) **서버 HTML(55,788 B)에 게시물 행이 없음**: 알려진 글 제목 "행동계획"·"대한민국" 각 0회, `<tr>` 4개(테이블 헤더 골격), ajax 호출 참조 16회. 즉 목록 행은 JS 실행 후 AJAX(`brdList.do`/`mainBbsList.do` POST)로 주입됨. (출처: content-policy 분석가 라이브 WebFetch 공유를 본 분석가가 비침투 curl로 교차 검증, 2026-06-05.)
  - **단 게시글 상세는 SSR 확인**: `brdDetail.do?num=523` 서버 HTML(63,905 B)에 글 제목 "행동계획" 3회·작성일/조회수 메타 2회 존재 → **개별 상세 페이지는 서버 렌더(색인 가능)**. content-policy 공유 중 "본문이 비어 반환"은 *목록/미리보기*에 해당하며, *상세 페이지*는 본문이 서버에 존재함을 측정으로 정정.
- 영향: 현대 Googlebot은 JS 실행으로 목록 색인 가능하나, **robots.txt에 유일 명시된 네이버봇(Yeti) 등 JS 미실행·제한 크롤러는 목록·미리보기를 빈 페이지로 인식**. 게다가 sitemap이 루트 1개뿐이라 상세 URL 발견 경로도 부재 → 565건·정책문서가 검색 발견성에서 실질 누락 위험. (sitemap 미관리 발견과 복합.)
- 권고: 게시판 목록·메인 미리보기를 SSR로 인라인(최소 1페이지)하거나 동적 렌더링(prerender) 제공. 동시에 전 상세 URL을 sitemap에 등록하여 발견 경로 확보. (PDF-only 정책 본문 이슈는 content-policy 보고서 참조.)
- 보강(ux-ia 교차확인): 사이트 *내부* 통합검색도 콘텐츠검색/게시물검색으로 이원화되어, "행동계획" 질의 시 콘텐츠검색 0건·게시물검색 147건. `content.do` 본문이 빈약해 내부 콘텐츠검색이 사실상 무력화된 정황 → 외부 크롤러뿐 아니라 사이트 내 탐색에서도 핵심 콘텐츠 도달성이 낮음(단 내부 검색 색인 ≠ 외부 SEO 색인, 구분).

### [주요] soft-404 (부재 콘텐츠가 HTTP 200) — (SEO/기술)
- 근거: 라이브 `content.do?menu_cd=999999` → **200**(1153 B). 부재 경로 `.do`는 별도로 **302 → 외부 가비아 평문 404**.
- 영향: 검색엔진이 빈/중복 페이지를 색인, 사용자가 오류를 인지 못함. 깨진 링크가 외부 도메인으로 이탈.
- 권고: 부재 메뉴/콘텐츠는 자체 404 페이지 + HTTP 404 반환. 외부 가비아 의존 제거(자체 404, https).

### [주요] 정적 CSS/JS 무압축 — 전송 낭비 — (성능)
- 근거: 라이브 — `/static/css/main.css`·`/static/js/main.js` 응답에 `content-encoding` 없음(JS 15,342 B 원본 전송). HTML은 gzip 적용(94KB→17.4KB).
- 영향: 텍스트 자산 전송량 증가 → 모바일·저대역 환경 로딩 지연·이탈(→ ux-ia 공유).
- 권고: nginx gzip/brotli를 `text/css`·`application/javascript` MIME에 확장. 정적자산 장기 캐싱(버전 해시) 병행.

### [경미] 영문판 `lang="ko"` — (SEO/접근성)
- 근거: 라이브 `/eng/web/main.do` → `<html lang="ko" dir="ltr">`.
- 영향: 검색엔진 언어 오인식, 스크린리더 한국어 발음(접근성). KWCAG 6.4.2 교차.
- 권고: 영문판 `lang="en"`. → accessibility 교차 확인.

### [경미] cache-control 오타 `must-revalicate` + 구식 토큰 — (성능/기술)
- 근거: 라이브 — `cache-control: s-maxage=60, max-age=300, public, no-cache, no-transform, must-revalicate, post-check=0 pre-check=0`. `must-revalidate` 오타, `post-check/pre-check`는 폐기된 IE 전용 토큰.
- 영향: 오타 토큰은 무시되어 `must-revalidate` 의도 미적용. 캐시 정책 신뢰성 저하.
- 권고: `must-revalidate`로 정정, 레거시 토큰 제거, 자산/문서별 캐시 정책 분리.

### [경미] TLS DV 인증서 + JSESSIONID 무상태 재발급 — (보안)
- 근거: openssl — Sectigo DV. 라이브 — 캐시버스팅 GET마다 상이한 JSESSIONID Set-Cookie.
- 영향: 정부 사이트로서 DV는 신원보증 낮음. 불필요한 세션 발급(경미).
- 권고: OV/EV 인증서 검토. 무상태 GET에서 불필요 세션 생성 억제.

### [경미] X-UA-Compatible IE=edge / referrer=always 메타 — (기술/보안)
- 근거: 라이브 HTML — `meta name="referrer" content="always"`, 인벤토리 — `X-UA-Compatible IE=edge`.
- 영향: 레거시 잔재. referrer=always는 외부로 전체 URL 유출(가장 느슨).
- 권고: IE 메타 제거, referrer 정책을 `strict-origin-when-cross-origin`으로.

---

## 점수 (영역별 + 종합, 근거 포함)

| 영역 | 점수 | 근거 요약 |
|------|------|----------|
| 기술 스택·구조 | **68 / 100** | (+) self-host·무추적·버전은닉·HTTP2·apex정규화 (−) viewport 1600 고정/JS교체, soft404, 외부404 의존, 레거시 라이브러리 |
| 성능 | **62 / 100** | (+) TTFB 빠름·HTML gzip(81%)·ETag (−) CSS/JS 무압축, max-age=300+no-cache, AJAX 6회 후속, CWV 미실측 |
| SEO | **41 / 100** | (+) 색인 허용·OG 기본 존재·상세 페이지 SSR (−) **목록·미리보기 AJAX 렌더(비-JS 크롤러 누락)**·sitemap 1개·캐노니컬/hreflang/JSON-LD 전무·og:image 깨짐·페이지별 메타 부재·soft404·영문 lang 오류 |
| 보안 | **48 / 100** | (+) HTTPS·쿠키 3속성·버전은닉·무추적 (−) 보안헤더 6종 전무·404 평문 외부 리다이렉트·DV 인증서·느슨한 referrer |
| **종합** | **54 / 100 (주의/中)** | 인프라 기본기·개인정보 태세는 양호하나, 공공(.go.kr) 권고 대비 보안헤더·SEO·검색색인·오류처리가 광범위하게 미흡. 즉시 개선 가능한 저비용 항목(헤더 추가·압축·sitemap·og:image)이 다수. |

### 우선 개선 순위 (효과 대비 비용)
1. **즉시(저비용·고효과):** og:image 교체, 보안 헤더 6종 추가, CSS/JS 압축 활성화, cache-control 오타 수정.
2. **단기:** 전 페이지 sitemap.xml 자동생성 + robots.txt 정비, 페이지별 title/description, 자체 404(https)·soft404 제거.
3. **중기:** canonical·hreflang·JSON-LD 도입, viewport 서버측 정정, **게시판 목록·미리보기 SSR/prerender화(비-JS 크롤러 색인 확보)**, 정적자산 버전해시 장기캐싱, CWV 실측 후 LCP/CLS 튜닝, OV/EV 인증서 검토.

---

## 측정 한계 / 미검증
- Core Web Vitals(LCP/CLS/INP) 정량 미계측(브라우저 선택 확인 절차 필요 + 비침투 라운드 범위). Lighthouse/PSI/CrUX 실측 권장.
- 게시물 565건 개별 링크·갤러리 썸네일·첨부 PDF 유효성 전수 미점검(부하 자제). og:image·정적자산 등 핵심 자산만 표적 검증.
- 콘솔 오류 상세는 본 라운드 미수집(인벤토리도 빈 결과). 브라우저 재계측 시 보완 권장.
