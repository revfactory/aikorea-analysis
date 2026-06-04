# 영문판 (English / NAIS)

- 출처 URL(메인): https://www.aikorea.go.kr/eng/web/main.do
- 수집 시각: 2026-06-05 05:10 (KST)
- 수집 도구: Chrome 브라우저(navigate + wait + javascript_tool + get_page_text)

## 영문판 메뉴 구조 (한글판 대비 축소)

| 영문 메뉴 | URL |
|----------|-----|
| About Us | `/eng/web/content.do?menu_cd=000017` |
| Leadership | `/eng/web/content.do?menu_cd=000019` |
| Authority and roles | `/eng/web/content.do?menu_cd=000020` |
| Organizational Chart | `/eng/web/content.do?menu_cd=000018` |
| Photo News | `/eng/web/board/garList.do?menu_cd=000012` |
| Korea AI Action Plan (Policy) | `/eng/web/content.do?menu_cd=000021` |

> 한글판 17개 메뉴 대비 영문판은 6개로 축소. 정책자료/보도자료/인터뷰/동영상/CAIO/운영위/분과위 등 다수 메뉴가 영문판에 없음.

## 영문 메인 (`/eng/web/main.do`)

- `<title>`: **NAIS** (한글판 title은 "국가인공지능전략위원회")
- `<html lang>`: **`ko`** ← **영문 페이지인데 lang이 한국어로 설정됨(불일치). 접근성·SEO 결함.**
- 헤딩:
  ```
  H1: KAIAC          ← (헤더, 중복1)  ※ 영문 약어 표기 'KAIAC' (사이트 영문명 NAIS와 상이)
  H2: PCNAIS Overview
  H2: News
  H2: Policy
  H1: KAIAC          ← (중복2)
  H1: (빈 값)        ← 빈 H1
  H2: PCNAIS Overview / News / Policy (반복)
  H1: Republic of Korea: A New Era of Prosperity
  H3: — Toward a Top 3 Global AI Powerhouse
  ```
  - 한글판과 동일한 빈 H1·H1 중복 패턴.
  - 영문 약어 혼재: title "NAIS", 헤딩 "KAIAC", 푸터 "PCNAIS"/"National Artificial Intelligence Strategy Committee" — 영문 명칭 비일관.
- 미번역 잔재: "본문 바로가기 / 주메뉴 바로가기"(스킵 네비)가 영문판에서 한글로 표시. "Top으로 이동"(스크롤 버튼)도 한글 잔존.

### 영문 메인 본문 (원본)
```
Republic of Korea: A New Era of Prosperity
— Toward a Top 3 Global AI Powerhouse
Launch Ceremony of the Presidential Council on National Artificial Intelligence Strategy (September 8, 2025)
[Footer] 16th–17th Floors, Seoul Square, 416 Hangang-daero, Jung-gu, Seoul, Republic of Korea
General Inquiries : aikorea@korea.kr
COPYRIGHT ⓒ National Artificial Intelligence Strategy Committee. All rights reserved.
```

## 영문 정책 페이지 — Korea AI Action Plan (`/eng/web/content.do?menu_cd=000021`)

- `<title>`: NAIS / source element: `<article>`
- 본문(원본 발췌):
  ```
  01  Building an AI Innovation Ecosystem
      - Build the AI Highway (GPUs, data, etc.)
      - Secure next-generation AI technologies
      - Cultivate core AI talent
      - Develop home-grown AI models
      - Advance AI regulatory innovation
  ```
  - (이하 02~ 항목 존재 추정. 본 정찰에서 01 섹션까지 수집.)

## 미수집 (영문판)

- About Us / Leadership / Authority and roles / Organizational Chart / Photo News 본문은 메뉴 구조만 확인, 본문 텍스트 미수집(시간/범위). 후속 분석가 필요 시 추가 수집 가능.

## 분석가 참고

- 접근성: `lang="ko"`로 고정된 영문 페이지 = 스크린리더 언어 오인식. KWCAG 6.4.2(언어 표시) 관련.
- 콘텐츠/일관성: 영문 명칭 3종(NAIS/KAIAC/PCNAIS) 혼재. 미번역 UI 텍스트.
- IA: 한영 메뉴 비대칭(영문 사용자 정보 접근성 제한).
