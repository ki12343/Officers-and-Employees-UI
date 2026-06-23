# Executive, Workforce & Compensation Disclosure UI

KR DART 사업보고서의 임원·직원 및 보상 현황 데이터를 공통 UI로 확인하기 위한 임시 배포본입니다.
(US SEC DEF 14A 대응 화면 및 핸드오프 문서 탭 포함)

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

현재 UI에는 KR DART 사업보고서 샘플 데이터(에스앤디·한텍·프리시젼바이오·비보존제약·수성웹툰·코스모화학·삼천당제약·신한알파리츠·현대자동차·차바이오텍)와
US SEC DEF 14A 데모 데이터가 포함되어 있습니다. 초기 진입 시 KR 첫 문서가 자동 선택되어 바로 렌더링됩니다.

## 외부 의존성

- 폰트 CDN: Google Fonts(Inter), jsDelivr(Pretendard). 네트워크 차단 환경에서는 시스템 폰트로 대체되며 동작에는 영향이 없습니다.
- 그 외 런타임 라이브러리는 `index.html` 내부에 인라인되어 있어 오프라인에서도 실행됩니다.

## 주의사항

- 본 산출물은 임시 검증/시연용입니다.
- 회사별 데이터는 현재 단일 HTML 내부에 포함되어 있습니다.
- 향후 운영 구조에서는 `/data/*.json` 외부화 및 단일 렌더러 구조 분리를 권장합니다(`HANDOFF.md` 참조).
