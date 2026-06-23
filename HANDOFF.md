# 임원·직원 및 보상 현황 UI — 핸드오프 (임시 배포본)

## 현재 구조
- **단일 자립형 HTML**(`index.html`) — 브라우저에서 바로 열리며 별도 빌드가 없습니다.
- 회사별 데이터는 HTML 내부 로직에 인라인되어 있고, **하나의 렌더러가 모든 회사·시장(KR/US) 데이터셋을 동일 IA로 소비**합니다.

## 단일 렌더러 방식
각 회사 함수(`krSnd()`, `krBibojon()`, … `US()`)가 동일한 공통 스키마 객체를 반환하고,
단일 `renderVals()`가 그 객체를 받아 KR·US 구분 없이 같은 템플릿으로 렌더링합니다.
회사를 추가하려면 같은 스키마를 반환하는 함수 하나만 추가하면 되고, 렌더러·템플릿은 수정하지 않습니다.

## 외부 JSON 분리를 이번에 하지 않은 이유
- **임시 배포 안정성 및 회귀 위험 최소화**가 목적입니다.
- 단일 HTML은 GitHub Pages/Vercel/Netlify에 그대로 올려 즉시 시연할 수 있고, 빌드 파이프라인 의존이 없습니다.
- 데이터 외부화는 다중 파일·번들 구조를 전제로 하므로 이번 임시 배포 범위에서는 제외했습니다.

## 향후 권장 구조 (운영 전환 시)
```text
/data
  /kr
    company-a.json
    company-b.json

/components
  ExecutiveSection.tsx
  EmployeeSection.tsx
  CompensationSection.tsx
  StockOptionSection.tsx

/render
  commonRenderer.ts
```
원칙: 소비 지점은 1곳 유지(현 `renderVals()`의 정규화 로직을 `commonRenderer`로 이식),
회사별 함수 → `*.json` 1:1 이전, 출처(sourceSection/table/page)는 row 단위로 확장.

## 현재 반영된 주요 데이터 규칙
- **단시간 근로자**는 정규직/기간제 내부 부분집합으로 처리 — 합계에 중복 합산 금지.
- **미제공 / 해당없음 / 제공** 상태 구분(`not_provided` / `not_applicable` / `provided`).
- **미등기임원 수**는 임원현황 표 기준과 보수현황 기준을 분리(혼용 금지) — 불일치 시 주석 노출.
- **기타비상무이사** `roleType: other_non_executive_director` — 등기 총수 포함, 사내/사외/감사 합산 제외.
- **출생년월·재직기간·직위** 등 숫자성 텍스트는 원문 그대로 보존(추정 금지).
- **1인평균급여** 이상치·산정기간 불명확 시 자동 연환산하지 않고 원문 보존 + `averageSalaryFlag: needs_review`.
- **개인별 보수**는 출처 구분(`individualCompensationSource`: director_auditor_over_500m / top_5_executives_over_500m / unregistered_executive_over_500m / other).
- **주식매수선택권**은 `fairValue / grantedQuantity / exercisedQuantity / cancelledQuantity / remainingQuantity`를 별도 필드로 보존.
- **REIT 등 특수 구조**는 `compensationContextFlag: reit_small_board_compensation`로 맥락 처리(이상치 아님).
- **섹션 탐색**은 번호/id가 아닌 `source_title`("임원 및 직원 등에 관한 사항" 등) 기준.
- **단위 변환** 시 원문 단위·환산 기준을 `notes`에 기록(천원→백만원 등).

## QA(요약)
초기 진입 시 KR 첫 문서 자동 선택·즉시 렌더링, KR/US 상호 배타 선택, 드롭다운 화살표 우측 여백 확보,
undefined/null/NaN/placeholder 미노출, 콘솔 오류 없음.
