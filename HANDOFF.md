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

## US 보상 데이터 (SEC DEF 14A / XBRL) — 이번 release 반영 기준
US 화면은 데모가 아니라 SEC DEF 14A의 **Pay-vs-Performance 실측 데이터(10개사: Apple·Microsoft·Alphabet·Amazon·Meta·NVIDIA·Tesla·JPMorgan·Johnson & Johnson·Walmart)**를 반영합니다.

- **표준 태그 기반**: US 보상 데이터는 SEC XBRL의 Pay-vs-Performance 표준 태그
  (`ecd:PeoTotalCompAmt` / `ecd:NonPeoNeoAvgTotalCompAmt` / `ecd:PeoActuallyPaidCompAmt`)에서 추출했습니다.
  반영 항목: CEO/PEO 이름 · CEO 총보수(SCT) · NEO 1인 평균 총보수 · CAP(Actually Paid Compensation) · 연도별 PvP 추이.
- **CEO Pay Ratio / 직원 중위보수**: PvP XBRL 블록에 포함되지 않아(태그 부재) **별도 텍스트 파싱 필요** — 임의 생성하지 않고 "추가 파싱 필요"로 표기.
- **NEO 로스터 / 이사회 명단 / 개인별 SCT 구성요소(급여·상여·주식·옵션)**: 회사별 표 구조가 달라 **추가 정형화 필요**.
  Outstanding Equity 수량·전체 NEO 로스터도 동일하게 추가 추출 대상.
- **CEO/PEO 중심 임원 표**: 현재 데이터 범위의 한계이며, 빈값처럼 보이지 않도록 안내 라벨을 유지합니다.
- **엣지케이스 — 원문 보존 원칙 적용**:
  - **JNJ / NVDA**: CEO 교체연도·회계연도 시프트로 PvP 연차가 4개이거나 연도 라벨이 다른 경우, 임의로 5개년에 맞추지 않고 원문/XBRL 기준 그대로 유지.
  - **Tesla(Elon Musk)**: 연간 SCT 총보수가 사실상 $0(2018년 일괄 옵션 맥락)이므로 "—" + 맥락 주석 유지(일반 연봉 구조와 다름).
  - **Amazon(Andrew Jassy)**: 낮은 SCT도 임의 보정 없이 XBRL 실측치 기준 유지.
- **외부 JSON 분리**: 이번 release에서는 진행하지 않고 **단일 자립형 HTML**을 유지합니다(회귀 위험 최소화).

## US 보상 데이터 — 원문(DEF 14A 텍스트) 파싱 13개사 추가
XBRL 10개사와 별개로, 첨부 DEF 14A 원문(HTML)에서 직접 파싱한 **CEO Pay Ratio 중심 13개사**를 US 드롭다운에 추가했습니다.
대상: Texas Instruments · Micron · Qualcomm · UnitedHealth · Deere · Broadcom · Cisco · Boeing · Salesforce · AMD · Caterpillar · 3M · General Electric.

- **추출 출처**: 대부분 PvP-rule(FY2022~) 이전 또는 XBRL ecd 블록 부재 필링이라, XBRL이 아닌 **Item 402(u) CEO Pay Ratio 공시 문장**에서 추출.
- **반영 항목(원문 실측)**: 회사명·CEO 성명·DEF 14A 제출일·회계연도 · CEO 총보수(SCT Total) · 직원 중위보수(median) · CEO Pay Ratio.
  - 예: Caterpillar(Umpleby) $24,298,032 / median $51,102 / 475:1, GE(Culp) $14,698,285 / median $73,038 / 201:1, Salesforce(Benioff) $29,868,893 / median $199,130 / 150:1(전 공동CEO Taylor 135:1).
- **이 13개사가 채우는 빈칸**: XBRL 10개사가 비워둔 **CEO Pay Ratio·직원 중위보수**를 실제로 표시(게이지 + KPI + 주석).
- **이 13개사의 미추출(추가 추출 대상)**: NEO 로스터·이사회 명단·개인별 SCT 구성요소·Outstanding Equity·Pay vs Performance(Item 402v) 표. UI에 "추가 추출 대상" 라벨로 명시.
- **CEO 성명은 원문에서 검증**(Templeton·Mehrotra·Amon·Witty·May·Tan·Robbins·Calhoun·Benioff·Su·Umpleby·Roman·Culp) — 임의 생성 아님.
- **엣지케이스**: Salesforce는 공동 CEO 구조(Benioff/Taylor) → 주 CEO(Benioff) 기준 표시 + 전 공동CEO 주석 보존. Boeing은 "approximately 154×" 서술형 → 154:1로 표기.
- **렌더러 영향**: Pay Ratio 게이지와 Pay-vs-Performance 차트를 **독립 조건(`ratioAvailable` / `pvpAvailable`)으로 분리**. 그 결과 XBRL 10개사의 PvP 차트도 정상 노출됩니다(이전에는 게이지 조건에 묶여 가려져 있었음 — 이번에 수정). 게이지 스케일 상한 300→500(고비율사 구분).

## CEO Pay Ratio 텍스트 파싱 규칙 (Item 402(u))
PvP XBRL(ecd) 블록이 없거나 PvP-rule(FY2022~) 이전 필링은 Pay-vs-Performance가 없는 것이 정상이며, 이때는 Pay Ratio 공시 문장에서 직접 추출한다.

추출 대상 문장 패턴:
```text
median ... was $X
median annual total compensation ... $X
CEO annual total compensation ... $X
ratio ... N:1
ratio ... N to 1
approximately N times        (서술형 → N:1 로 정규화)
```

필드 매핑: CEO 총보수=「CEO ... $X」/SCT Total · 직원 중위보수=「median employee ... $X」 · Pay Ratio=「ratio ... N:1 / N to 1 / approximately N times」.

