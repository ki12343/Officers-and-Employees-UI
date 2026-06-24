# Executive, Workforce & Compensation Disclosure UI

KR DART 사업·분기보고서의 **임원·직원 및 보상 현황**과 US SEC **DEF 14A** 임원·이사·보상 데이터를
**하나의 공통 UI/스키마**로 보여 주는 정적 산출물입니다. (앱 내 「핸드오프 문서」 탭 포함)

- KR/US가 다른 서비스처럼 보이지 않도록 동일 섹션 구조·동일 컴포넌트·동일 라벨을 사용합니다.
- 원문에 있는 데이터는 실제 값으로 채우고, 원문에 없는 값은 생성하지 않습니다.
- 사용자 화면에는 내부 작업 문구를 노출하지 않습니다(미해결 데이터 범위 이슈는 본 문서에만 기록).

## 실행 / 배포 (다중 파일 — 폴더 전체 배포)

`release/` **폴더 전체**를 업로드해야 합니다. `index.html` 단독 배포 또는 단일 인라인 HTML로 되돌리지 마세요.

```text
release/
  index.html          ← 진입 HTML
  support.js          ← DC 런타임
  kr-ext-data.js      ← KR 원문 추출 11개사 데이터 (window.KR_EXT)
  kr-officers-data.js ← KR 임원현황 원문 표 (현대차/차바이오텍/삼천당, window.KR_OFFICERS)
  us-sct-data.js      ← US Summary Compensation Table 13개사 (window.US_SCT)
  us-pvp-sct-data.js  ← US PvP 중심 6개사 SCT (window.US_PVP_SCT — Apple/MSFT/Alphabet/NVIDIA/JPM/Walmart)
  README.md
  HANDOFF.md
```

- **GitHub Pages**: 폴더 업로드 후 Settings > Pages에서 branch/경로 지정.
- **Vercel / Netlify**: `release/` 폴더 업로드, Framework=Static/Other, Build Command 비움, Output=`.`.
- 스크립트 로드 순서는 `support.js` → `kr-ext-data.js` → `kr-officers-data.js` → `us-sct-data.js` → `us-pvp-sct-data.js`이며 `index.html` `<head>`에 포함되어 있습니다. (`us-sct-data.js`와 `us-pvp-sct-data.js`는 서로 다른 글로벌(window.US_SCT / window.US_PVP_SCT)을 써 충돌 없음.) **us-pvp-sct-data.js 누락 시 PvP 6개사의 개인별 보수 상세·원문 SCT 모달 데이터가 빠집니다.**

> ⚠ **단일 인라인 HTML 금지**: 모든 데이터를 하나의 HTML에 인라인하면 ~22MB로 비대해져 런타임 스트리밍 렌더가 끝까지 해소되지 못하고, 원문 표 모달·일부 바인딩이 빈 상태로 남을 수 있습니다. 반드시 위 7개 파일을 함께 배포하세요.

## 포함 데이터

### KR — DART 사업·분기보고서 (실제 21개사)
에스앤디 · 비보존제약 · 프리시젼바이오 · 한텍 · 현대자동차 · 수성웹툰 · 코스모화학 · 차바이오텍 · 신한알파리츠 · 삼천당제약 · 동성제약 · 디와이디 · 세토피아 · 쌍방울 · 아이빔테크놀로지 · 아이큐어 · 코오롱글로텍 · 탑선 · 트리니티항공 · 티케이지휴켐스 · 파멥신.
초기 진입 시 KR 첫 문서가 자동 선택됩니다. (드롭다운 옵션이 22로 보이면 "선택 안됨" placeholder 1개 포함 — 실제 회사는 21개사.)

**KR 대량 임원 3개사 원문 임원현황 표(sourceTables.officers) 보강 완료** — DART 구조화 원문에서 실제 추출:
- 현대자동차: **168 rows** (등기 12 + 미등기 156). KPI 임원 수 **481명 유지**(총원, 텍스트 명시). 기본 화면은 대표/요약 5명 표시, 원문 표 버튼/모달은 **168 rows 기준**. → KPI 총원과 원문 임원현황 표 row 수는 다를 수 있음(표에 개별 등재된 인원 기준).
- 차바이오텍: **46 rows**.
- 삼천당제약: **22 rows**.

