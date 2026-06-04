---
name: site-analysis-orchestrator
description: "웹사이트 종합 진단 에이전트 팀을 조율하는 오케스트레이터. aikorea.go.kr(국가인공지능전략위원회) 등 웹사이트의 콘텐츠·정책, UX·정보구조, 웹 접근성(KWCAG), 기술·성능·SEO·보안을 4개 전문 에이전트로 병렬 분석하고 종합 진단 리포트를 만든다. '사이트 분석', 'aikorea 분석', '웹사이트 진단', '사이트 종합 분석', '접근성/UX/SEO/보안 점검', '사이트 평가 리포트' 요청 시 사용. 후속 작업: 분석 다시 실행, 재분석, 결과 업데이트·수정·보완, 특정 영역(접근성/UX/콘텐츠/기술)만 다시, 이전 결과 기반 개선, 리포트 갱신 요청 시에도 반드시 이 스킬을 사용."
---

# Site Analysis Orchestrator — 웹사이트 종합 진단 오케스트레이터

웹사이트를 4개 전문 영역으로 병렬 분석하고 검증하여 **종합 진단 리포트**를 생성하는 통합 스킬. 기본 대상은 `https://www.aikorea.go.kr`(대통령직속 국가인공지능전략위원회)이며, 다른 URL이 주어지면 그 사이트를 분석한다.

## 실행 모드: 하이브리드

| Phase | 모드 | 이유 |
|-------|------|------|
| Phase 2 (정찰·수집) | 서브 에이전트 | 단일 에이전트가 인벤토리를 1회 수집, 팀 통신 불필요 |
| Phase 3 (4영역 분석) | **에이전트 팀** | 분석가 간 교차 영역 이슈를 SendMessage로 실시간 공유 — 팬아웃/팬인 |
| Phase 4 (검증·종합) | 서브 에이전트 | 독립 검증자 1명이 발견을 증거와 대조 후 통합 |

## 에이전트 구성

| 단계 | 에이전트 | subagent_type | 스킬 | 출력 |
|------|---------|--------------|------|------|
| 정찰 | site-scout | site-scout | web-site-scout | `_workspace/01_inventory/` |
| 분석 | content-policy-analyst | content-policy-analyst | content-policy-audit | `_workspace/02_content_policy.md` |
| 분석 | ux-ia-analyst | ux-ia-analyst | ux-ia-audit | `_workspace/02_ux_ia.md` |
| 분석 | accessibility-analyst | accessibility-analyst | accessibility-audit | `_workspace/02_accessibility.md` |
| 분석 | tech-seo-security-analyst | tech-seo-security-analyst | tech-seo-security-audit | `_workspace/02_tech_seo_security.md` |
| 종합 | report-synthesizer | report-synthesizer | analysis-report-synthesis | 최종 리포트 + `_workspace/03_verification_log.md` |

> 모든 Agent/TeamCreate 호출에 `model: "opus"`를 명시한다.

## 워크플로우

### Phase 0: 컨텍스트 확인 (후속 작업 지원)
작업 디렉토리에서 기존 산출물을 확인하여 실행 모드를 결정한다:
1. `_workspace/` 존재 여부 확인.
2. 분기:
   - **`_workspace/` 미존재** → 초기 실행. Phase 1로.
   - **존재 + 사용자가 특정 영역만 수정/보완 요청**(예: "접근성만 다시") → **부분 재실행**. 해당 분석 에이전트만 재호출하고, 기존 인벤토리(`01_inventory/`)를 재사용한다. 이전 산출물 경로를 에이전트 프롬프트에 넣어 기존 결과를 읽고 개선하도록 지시한다. 이후 Phase 4(종합)만 다시 돌려 리포트를 갱신한다.
   - **존재 + 새 대상 URL/새 수집 요청** → **새 실행**. 기존 `_workspace/`를 `_workspace_{YYYYMMDD_HHMMSS}/`로 이동 후 Phase 1로.

