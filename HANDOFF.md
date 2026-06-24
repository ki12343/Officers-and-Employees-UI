# 임원·직원 및 보상 현황 UI — 핸드오프

KR DART 사업·분기보고서의 `VIII. 임원 및 직원 등에 관한 사항`과 US SEC `DEF 14A` 임원·이사·보상 데이터를
**하나의 공통 스키마/렌더러**로 보여 주는 정적 산출물의 개발자 문서입니다.

> **검증 범위**: KR DART 21개사 · US DEF 14A SCT/NEO 13개사 · US PvP 중심 10개사 = 총 44개사/문서. KR 원문 표 모달(officers sourceTables): 현대자동차 168 · 차바이오텍 46 · 삼천당제약 22 rows. US SCT 다년치 allRows: Micron 15 · 3M 14 · Boeing 8 rows. (앱 「핸드오프 문서」 탭에 동일 요약 카드 수록)

## 1. 배포 구조 (다중 파일)

`release/` 폴더 전체를 배포합니다. **`index.html` 단독 배포 / 단일 인라인 HTML 회귀 금지.**

```text
release/
  index.html          ← 진입 HTML (DC 템플릿 + 로직)
  support.js          ← DC 런타임
  kr-ext-data.js      ← window.KR_EXT  (KR 원문 추출 11개사)
  kr-officers-data.js ← window.KR_OFFICERS (현대차/차바이오텍/삼천당 임원현황 원문 표)
  us-sct-data.js      ← window.US_SCT  (US SCT 13개사)
  us-pvp-sct-data.js  ← window.US_PVP_SCT  (US PvP 6개사 SCT — Apple/MSFT/Alphabet/NVIDIA/JPM/Walmart)
  README.md
  HANDOFF.md
```

로드 순서: `support.js` → `kr-ext-data.js` → `kr-officers-data.js` → `us-sct-data.js` → `us-pvp-sct-data.js` (모두 `index.html` `<head>`). window.US_SCT / window.US_PVP_SCT는 별개 글로벌로 충돌 없음. **총 8개 파일 구조** — us-pvp-sct-data.js 누락 시 PvP 6개사 SCT 미표시.
단일 인라인(~22MB) 시 런타임 스트리밍 렌더가 끝까지 해소되지 못해 원문 표 모달·일부 바인딩이 빈 상태로 남을 수 있어 금지합니다.

## 2. 공통 렌더러

각 회사 함수(`krSnd()`, `krHyundai()`, `krExtBuild(key)`, `usBuild()` …)가 동일한 공통 스키마 객체를 반환하고,
단일 `renderVals()`가 KR/US 구분 없이 같은 템플릿으로 렌더링합니다. 회사 추가는 같은 스키마를 반환하는 함수/데이터 한 건만 추가하면 되고 렌더러·템플릿은 불변입니다.

공통 섹션 순서: 문서 요약 → AI Brief → 핵심 KPI → 임원 및 이사 현황 → 직원 등 현황 → 임원 보수 요약 → 이사 보수 → 개인별 보수 상세 → 주식보상/주식매수선택권 → 보상 비율·성과 지표 → 주석·산정 기준 → 원문 표 보기 모달. KR/US 동일.

## 3. 데이터 규칙 (원문 보존)

- **단시간 근로자**는 정규/기간제 내부 부분집합 — 합계 중복 합산 금지.
- **미등기임원 수**: 임원현황 표 기준 vs 보수현황 기준 분리(혼용 금지), 불일치 시 주석.
- **기타비상무이사**: 등기 총수 포함, 사내/사외/감사 합산 제외(억지 합산 금지).
- **출생년월·재직기간·직위·주식수** 등 숫자성 텍스트는 원문 문자열 보존(추정 금지).
- **1인평균급여**: 총액÷인원과 불일치 시 자동 보정 금지, 계산 기준값 표시 + 주석(`averageCompensationStatus`). 예: 아이큐어(원문 73,022천원 vs ≈10백만원).
- **개인별 5억원 이상 보수** 대상자 없음 → `해당사항 없음`. 원문 표 미연결/확인 불가 → `현재 확인할 수 없습니다`.
- **맥락 플래그**(`contextFlags`: 회생절차·이사 다수 사임·전직 임원 퇴직보수·퇴직소득 포함·정정공시)는 오류 단정 금지, 원문 보존. 예: 동성제약·세토피아·트리니티항공.
- **금액 단위**: 원/천원/백만원 혼재 → 백만원 통일, 환산 기준은 notes 보존.

