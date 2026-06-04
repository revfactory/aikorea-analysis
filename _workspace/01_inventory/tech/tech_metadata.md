# 기술 메타데이터 — aikorea.go.kr

- 대상: https://www.aikorea.go.kr
- 수집 일시: 2026-06-05 05:14 (KST) — curl 헤더/성능 재확인, 브라우저 네트워크 기록 05:04
- 수집 도구: curl, Chrome 브라우저(read_network_requests), openssl
- 주의: 본 문서는 사실 기록이다. 좋다/나쁘다 판단은 후속 기술·보안 분석가의 몫. 단, 헤더 부재·404 동작 등 객관적 사실은 그대로 기록한다.

## 1. HTTP 응답 헤더 (메인 `/web/main.do`, HTML accept)

```
HTTP/2 200
server: nginx
date: Thu, 04 Jun 2026 20:14:05 GMT
content-type: text/html;charset=UTF-8
set-cookie: JSESSIONID=...; Path=/; Secure; HttpOnly; SameSite=Lax
content-language: ko-KR
cache-control: s-maxage=60, max-age=300, public, no-cache, no-transform, must-revalicate, post-check=0 pre-check=0
```

관찰 사실:
- HTTP/2 사용, 서버 `nginx`(버전 미노출).
- `cache-control` 값에 오타로 보이는 토큰 `must-revalicate`(정상 표기는 `must-revalidate`) 및 구식 토큰 `post-check=0 pre-check=0` 포함.
- 컨텐츠 협상: `Accept` 헤더에 따라 응답이 달라짐. `Accept: text/html`이면 `text/html`, 일반 curl(기본 Accept)에서는 `content-type: application/json` 으로 응답하는 경우 관찰됨(SSR+API 하이브리드 구조).
- `content-language: ko-KR` 고정.

## 2. 보안 헤더 점검 (부재 확인)

다음 보안 관련 응답 헤더는 **모두 미설정**(curl로 메인 페이지 응답에서 grep, 일치 없음):

| 헤더 | 상태 |
|------|------|
| Strict-Transport-Security (HSTS) | 없음 |
| X-Frame-Options | 없음 |
| X-Content-Type-Options | 없음 |
| Content-Security-Policy (CSP) | 없음 |
| Referrer-Policy | 없음 (단, HTML meta로 `referrer=always` 지정됨 — 가장 느슨한 값) |
| Permissions-Policy | 없음 |
| X-XSS-Protection | 없음 |

## 3. 쿠키

| 쿠키명 | 속성 | 비고 |
|--------|------|------|
| JSESSIONID | `Path=/; Secure; HttpOnly; SameSite=Lax` | 세션 쿠키. Secure/HttpOnly/SameSite 적용됨. 매 요청마다 신규 발급되는 정황(무상태 GET에도 Set-Cookie) |

- 추적/분석용 서드파티 쿠키 미발견(아래 외부 리소스 참조).

## 4. HTTPS / 전송 보안

- **HTTP→HTTPS 리다이렉트**: `http://www.aikorea.go.kr/` → 302 → `https://www.aikorea.go.kr/web/main.do?editorType=smarteditor&screenTp=USER`
  - 평문(HTTP)도 응답 후 HTTPS로 유도하나, HSTS 헤더가 없어 브라우저 강제 업그레이드는 미보장.
- **루트 리다이렉트**: `https://www.aikorea.go.kr/` → 302 → `/web/main.do?editorType=smarteditor&screenTp=USER` (쿼리 파라미터 노출).
- **TLS 인증서**:
  - 발급자: Sectigo RSA Domain Validation Secure Server CA (DV 인증서)
  - 주체(CN): www.aikorea.go.kr
  - 유효기간: 2025-09-24 ~ 2026-10-25 (수집 시점 기준 유효)
  - 정부 사이트이나 DV(도메인 검증) 등급 인증서 사용.

## 5. 404 / 오류 응답 동작

| 케이스 | 결과 |
|--------|------|
| 존재하지 않는 `.do` 경로 (`/web/nonexistent-12345.do`) | **302 리다이렉트 → `http://errdoc.gabia.io/404.html`** (외부 가비아 호스팅 에러 페이지, HTTP 평문으로 다운그레이드. 자체 404 페이지 부재) |
| 잘못된 menu_cd (`content.do?menu_cd=999999`) | **HTTP 200** (size≈10KB). 오류 코드 없이 정상 응답 — soft 404 |

관찰: 깨진 링크에 대해 적절한 4xx 상태코드 대신 외부 도메인 리다이렉트 또는 200 응답을 반환. 검색엔진·접근성·UX 측면에서 후속 분석 필요.

## 6. 성능 지표 (curl, 메인 HTML)

| 측정 | run1 | run2 | run3 |
|------|------|------|------|
| DNS | 4.7ms | 4.1ms | 3.6ms |
| TLS handshake(완료) | 71.6ms | 70.8ms | 134.3ms |
| TTFB | 154.6ms | 166.6ms | 238.4ms |
| Total | 156.3ms | 167.2ms | 239.2ms |
| HTML 크기 | 24,467 B | 동일 | 동일 |