### Phase 1: 준비
1. 대상 URL 확정(기본 `https://www.aikorea.go.kr`), 분석 범위 확인(기본 4영역 전부).
2. `_workspace/` 생성(초기/새 실행 시). 새 실행이면 기존 디렉토리를 타임스탬프 디렉토리로 이동한 직후 생성.
3. 분석 메타(대상·일시·범위)를 `_workspace/00_run_meta.md`에 기록.

### Phase 2: 정찰·수집
**실행 모드:** 서브 에이전트

`Agent` 도구로 site-scout를 단독 호출:
```
Agent(
  subagent_type: "site-scout",
  model: "opus",
  description: "사이트 인벤토리 구축",
  prompt: "web-site-scout 스킬을 사용해 {대상 URL}을 정찰하고 _workspace/01_inventory/ 에 인벤토리를 구축하라. 후속 4명의 분석가(콘텐츠/UX/접근성/기술)가 공유할 토대다. 완료 후 INVENTORY.md 경로와 수집 요약을 반환하라."
)
```
- 부분 재실행이고 인벤토리가 이미 있으면 이 Phase를 건너뛴다.
- 완료 후 `_workspace/01_inventory/INVENTORY.md`를 Read하여 수집 결과(페이지 수·샘플링·누락)를 확인한다.

### Phase 3: 4영역 병렬 분석
**실행 모드:** 에이전트 팀 (팬아웃/팬인)

1. 팀 생성:
   ```
   TeamCreate(team_name: "site-analysis-team",
     description: "{대상} 4영역 병렬 분석")
   ```
   그리고 4명을 Agent 도구로 스폰(각각 `team_name: "site-analysis-team"`, `model: "opus"`, 해당 `subagent_type`, 스킬 사용 지시 + `_workspace/01_inventory/` 입력 + 출력 경로 명시).

2. 작업 등록:
   ```
   TaskCreate 4건:
   - "콘텐츠·정책 분석" → content-policy-analyst → 02_content_policy.md
   - "UX·정보구조 분석" → ux-ia-analyst → 02_ux_ia.md
   - "웹 접근성 진단" → accessibility-analyst → 02_accessibility.md
   - "기술·성능·SEO·보안 진단" → tech-seo-security-analyst → 02_tech_seo_security.md
   ```

3. 팀원 자체 조율: 4명이 인벤토리를 공통 입력으로 병렬 분석하며, 교차 영역 발견을 SendMessage로 공유한다.
   - 콘텐츠↔UX: 가독성/위치 문제, 접근성↔UX: 키보드·포커스, 접근성↔기술: 마크업·ARIA, 기술↔UX: 성능·모바일, 기술↔콘텐츠: 색인.

4. 리더 모니터링: TaskGet으로 진행 확인. 팀원이 막히면 SendMessage로 개입. 4개 산출물이 모두 저장되면 다음 단계로.

5. 팀 정리: 모든 분석 완료 후 분석가들에게 shutdown_request → `TeamDelete`. (Phase 4는 서브 에이전트 모드이므로 팀을 반드시 정리한 뒤 진행.)

### Phase 4: 검증·종합
**실행 모드:** 서브 에이전트

팀 정리 후 `Agent` 도구로 report-synthesizer 단독 호출:
```
Agent(
  subagent_type: "report-synthesizer",
  model: "opus",
  description: "검증·종합 리포트 통합",
  prompt: "analysis-report-synthesis 스킬을 사용하라. _workspace/01_inventory/(증거)와 02_*.md 4개 보고서를 읽고, 각 발견을 인벤토리와 대조 검증(채택/강등/기각/검증필요)한 뒤 종합 진단 리포트를 {출력경로}에 작성하라. 검증 로그는 _workspace/03_verification_log.md 에 남겨라."
)
```
- 완료 후 최종 리포트 경로와 검증 요약(채택/강등/기각 건수, 종합 점수)을 확인한다.

