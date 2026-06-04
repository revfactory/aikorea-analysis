# 사이트맵 — 국가인공지능전략위원회 (aikorea.go.kr)

- 대상: https://www.aikorea.go.kr
- 수집 일시: 2026-06-05 05:00~05:12 (KST)
- 수집 도구: WebFetch + Chrome 브라우저(MCP) + headless Chrome(스크린샷) + curl(헤더/성능)
- 출처 기준: GNB 메뉴(`#gnb`) DOM 추출 + `/web/sitemap.do?menu_cd=000016` 페이지 교차 확인

## 사이트 구조 개요

- 플랫폼: nginx + 서버사이드 렌더링 + AJAX 하이브리드(GIOINFRA 구축 CMS)
- 페이지 라우팅: `menu_cd` 쿼리 파라미터 기반. `content.do`(정적 콘텐츠), `brdList.do`(일반 게시판), `garList.do`(갤러리형 게시판), `brdDetail.do`(게시글 상세), `search.do`(통합검색), `sitemap.do`(사이트맵)
- 한글판(`/web/`) + 영문판(`/eng/web/`) 2개 언어. 영문판은 축소 구조.

## 한글판 페이지 목록 (전수)

| # | 메뉴 경로 | 제목 | URL | 유형 | 응답 | 게시물수 | 수집 |
|---|----------|------|-----|------|------|--------|------|
| 1 | (홈) | 메인 | `/web/main.do` | 메인 | 200 | - | 본문+HTML+스크린샷(D/M)+네트워크 |
| 2 | 위원회 개요 > 위원회 소개 | 위원회 소개 | `/web/content.do?menu_cd=000005` | 콘텐츠 | 200 | - | 본문+HTML+스크린샷(D/M) |
| 3 | 위원회 개요 > 위원 소개 | 위원 소개 | `/web/content.do?menu_cd=000018` | 콘텐츠 | 200 | - | 본문+HTML+스크린샷(D) |
| 4 | 위원회 개요 > 위원회 및 지원단 조직 소개 | 조직 소개 | `/web/content.do?menu_cd=000024` | 콘텐츠 | 200 | - | 본문+HTML |
| 5 | 위원회 개요 > MI 소개 | MI 소개 | `/web/content.do?menu_cd=000031` | 콘텐츠 | 200 | - | 본문+HTML |
| 6 | 위원회 활동 > 전체 회의 | 전체 회의 | `/web/board/brdList.do?menu_cd=000008` | 게시판 | 200 | 2 | 본문+HTML+목록 |
| 7 | 위원회 활동 > 운영위원회 | 운영위원회 | `/web/board/brdList.do?menu_cd=000022` | 게시판 | 200 | 17 | 본문+HTML+목록 |
| 8 | 위원회 활동 > 분과위원회 | 분과위원회 | `/web/board/brdList.do?menu_cd=000009` | 게시판 | 200 | 281 | 본문+HTML+목록 |
| 9 | 위원회 활동 > CAIO 협의회 | CAIO 협의회 | `/web/board/brdList.do?menu_cd=000023` | 게시판 | 200 | 3 | 본문+HTML+목록 |
| 10 | 소통공간 > 공지사항 | 공지사항 | `/web/board/brdList.do?menu_cd=000010` | 게시판 | 200 | 15 | 본문+HTML+목록 |
| 11 | 소통공간 > 정책자료 | 정책자료 | `/web/board/brdList.do?menu_cd=000011` | 게시판 | 200 | 11 | 본문+HTML+목록+상세샘플+스크린샷(D) |
| 12 | 소통공간 > 보도자료 | 보도자료 | `/web/board/brdList.do?menu_cd=000012` | 게시판 | 200 | 74 | 본문+HTML+목록 |
| 13 | 소통공간 > 카드뉴스 | 카드뉴스 | `/web/board/garList.do?menu_cd=000037` | 갤러리 | 200 | 6 | 본문+목록 |
| 14 | 소통공간 > 인터뷰 및 기고 | 인터뷰 및 기고 | `/web/board/brdList.do?menu_cd=000014` | 게시판 | 200 | 34 | 본문+HTML+목록 |
| 15 | 소통공간 > 사진 자료 | 사진 자료 | `/web/board/garList.do?menu_cd=000013` | 갤러리 | 200 | 99 | 본문+목록 |
| 16 | 소통공간 > 동영상 자료 | 동영상 자료 | `/web/board/garList.do?menu_cd=000038` | 갤러리 | 200 | 23 | 본문+목록 |
| 17 | 통합검색 | 통합검색 | `/web/search/search.do?menu_cd=000015` | 검색 | 200 | - | 폼 구조 |
| 18 | 사이트맵 | 사이트맵 | `/web/sitemap.do?menu_cd=000016` | 사이트맵 | 200 | - | 링크 트리 |
| S | (게시글 상세 샘플) | 대한민국 인공지능 행동계획(한글, 인쇄본) | `/web/board/brdDetail.do?menu_cd=000011&num=523` | 상세 | 200 | - | 본문+HTML 구조 |