## 4. KR 임원현황 원문 표 보강 (현대차/차바이오텍/삼천당)

DART 구조화 원문(`sections > 가. 임원 현황`)에서 등기·미등기 임원 표를 추출해 `kr-officers-data.js`(`window.KR_OFFICERS`)에 보존, 각 데이터셋의 `sourceTables.officers`로 연결.

- 현대자동차: **168 rows**(등기 12 + 미등기 156). 13개 컬럼(구분/성명/성별/출생년월/직위/등기임원 여부/상근 여부/담당업무/주요경력/소유주식수/재직기간/임기만료일/비고), rowspan(구분/비고) carry-forward 반영, 원문 문자열 보존.
- 차바이오텍: **46 rows** · 삼천당제약: **22 rows**.
- **KPI 총원 vs 원문 표 row 수 기준 차이**: 현대차 KPI 임원 수 481명(텍스트 명시 총원)은 유지하되, 원문 임원현황 표에 개별 등재된 인원은 168명. 모달 상단 스코프 주석으로 "전체 481명 중 원문에서 개별 확인된 168명" 명시.
- 기본 화면 summary rows(대표/요약)와 원문 표 모달 sourceTables rows는 분리. 버튼 count = 모달 row count = sourceTables rows.

## 5. US 데이터

### (A) DEF 14A 텍스트 원문 13개사 — SCT/NEO 완료
Texas Instruments · Micron · Qualcomm · UnitedHealth · Deere · Broadcom · Cisco · Boeing · Salesforce · AMD · Caterpillar · 3M · General Electric.
- 원본 DEF 14A의 Summary Compensation Table을 HTML `<table>` 파싱(colspan/rowspan·name carry-forward·footnote marker는 숫자 파싱 시에만 제거)하여 `us-sct-data.js`(`window.US_SCT`)에 보존, `individualCompensation`(개인별 보수 상세)·`officers`(NEO 임원 현황)·`sourceTables.summaryCompensation`(원문 표 모달)로 매핑.
- 컬럼: 성명/직위/회계연도/급여/상여/주식보상/옵션보상/비주식 인센티브/기타/총보수. `totalCompensation`을 Pay Ratio CEO total로 대체하지 않음(SCT 원문값 사용).
- **검산**: CEO 최신연도 SCT total = Pay Ratio CEO total 일치 확인. UnitedHealth는 약 $17K 차이 — 원문값 보존 + 차이 notes.
- **과거연도(다년치) rows 확장 — 검산 통과 3개사 반영**: Micron·3M·Boeing은 SCT 표가 정렬되어(컬럼 매핑 확정·이름 carry-forward 정상) **모든 NEO 행의 구성요소 합계 = 총보수 검산 PASS, CEO 최신연도 total 일치**를 만족 → 원문 다년치 행(Micron 15행 2021/20/19, 3M 14행 2022/21/20, Boeing 8행 2022/21)을 `us-sct-data.js`의 `allRows`로 보존하고 **원문 표 모달(sourceTables.summaryCompensation)에만** 표시(버튼/모달 row count = allRows). **기본 화면 개인별 보수 상세는 최신연도 NEO만 유지**(`sct.rows`).
- **미반영 10개사**: Texas Instruments(CEO total ~$25K 차이=각주/조정 컬럼), Caterpillar/GE/AMD/Qualcomm/UnitedHealth/Deere/Salesforce/Cisco(rowspan 이름 셀·zero-width 분산 셀·컬럼 미정렬 → row 단위 검산/귀속 불확실), Broadcom(CEO PASS이나 NEO 이름 carry-forward 불완전). **정확도 우선 원칙으로 미주입**, 최신연도 rows 유지. 회사별 전용 파서(이름/직위 splitNameRole·컬럼 위치 매핑·각주 조정 처리)가 가능한 별도 라운드 대상. 사용자 화면에는 내부 작업 문구 미노출.