- 메인 HTML 응답은 빠름(TTFB 약 0.15~0.24초). 단, 게시판 데이터는 추가 AJAX(POST)로 후속 로드되어 체감 LCP에는 별도 측정 필요(브라우저 네트워크 기록상 mainBbsList.do 6회 호출).
- Core Web Vitals(LCP/CLS/INP)는 본 정찰에서 정량 측정하지 않음 — 기술 분석가가 Lighthouse 등으로 측정 권장. fullPage.js 풀스크린 히어로(대형 인물 사진 webp)가 LCP 후보.

## 7. 기술 스택 / 외부 리소스 (메인 페이지 네트워크 기록, 총 61개 요청)

### 자바스크립트 라이브러리 (self-hosted, `/static/`)
- jQuery 3.7.1 (`gioinfra-jquery-3.7.1.js`)
- fullPage.js 2.9.7 (풀스크린 스크롤 섹션)
- Swiper (`swiper.min.js` / `swiper.min.css`)
- nice-select (`jquery.nice-select.js` — 커스텀 select)
- scrollbar 플러그인
- 자체 스크립트: `main.js`, `design.js`, `headerCommonTld.js`

### CSS
- `style.css`, `main.css`, `common.css`, `nice-select.css`, `swiper.min.css`, `scrollbar.css`, `fullPage2.9.7.css`
- **`tw.build.css`** — Tailwind CSS 빌드 산출물(유틸리티 클래스 사용 정황)

### 폰트 (self-hosted, woff2)
- **PretendardGOV** (정부 전용 변형) — Bold/Regular/Medium/ExtraBold/SemiBold/Light
- **GmarketSans** (`GmarketSans.css`)
- 외부 폰트 CDN(Google Fonts 등) 미사용 — 모두 자체 호스팅.

### 분석/추적 도구
- **미발견.** Google Analytics, GTM, Meta Pixel 등 서드파티 추적 스크립트 네트워크 기록에 없음(개인정보 측면 양호 정황).

### 외부 도메인 요청
- 정적/이미지 리소스 전부 `aikorea.go.kr` 자체 출처(`/static/`, `/attach/`). 외부 CDN 의존 없음.
- 인라인 외부 링크는 youtube.com, x.com 뿐(소셜 채널, 사용자가 클릭 시 이동).

### 정적 자산 캐싱 (예: `/static/css/main.css`)
```
content-type: text/css
etag: W/"53068-1776145524000"
last-modified: Tue, 14 Apr 2026 05:45:24 GMT
cache-control: s-maxage=60, max-age=300, public, no-cache, no-transform, must-revalicate, ...
```
- ETag·Last-Modified 제공(조건부 요청 가능, 재방문 시 304 다수 관찰).
- 단, `max-age=300`(5분)으로 정적 자산치고 짧고 `no-cache` 동반 — 정적 리소스 장기 캐싱 미적용. 자산 URL에 버전 쿼리(`?ver=`)가 비어 있음(`?ver=` / `?v=`).

## 8. 메타태그 (메인 `/web/main.do`, 렌더링 후 DOM)

| 항목 | 값 |
|------|-----|
| `<html lang>` | `ko` (한글판). **영문판도 `ko`로 동일 — 불일치** |
| charset | UTF-8 |
| title | 국가인공지능전략위원회 |
| description | 국가인공지능전략위원회 |
| keywords | 국가인공지능전략위원회,국가인공지능,전략위원회,인공지능전략위원회 |
| author | (빈 값) |
| robots | `index` |
| copyright | `copyrights 2021 GIOINFRA corp.` (구축 업체, 연도 2021) |
| viewport | `width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=yes` (maximum-scale=1.0 — 확대 제한 정황, 단 user-scalable=yes) |
| X-UA-Compatible | `IE=edge` (구형 IE 호환 잔재) |
| og:type | website |
| og:url | https://aikorea.go.kr |
| og:title | 국가인공지능전략위원회 |
| og:description | 국가인공지능전략위원회 |
| og:image | https://aikorea.go.kr/BB2EA8F120FB4DF89749696287F0170A.jpg (1200×630) |
| format-detection | telephone=no |
| apple-mobile-web-app-capable | yes |
| referrer | always (느슨한 referrer 정책) |

- Twitter Card 메타 미발견. canonical 링크 미확인(별도 점검 권장).
- description/og:description이 사이트명과 동일 — 페이지별 고유 설명 부재.

## 9. robots.txt / sitemap.xml

### robots.txt (`/robots.txt`)
```
User-agent: Yeti
Allow:/
```
- **네이버 봇(Yeti)만 명시.** Googlebot 등 다른 크롤러 대상 규칙 없음(기본 허용으로 동작하나 명시적 관리 안 됨).
- `Sitemap:` 지시어 없음. `Allow:/`에 공백 누락(`Allow: /` 표준 표기와 다름).

### sitemap.xml (`/sitemap.xml`)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset ...>
<!-- Generated by Web-Site-Map.com -->
<url><loc>http://www.aikorea.go.kr/</loc><changefreq>daily</changefreq><priority>1.00</priority></url>
</urlset>
```
- 외부 무료 도구(Web-Site-Map.com) 생성. **루트 URL 1개만 등록** — 실질적 사이트맵 미비.
- `<loc>`가 `http://`(평문)로 등록됨(실제 사이트는 https).

## 10. 콘솔 오류

- 메인 페이지 로드 시 JS 콘솔 오류 패턴(error/warn/fail/404/mixed/deprecated) 매칭 결과 특이사항 미수집(추적 시작 타이밍 한계로 빈 결과). 후속 분석가가 페이지 새로고침 후 재확인 권장.