## 영문판 페이지 목록 (전수)

| # | 메뉴 | 제목 | URL | 응답 | 수집 |
|---|------|------|-----|------|------|
| E1 | (홈) | NAIS Main | `/eng/web/main.do` | 200 | 본문+HTML |
| E2 | About Us | About Us | `/eng/web/content.do?menu_cd=000017` | 200 | (목록만, 본문 미수집) |
| E3 | Leadership | Leadership | `/eng/web/content.do?menu_cd=000019` | 200 | (목록만, 본문 미수집) |
| E4 | Authority and roles | Authority and roles | `/eng/web/content.do?menu_cd=000020` | 200 | (목록만, 본문 미수집) |
| E5 | Organizational Chart | Organizational Chart | `/eng/web/content.do?menu_cd=000018` | 200 | (목록만, 본문 미수집) |
| E6 | Photo News | Photo News | `/eng/web/board/garList.do?menu_cd=000012` | 200 | (목록만, 본문 미수집) |
| E7 | Policy > Korea AI Action Plan | Korea AI Action Plan | `/eng/web/content.do?menu_cd=000021` | 200 | 본문 일부 수집 |

> 영문판 메뉴 코드는 한글판과 다름 주의(예: 영문 Organizational Chart=000018, 한글 위원 소개=000018).

## 외부 링크 (따라가지 않음, 기록만)

| 대상 | URL | 위치 |
|------|-----|------|
| YouTube 채널 (@PCNAIS) | https://www.youtube.com/@PCNAIS | 헤더/푸터 |
| X(트위터) 채널 (@AIkorea00) | https://x.com/AIkorea00 | 헤더/푸터 |
| 대표이메일 | mailto:aikorea@korea.kr | 푸터 |

## 시스템/인프라 URL (수집하지 않음)

| 대상 | URL | 비고 |
|------|-----|------|
| 404 에러 페이지 | http://errdoc.gabia.io/404.html | 외부(가비아) 호스팅. 존재하지 않는 `.do` 접근 시 302 리다이렉트 |
| AJAX 엔드포인트 | `/web/header/menuMng/ajax/menuList.do` (POST) | 메뉴 동적 로드 |
| AJAX 엔드포인트 | `/web/main/ajax/mainBbsList.do` (POST) | 메인 게시판 목록 동적 로드(6회 호출) |

## 샘플링 정책

- **콘텐츠/구조 페이지(content.do, search.do, sitemap.do, main.do)**: 한글판 전수 수집(본문+HTML 구조).
- **게시판 목록 페이지(brdList/garList) 11종**: 전수 방문, 목록 1페이지(최신 글)와 총 게시물 수 기록.
- **게시글 상세(brdDetail/garView)**: 총 약 565건. 전수 수집하지 않고 **대표 1건(정책자료 핵심문서 num=523)만 구조 샘플링.** 사유: 깊이보다 넓이 우선, 정부 사이트 부하 자제. 개별 글 본문은 후속 분석가가 필요 시 특정 글을 추가 요청하면 수집.
- **영문판**: 메인+정책 페이지 본문 수집, 나머지 4개 콘텐츠 페이지는 메뉴 구조만 확인(본문 미수집 — INVENTORY.md 미수집 항목 참조).
