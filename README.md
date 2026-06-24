# Executive, Workforce & Compensation Disclosure UI

KR DART 사업·분기보고서의 **임원·직원 및 보상 현황** 데이터와 US SEC **DEF 14A** 보상 데이터를
하나의 공통 UI로 확인하기 위한 **임시 배포본**입니다. (핸드오프 문서 탭 포함)

> 이번 배포본은 **KR DART 보상 데이터 + US SEC DEF 14A 보상 데이터를 함께 포함한 임시 배포본**입니다.
> US 화면은 데모가 아니라 SEC XBRL Pay-vs-Performance 표준 태그에서 추출한 **실측 데이터(10개사)**를 반영합니다.

## 실행 방법

브라우저에서 `index.html`을 직접 열거나, GitHub/Vercel/Netlify에 업로드해 확인할 수 있습니다.
별도 빌드·설치 과정이 없는 단일 자립형 HTML입니다.

## 배포 방법

### GitHub Pages
1. 이 폴더의 파일을 GitHub 저장소에 업로드합니다.
2. Settings > Pages에서 branch를 선택합니다.
3. `/root` 또는 `/docs` 경로를 선택해 배포합니다.

### Vercel
1. `release/` 폴더를 프로젝트 루트로 업로드합니다.
2. Framework Preset은 `Other` 또는 `Static`으로 설정합니다.
3. Build Command는 비워둡니다.
4. Output Directory도 비워두거나 `.` 로 설정합니다.

### Netlify
1. `release/` 폴더를 드래그&드롭으로 업로드합니다.
2. Build Command / Publish Directory를 비워두거나 `.` 로 둡니다.

## 포함 데이터

**KR — DART 사업·분기보고서 (총 21개사)**
- *기존 샘플 (10개사)*: 에스앤디 · 한텍 · 프리시젼바이오 · 비보존제약 · 수성웹툰 · 코스모화학 · 삼천당제약 · 신한알파리츠 · 현대자동차 · 차바이오텍.
- *원문 PDF 추출 (11개사, 2025 사업연도)*: 동성제약 · 디와이디 · 세토피아 · 쌍방울 · 아이빔테크놀로지 · 아이큐어 · 코오롱글로텍 · 탑선 · 트리니티항공 · 티케이지휴켐스 · 파멥신.
  DART 사업보고서 PDF 원문의 `VIII. 임원 및 직원 등에 관한 사항`에서 임원 명단·직원 현황·이사·감사 보수·개인별 5억원 이상 보수·주식매수선택권 유무를 추출했습니다.
초기 진입 시 KR 첫 문서가 자동 선택되어 바로 렌더링됩니다.

### KR 원문 PDF 추출 11개사 — 범위 / 주의
- **반영 항목**: 회사·문서유형(정정 여부)·기준일 · 임원 명단(성명·직위·구분·상근) · 직원 수(정규/기간제)·1인 평균급여 · 이사·감사 전체 보수총액·유형별 보수 · 주주총회 승인 한도 · 개인별 5억원 이상 보수 · 주식매수선택권 부여 여부.
- **원문 표 참조(추가 정형화 대상)**: 임원 개인별 주요경력·보유주식·재직기간, 주식매수선택권 개인별 수량·행사가·공정가치, 개인별 SCT 세부 구성.
- **금액 단위**: 원문이 천원/백만원/원으로 혼재 → 모두 **백만원**으로 환산 표기.
- **엣지케이스(원문 보존)**: 동성제약은 회생절차 진행 중(2025.06.23 개시 → 2026.03.27 인가) 맥락 주석 유지 · 세토피아는 2025년 이사 다수 사임으로 이사회 변동 · 아이큐어는 원문 1인평균급여 표기(73,022천원)가 연간급여총액÷인원(≈10백만원)과 불일치하여 후자 기준 표기(원문 확인 필요) 주석 유지 · 트리니티항공 등 전직 임원 5억원 이상 퇴직보수는 원문대로 보존.

**US — SEC DEF 14A 실측 (총 23개사)**
- *원문(DEF 14A 텍스트) 파싱 · CEO Pay Ratio 중심 (13개사)*: Texas Instruments · Micron · Qualcomm · UnitedHealth · Deere · Broadcom · Cisco · Boeing · Salesforce · AMD · Caterpillar · 3M · General Electric. (CEO 총보수·직원 중위보수·CEO Pay Ratio 실측)
- *XBRL Pay-vs-Performance 실측 (10개사)*: Apple · Microsoft · Alphabet · Amazon · Meta · NVIDIA · Tesla · JPMorgan · Johnson & Johnson · Walmart.
SEC XBRL(Item 402(v)) Pay-vs-Performance 표준 태그 기반으로 CEO/PEO 총보수(SCT), NEO 1인 평균 총보수,
CAP(Actually Paid Compensation), 연도별 Pay vs Performance 추이를 반영합니다.
US 드롭다운에서 10개사를 선택할 수 있습니다.

### US 데이터 — 현재 범위 / 엣지케이스
- **현재 정형 추출 범위 밖(추가 파싱 필요)**: CEO Pay Ratio · 직원 중위보수 · 이사회 전체 명단 ·
  개인별 SCT 구성요소(급여/상여/주식/옵션 세부 분해) · Outstanding Equity 수량 · 전체 NEO 로스터.
  → 임의 생성하지 않고 UI/HANDOFF에 "추가 파싱 필요"로 명시했습니다.
- **CEO/PEO 중심 임원 표**는 현재 데이터 범위의 한계이며, 빈값처럼 보이지 않도록 안내 라벨을 유지합니다.
- **연차/엣지케이스 원문 보존**: JNJ·NVIDIA처럼 CEO 교체연도·회계연도 시프트로 PvP 연차가 4개이거나
  연도 라벨이 다른 경우 임의로 5개년에 맞추지 않고 원문/XBRL 기준 그대로 유지합니다.
- **Tesla(Elon Musk)**: 연간 SCT 총보수가 사실상 $0(2018년 일괄 옵션 맥락)이므로 "—" 및 맥락 주석을 유지합니다.
- **Amazon(Andrew Jassy)**: 낮은 SCT도 임의 보정 없이 XBRL 실측치 기준으로 유지합니다.

## 외부 의존성

- 폰트 CDN: Google Fonts(Inter), jsDelivr(Pretendard). 네트워크 차단 환경에서는 시스템 폰트로 대체되며 동작에는 영향이 없습니다.
- 그 외 런타임 라이브러리는 `index.html` 내부에 인라인되어 있어 오프라인에서도 실행됩니다.

## 주의사항

- 본 산출물은 임시 검증/시연용입니다.
- 회사별 데이터는 현재 단일 HTML 내부에 포함되어 있습니다(외부 JSON 분리는 이번 release에서 진행하지 않음).
- 향후 운영 구조에서는 `/data/*.json` 외부화 및 단일 렌더러 구조 분리를 권장합니다(`HANDOFF.md` 참조).