### Phase 5: 정리·보고
1. `_workspace/`는 보존한다(사후 검증·감사 추적용).
2. 사용자에게 결과 요약: 종합 점수·등급, 영역별 점수, 시급 개선 3가지, 최종 리포트 경로.
3. 피드백을 요청한다(Phase 7 진화): "특정 영역을 더 깊이 보거나, 팀 구성을 바꾸고 싶은 점이 있나요?"

## 데이터 흐름
```
[리더] Phase 1: _workspace/ 준비
   │
   ▼ Phase 2 (서브)
site-scout ──→ _workspace/01_inventory/  (공통 토대)
   │
   ▼ Phase 3 (팀)  ── 모두 01_inventory/ 를 Read ──
content-policy ⇄ ux-ia ⇄ accessibility ⇄ tech-seo-security  (SendMessage 교차공유)
   │   │            │              │
   ▼   ▼            ▼              ▼
02_content_policy / 02_ux_ia / 02_accessibility / 02_tech_seo_security
   │
   ▼ Phase 4 (서브) ── 01_inventory/ 로 발견 검증 ──
report-synthesizer ──→ 최종 종합 진단 리포트 + 03_verification_log.md
```

## 에러 핸들링
| 상황 | 전략 |
|------|------|
| site-scout 수집 실패(사이트 접근 불가) | 원인 보고, 사용자에게 재시도/대체 확인. 인벤토리 없으면 분석 불가하므로 중단·보고 |
| 인벤토리 부분 누락 | 분석가가 직접 보완 수집하도록 허용, 보완 출처 명시 |
| 분석가 1명 실패/중지 | 리더가 유휴 알림 감지 → SendMessage 상태 확인 → 1회 재시작. 재실패 시 해당 영역 누락 명시하고 진행 |
| 분석가 과반 실패 | 사용자에게 알리고 진행 여부 확인 |
| 분석가 간 발견 충돌 | 삭제하지 않고 출처 병기, synthesizer가 검증 단계에서 처리 |
| synthesizer가 근거 모순 발견 | 기각하지 말고 리포트 6번 섹션에 "검증필요"로 기록 |
| 팀→서브 전환 누락 | Phase 4 전 반드시 TeamDelete 확인(세션당 1팀만 활성) |

## 테스트 시나리오

### 정상 흐름
1. 사용자: "aikorea.go.kr 사이트 분석해줘"
2. Phase 0: `_workspace/` 없음 → 초기 실행.
3. Phase 1: 대상 확정, `_workspace/` 생성.
4. Phase 2: site-scout가 인벤토리 구축 → `01_inventory/INVENTORY.md` 확인.
5. Phase 3: 4명 팀 분석, 교차 공유, 4개 `02_*.md` 생성.
6. Phase 4: synthesizer가 검증·통합 → 최종 리포트 생성.
7. Phase 5: 요약 보고 + 피드백 요청.
8. 예상 결과: `aikorea_종합진단리포트_{YYYYMMDD}.md` 생성.

### 에러 흐름
1. Phase 3에서 accessibility-analyst가 브라우저 키보드 테스트 불가로 다수 항목 "확인필요".
2. 리더가 유휴 알림 수신, 상태 확인.
3. 분석가가 가능한 정적 점검은 완료하고 한계를 명시 → 정상 산출물로 인정.
4. Phase 4에서 synthesizer가 "수동 확인 필요" 항목을 리포트 6번 섹션에 집약.
5. 최종 리포트에 "접근성 일부 항목 수동 검증 권고" 명시.

### 부분 재실행 흐름
1. 사용자: "접근성 부분만 다시 분석해줘"
2. Phase 0: `_workspace/` 존재 + 특정 영역 → 부분 재실행.
3. 인벤토리 재사용, accessibility-analyst만 재호출(이전 `02_accessibility.md` 읽고 개선).
4. Phase 4 재실행하여 리포트의 접근성 섹션·종합 점수·로드맵 갱신.
