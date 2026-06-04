---
name: tech-seo-security-audit
description: "웹사이트의 기술 구조·로딩 성능·검색최적화(SEO)·기본 보안 태세를 비침투적으로 진단하는 절차. 기술 스택, Core Web Vitals, 메타태그/구조화 데이터, HTTPS·보안 헤더·쿠키, 깨진 링크를 점검하여 _workspace/02_tech_seo_security.md 를 생성한다. 기술 진단, 성능 분석, SEO 점검, 보안 헤더/HTTPS 점검이 필요할 때 사용."
---

# Tech / SEO / Security Audit — 기술·성능·검색·보안 진단 절차 (비침투)

사이트의 **기술적 건전성**을 진단한다. 공개적으로 관찰 가능한 정보만 사용한다.

## 비침투 원칙 (최우선)
관찰·조회만 한다. 인증 우회, 페이로드 주입, 부하 테스트, 디렉토리 브루트포스 등 **공격적 행위는 절대 하지 않는다.** 정부 사이트는 특히 보수적으로 접근한다. WAF 차단을 만나면 차단 사실만 기록하고 우회하지 않는다. 이 원칙을 어기면 분석이 아니라 침해다.

## 4개 영역
### 1. 기술 스택·구조
- 프레임워크/CMS, 렌더링 방식(SSR/CSR), URL 구조, 외부 의존 리소스(폰트·스크립트·분석도구).
- 단서: HTTP 헤더(Server, X-Powered-By), HTML 패턴, 스크립트 경로, 쿠키 이름.

### 2. 성능 (Core Web Vitals 관점)
- LCP(최대 콘텐츠 렌더), CLS(레이아웃 이동), INP(상호작용 반응) 체감.
- 리소스 총량·요청 수, 이미지 최적화(포맷·크기·lazy-load), 캐싱(Cache-Control)·압축(gzip/br) 헤더.
- 방법: 브라우저 read_network_requests로 리소스 크기·수·헤더 수집, 콘솔 오류 확인.

### 3. SEO
| 항목 | 점검 |
|------|------|
| title / meta description | 페이지별 고유·적절한가 |
| 헤딩 위계 | h1 단일·논리적 위계인가 |
| Open Graph / 구조화 데이터 | og:*, JSON-LD 존재·유효한가 |
| robots.txt / sitemap.xml | 존재하고 올바른가, 색인 차단 오류 없는가 |
| canonical / 언어 | rel=canonical, hreflang(다국어 시) |
| 색인 가능성 | noindex 오용, 검색엔진 접근 차단 여부 |

### 4. 보안 태세 (비침투 관찰)
| 항목 | 점검 |
|------|------|
| HTTPS | 전체 HTTPS·인증서 유효·HTTP→HTTPS 리다이렉트 |
| 보안 헤더 | CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy |
| 쿠키 | Secure / HttpOnly / SameSite 속성 |
| 정보 노출 | 서버·프레임워크 버전 노출, 디렉토리 리스팅, 혼합 콘텐츠(mixed content) |

## 절차
1. `_workspace/01_inventory/tech_metadata.md`를 읽어 헤더·성능·리소스를 파악한다.
2. 부족하면 브라우저로 직접 network/console을 수집하고, WebFetch로 robots.txt·sitemap.xml을 확인한다.
3. 각 영역을 점검하며 **추정이 아니라 측정값**(응답 헤더·네트워크 기록·콘솔 로그)을 근거로 삼는다.
4. 발견을 심각도(치명/주요/경미)로 분류한다. 보안은 "관찰된 사실 → 추정 위험"을 명시하되 과장하지 않는다.
   - 치명: HTTPS 미적용, 심각한 정보 노출, 핵심 페이지 색인 차단 오류
   - 주요: 주요 보안 헤더 다수 누락, 성능 심각 저하, SEO 기본 요소 부재
   - 경미: 권고 수준 헤더 누락, 최적화 여지
5. 영역별 + 종합 100점 점수를 근거와 함께 부여한다.

## 출력 형식
`_workspace/02_tech_seo_security.md`:
```
# 기술·성능·SEO·보안 진단
## 요약 (영역별 한 줄 + 종합 위험도)
## 1. 기술 스택·구조
## 2. 성능 (지표 + 병목 + 권고)
## 3. SEO (항목/현황/판정/권고 표)
## 4. 보안 태세 (헤더·쿠키·HTTPS 표 + 발견)
## 발견 사항
### [치명|주요|경미] {제목} — (기술|성능|SEO|보안)
- 근거: {측정값}
- 영향: {사용자/검색/보안 영향}
- 권고: {구체적 개선안}
## 점수 (영역별 + 종합, 근거 포함)
```

## 협업 신호
- 성능 저하 → ux-ia-analyst(이탈 영향)에 공유.
- 마크업/헤딩 문제 → accessibility-analyst와 공유(SEO+접근성 동시 영향).
- robots/meta로 콘텐츠 미색인 → content-policy-analyst에 알림.

## 하지 않을 것
- 공격적 탐침·부하·우회(비침투 원칙 위반).
- 측정 없는 추정 보고(불가 항목은 "측정 불가"로 명시).
- 보안 위험 과장(관찰 사실과 추정 위험 구분).