### US — SEC DEF 14A (실제 23개사)

**(A) DEF 14A 텍스트 원문 13개사 — SCT/NEO 반영 완료**
Texas Instruments · Micron · Qualcomm · UnitedHealth · Deere · Broadcom · Cisco · Boeing · Salesforce · AMD · Caterpillar · 3M · General Electric.
- 최신 회계연도 Summary Compensation Table 기반 **개인별 보수 상세**(급여·상여·주식·옵션·비주식인센티브·기타·총보수) + **NEO 임원 현황** 표시.
- `sourceTables.summaryCompensation` 원문 표 모달 연결(버튼 count = 모달 row count).
- CEO total = Pay Ratio total 검산 notes 유지. UnitedHealth는 SCT CEO total과 Pay Ratio CEO total 약 $17K 차이 — 원문값 보존 + 차이 notes 유지.
- **US SCT 과거연도(다년치) rows**: 검산 전수 통과 3개사(**Micron·3M·Boeing**)는 원문 다년치 행을 원문 표 모달에 표시(기본 화면 개인별 보수 상세는 최신연도 유지). 나머지 10개사는 회사별 표 형식 이질성으로 검산/귀속이 불확실해 미반영(최신연도 유지) — 별도 라운드 대상. 현재 사용자 화면에는 내부 작업 문구를 노출하지 않습니다.

**(B) DEF 14A PvP 중심 10개사 — CEO/PEO 보상-성과 정보 중심**
Apple · Microsoft · Alphabet · Amazon · Meta · NVIDIA · Tesla · JPMorgan · Johnson & Johnson · Walmart.
- **SCT 연결 완료 9개사**: Apple·Microsoft·Alphabet·NVIDIA·JPMorgan·Walmart·Meta·Johnson & Johnson·Amazon(SCT 수동 지정, Jassy 2025 total $2,069,861 검산 PASS; PvP 보조표 미사용). 행 검산 PASS분만 연결.연도 NEO, 원문 표 모달=다년치 전체 행(`us-pvp-sct-data.js`). 버튼 count=모달 row.
- **미연결 1개사**: Tesla(SCT 표 셀 분산으로 행 정렬 실패) — 정확도 우선으로 미주입, PvP/CEO Pay Ratio 중심 표시 유지(원문·파싱표는 존재, 연결 대기).
- SCT/이사회 전체 명단을 **억지로 주입하지 않습니다**. 미연결 항목은 `현재 확인할 수 없습니다` null-state, CEO/PEO 1명이 전체 임원/이사회처럼 보이지 않도록 안내. Pay Ratio/CAP/PvP를 SCT 개인별 보수로 둔갑시키지 않음.

## 원문 표 보기 모달

- 모달은 기본 화면 요약(summary) rows가 아니라 **sourceTables rows**를 표시합니다(둘은 분리).
- **버튼 count = 모달 row count = sourceTables rows** (KR 현대차 168 / 차바이오텍 46 / 삼천당 22, US 13개사 각 SCT rows).
- 모달 내부에서 가로/세로 스크롤하며, 페이지 전체 가로스크롤은 발생하지 않습니다. source table은 `word-break: keep-all` 계열로 짧은 단어가 세로로 쪼개지지 않게 처리합니다. 390/430 모바일에서도 닫기 버튼에 접근할 수 있습니다.

## 반응형

- 모바일: 임원 및 이사 현황은 기본 3명 표시 후 더보기/접기. 개인별 보수 상세는 요약카드를 유지하고 상세카드 3명 표시 후 더보기/접기.
- 원문 표 모달은 내부 가로/세로 스크롤, 페이지 전체 가로스크롤 0.
- 개인별 보수 구성 막대 차트는 모바일에서 **카드 내부 가로 스크롤**(막대 고정폭 + `overflow-x:auto`)로 표시되어 막대/이름이 압축되지 않으며, 페이지 전체 가로스크롤은 발생하지 않습니다.

## null-state

사용자 화면 null-state 제목은 2종으로 통일합니다: **`해당사항 없음`** / **`현재 확인할 수 없습니다`**.