엣지케이스(원문 보존):
- **Salesforce**: 공동 CEO 구조 → 주 CEO(Benioff 150:1) 기준 표시 + 전 공동CEO(Taylor 135:1) 주석 보존.
- **Boeing**: "약 154배(approximately 154 times)" 서술형 → 154:1로 정규화하되 원문 표현 주석 보존.
- **회계연도 라벨**: 회사별 원문 기준 보존(FY2021~FY2025 혼재) — 임의 통일 금지.

`pvpStatus` 상태값: `available`(XBRL 10) · `not_in_filing`(원문 13, 정상) · `requires_text_table_parsing`(향후 본문 표 정형화 대상).
UI는 PvP 부재 시 "해당 문서에서는 Pay-vs-Performance XBRL 블록이 확인되지 않았습니다. CEO Pay Ratio 공시를 우선 표시합니다." 문구 + 상태칩으로 표시(오류 아님).

**FYE 주의**: 원문 13개사 회계연도 종료일은 표준 회계 캘린더 기반으로 채워져 있으므로, 운영 전환 시 filing cover(원문 표지) 기준 재확인 필요.

## KR 원문 PDF 추출 11개사 (2025 사업연도)
DART 사업보고서 **PDF 원문**(브라우저 내 pdf-parse)에서 `VIII. 임원 및 직원 등에 관한 사항`을 파싱해 11개사를 추가했습니다.
대상: 동성제약·디와이디·세토피아·쌍방울·아이빔테크놀로지·아이큐어·코오롱글로텍·탑선·트리니티항공·티케이지휴켐스·파멥신.

- **데이터 분리**: 11개사 추출값은 `kr-ext-data.js`(`window.KR_EXT`)에 분리하고, 로직의 `krExtBuild(key)`가 공통 스키마로 변환합니다(렌더러·템플릿 불변). release 번들 시 helmet 스크립트로 인라인됩니다.
- **반영 항목**: 임원 명단(성명·직위·구분·상근) · 직원 수(정규/기간제)·1인 평균급여 · 이사·감사 전체/유형별 보수 · 주총 승인 한도 · 개인별 5억원 이상 보수 · 주식매수선택권 유무.
- **원문 표 참조(추가 정형화 대상)**: 임원 개인별 주요경력·보유주식·재직기간, 주식매수선택권 개인별 수량·행사가·공정가치, 개인별 SCT 세부 구성. UI에 "원문 표 참조" 라벨로 명시.
- **단위 정규화**: 원문 천원/백만원/원 혼재 → 백만원 환산. 환산 기준은 `notes`에 보존.
- **엣지케이스(원문 보존 원칙)**: 동성제약 회생절차 맥락 주석 · 세토피아 이사 다수 사임 · **아이큐어 1인평균급여 원문 표기(73,022천원)와 연간급여총액÷인원(≈10백만원) 불일치** → 후자 기준 표기 + 주석(원문 확인 필요) · 트리니티항공 등 전직 임원 5억원+ 퇴직보수 원문대로 보존.
- **FYE/문서**: 모두 기준일 2025.12.31 · 사업연도 2025. 11개 중 9개가 [정정]사업보고서 → `amendment.isAmended=true` + 정정공시 뱃지.
- **PDF 파일명 주의**: 한글·NFD·괄호 파일명은 샌드박스 경로 검증에서 거부되므로, 추출 시 ASCII 파일명(`r01.pdf`…)으로 재업로드해 처리했습니다.

## KR 원문 PDF 추출 — 자동화 리스크 규칙
- **금액 단위 자동판별**: 원/천원/백만원 혼재 → 백만원 통일 표시, 원문 단위·환산식 보존. 천원÷1,000 · 원÷1,000,000 · 불명확=needs_review. 필드: `amountOriginalUnit`/`amountDisplayMillionKRW`/`amountConversionNote`.
- **1인평균 검산 + flag**: `총액÷인원≈1인평균` 검산, 불일치 시 자동 덮어쓰기 금지. `averageCompensationStatus`(consistent/calculated_from_total/source_value_conflicts_with_total/needs_review) + `averageCompensationNote`. 예: 아이큐어(73,022천원 vs ≈10백만원)=source_value_conflicts_with_total.
- **맥락성 이상치**: `contextFlags`(rehabilitation_proceeding/multiple_director_resignations/former_executive_retirement_pay/retirement_income_included/amended_filing) + `contextNote`. 오류 단정 금지, 원문 보존. 예: 동성제약·세토피아·트리니티항공.
- **PDF 파일명 규칙(필수)**: ASCII만, 한글·`[]`·`()` 금지, 공백 대신 `_`, 회사명 영문/ticker. 예: `dongsung.pdf`…`pharmabcine.pdf`. (한글·NFD·괄호 파일명은 파싱 도구가 읽지 못함.)
- **UI 진단 TODO(미수정)**: 임원표 경력/주식/임기 파싱 · 합계만 있는 회사 직원 차트 · KR 21개사 그룹화 · 원문 표 모달.
- 위 규칙은 공통데이터 프롬프트(앱 「핸드오프 문서」 탭 `promptText`)와 스키마(`contextFlags`/`averageCompensationStatus`/`amountOriginalUnit`)에 반영됨.

## QA(요약)
초기 진입 시 KR 첫 문서 자동 선택·즉시 렌더링, KR/US 상호 배타 선택, US 드롭다운 **23개사**(원문 13 + XBRL 10) 선택 가능,
원문 13개사는 CEO Pay Ratio 게이지·직원 중위보수·CEO 총보수 표시 + PvP는 "추가 추출 대상" 안내, XBRL 10개사는 PvP 차트 노출,
undefined/null/NaN/placeholder 미노출, 콘솔 오류 없음.