### (B) DEF 14A PvP 중심 10개사
Apple · Microsoft · Alphabet · Amazon · Meta · NVIDIA · Tesla · JPMorgan · Johnson & Johnson · Walmart. (DEF 14A 원문·파싱표 `def14a_samples/<TICKER>/parsed`)
- **SCT 연결 완료(PASS-CONNECTED) 9개사**: Apple·Microsoft·Alphabet·NVIDIA·JPMorgan·Walmart·Meta·**Johnson & Johnson**·**Amazon** — DEF 14A 파싱표(`tables_md.json`)에서 SCT를 추출, **전 행 구성요소 합계=총보수 검산 PASS**. `us-pvp-sct-data.js`(`window.US_PVP_SCT`)로 분리, usBuild의 `sct` 폴백으로 연결. 기본 화면 개인별 보수 상세 = 최신 회계연도 NEO(예: Apple 2025 5명, Amazon 2025 6명), 원문 표 모달 = 다년치 allRows(Apple 14·MSFT 13·Alphabet 14·NVIDIA 15·JPM 15·Walmart 13·Amazon 17). 버튼 count=모달 row=allRows.
- **Amazon**(table_id 60 수동 지정): CEO/PEO Andrew R. Jassy 2025 SCT total $2,069,861. Amazon SCT는 Salary/Stock Awards/All Other/Total 4컬럼 구조(Bonus/Option/Non-Equity/Pension 없음 → null). 전 행 검산 PASS(Garman·Herrington 2024 행만 $1 반올림 차이). PvP 보조표(table_id 63/64)는 SCT로 오선택하지 않음.
- **미연결(FOUND-NOT-CONNECTED) 1개사**: Tesla(파싱표에서 SCT 표 식별/행 정렬 실패; Musk 보수구조 특이). 정확도 우선 원칙으로 미주입, 기존 PvP/CEO Pay Ratio 중심 표시 유지. 원문·파싱표는 존재하므로 "원문 없음"이 아니라 "UI 연결 대기".
- Director Compensation/officers 전체 명단은 이번 라운드 미연결(파싱표 식별·검산 추가 필요). SCT/이사회 전체 명단을 억지로 채우지 않음.
- 엣지케이스 원문 보존: Tesla(Musk 연간 SCT 사실상 $0 — "—" + 맥락 주석), Amazon(낮은 SCT 보정 없음), JNJ/NVDA(PvP 연차/연도 라벨 원문대로). Pay Ratio/CAP/PvP 값을 SCT 개인별 보수로 둔갑시키지 않음.

## 6. null-state / 반응형

- null-state 제목 2종: `해당사항 없음`(문서·제도상 미적용/미제공) / `현재 확인할 수 없습니다`(연결 원문 범위상 확인 불가). 섹션 전체·동일 레벨 카드 전체 빈 경우 통합 null-state, 일부만 있으면 카드 레이아웃 유지, 보상 비율·성과 지표 둘 다 없을 때 compact inline, KPI는 compact(`—`).
- 모바일: 임원 및 이사 현황 3명 + 더보기/접기, 개인별 보수 상세 요약카드 유지 + 상세 3명 + 더보기/접기. 원문 표 모달 내부 가로/세로 스크롤, 페이지 가로스크롤 0.

## 7. 이번 라운드 적용 / 남은 선택 개선 항목

- **[적용 완료] 모바일 개인별 보수 구성 막대 차트 내부 가로 스크롤**: 모바일(≤767px)에서 막대는 `min-width:88px; flex:0 0 auto`로 고정되어 카드 내부 `overflow-x:auto`로 좌우 스크롤되며(`overscroll-behavior-x:contain`), 막대 수가 많은 US 회사(예: Cisco 6 NEO)에서도 페이지 전체 가로스크롤 없이 카드 내부에서만 스크롤됩니다. 모바일 안내문 "↔ 좌우로 스크롤해 전체 보수 구성을 확인할 수 있습니다." 표시. 데스크톱 레이아웃은 불변.
- **[부분 반영 — 검산 통과 3개사] US SCT 과거연도(다년치) rows**: Micron·3M·Boeing은 row 단위 검산(구성요소 합계=총보수) 전수 PASS + CEO total 일치로 원문 다년치 행을 `allRows`에 반영, 원문 표 모달에만 표시(기본 화면은 최신연도 유지). 나머지 10개사는 표 형식 이질성으로 검산/귀속 불확실 → 미주입(전용 파서 별도 라운드).

## QA(요약)

KR 21개사 / US 23개사 드롭다운 정상(placeholder 제외). 원문 표 모달 버튼 count = 모달 row count = sourceTables rows(현대차 168 / 차바이오텍 46 / 삼천당 22 / US 각 SCT rows). 사용자 화면 내부 작업 문구(추가 추출 대상·원문 미기재·XBRL·파싱 필요·undefined·NaN 등) 0건. 콘솔 404/script error 없음. 다중 파일 release 정상.
