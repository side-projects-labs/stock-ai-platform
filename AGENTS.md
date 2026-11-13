# AGENTS.md

이 문서는 stock-ai-platform 저장소 전체에 적용합니다. 사용자 지시를 우선하며 아래 Hub 관리 구간과 함께 따릅니다.

## 프로젝트와 실행 기준

- 국내·해외 주식 조회, 종목 검색·상세, 가격 차트, 뉴스 기반 분석과 추천을 제공하는 Next.js App Router 웹앱입니다. 배포 주소는 https://stock.ycseng.com 입니다.
- 현재 package.json은 Next.js 16.1.6·React 19.2.3·TypeScript·TanStack React Query 5·Tailwind CSS 4·Lightweight Charts 5를 사용합니다. 버전과 실제 동작은 manifest·lockfile·코드를 기준으로 확인하며 README나 STOCK_INTRO.md만으로 단정하지 않습니다.
- 패키지 매니저는 pnpm이며 lockfile을 유지합니다. 현재 E2E 워크플로는 Node.js 20·pnpm 10을 사용합니다. 다른 저장소의 명령이나 런타임을 그대로 가져오지 않습니다.
- 개발은 pnpm dev, 빌드는 pnpm build, 운영 실행은 pnpm start입니다. 기존 개발 서버와 포트 점유를 먼저 확인하고 무관한 서버를 종료하지 않습니다.

## 구조와 데이터 소유권

- src/app은 페이지·layout·providers, api/**/route.ts는 서버 API 경계를 소유합니다. /는 /domestic으로 이동하며 /foreign, /search, /stock/[id] 페이지가 있습니다.
- src/types/stock.ts의 Stock·StockPriceSnapshot·StockAIAnalysis·StockRecommendation을 공통 계약으로 사용합니다. 임의의 별도 타입이나 시장·감성·추천 action 값을 중복 정의하지 않습니다.
- src/services/kisApi.ts는 KIS 조회·종목 마스터 파싱·가격 이력 정규화, kisTokenIssuer.ts는 토큰 발급, kisTokenStore.ts는 재사용·갱신 동시성 제어를 담당합니다. server-only 경계를 유지합니다.
- kisTokenStore.ts는 Supabase 설정이 있으면 kis_tokens 테이블의 만료·갱신 락을 사용하고, 없으면 메모리 캐시를 사용합니다. 요청 중복 방지·락 해제·만료 검증을 보존하고 토큰 저장 구조를 바꾸면 supabase/kis_tokens.sql과 관련 테스트도 대조합니다.
- src/services/mockApi.ts는 이름과 달리 /api/stocks를 호출하는 실제 클라이언트입니다. 이를 정적 모의 데이터로 대체하거나 클라이언트에서 KIS를 직접 호출하지 않습니다.
- src/hooks/useStocks.ts·useStockDetail.ts와 src/app/providers.tsx가 React Query의 조회 키·캐시·갱신 상태를 소유합니다. 시장·정렬·종목·가격 이력이 다른 요청을 섞지 않고 로딩·오류·스트리밍 상태를 함께 처리합니다.
- src/services/aiAnalysis.ts가 모델·뉴스 컨텍스트·분석 프롬프트를 공유합니다. /api/stock-analysis의 텍스트 스트림과 src/app/actions/stockAnalysis.ts의 비스트리밍 경로를 구분하고 소비자와 함께 계약을 확인합니다.
- 추천은 /api/recommendations → src/services/aiRecommendation.ts로 이어집니다. 후보 종목 제한, 응답 정규화, 점수·신뢰도 범위 보정과 정량 대체 결과를 보존하며 실패 경로별 실제 동작을 확인합니다.

## 데이터 의미와 화면

- 국내·해외 시세 경로는 KIS를 사용하며 실패 시 오류를 반환합니다. 시장을 지정하지 않은 /api/stocks에는 가상 종목 생성 경로가 있으므로 이를 실제 시세로 설명하거나 조용히 대체하지 않습니다. X-Stock-Source와 응답 상태를 유지합니다.
- 가격 이력 길이는 src/constants/history.ts의 HISTORY_POINT_COUNT를 사용합니다. 누락 이력을 보정한 값과 실제 관측치를 구분하며 날짜 순서·통화·등락률 단위를 확인합니다.
- StockChart.tsx는 가격 이력에서 캔들을 구성합니다. 생성한 시가·고가·저가를 거래소가 제공한 실제 OHLC로 설명하지 않습니다. 차트 제거·ResizeObserver 해제와 국내 상승 빨강·하락 파랑 표현을 보존합니다.
- 차트·테마·브라우저 API는 클라이언트 경계에서 사용합니다. Tailwind 4의 src/app/globals.css·postcss.config.mjs 및 @config로 연결된 tailwind.config.ts를 함께 확인합니다. 기존 컴포넌트와 src/lib/cn.ts를 재사용합니다.

## 환경 변수와 외부 서비스

- KIS_APP_KEY·KIS_APP_SECRET·KIS_BASE_URL, GROQ_API_KEY, SERPAPI_KEY·BING_NEWS_KEY, SUPABASE_URL·SUPABASE_SERVICE_ROLE_KEY는 서버 소비 경계를 유지합니다. 값을 문서·로그·오류 출력에 넣거나 NEXT_PUBLIC_*로 옮기지 않습니다.
- .env*·토큰·서비스 역할 키·인증서와 실제 외부 응답의 민감정보를 커밋하지 않습니다. README의 오래된 환경 변수 설명을 실제 소비 코드 확인 없이 필수 계약으로 추가하지 않습니다.
- 자동 검증에서는 KIS·Groq·뉴스·Supabase를 모킹하고 실제 토큰 발급·API 과금·운영 DB 변경을 일으키지 않습니다. 운영 설정·배포 연결·DB 스키마 변경은 별도 요청 범위를 확인합니다.

## 변경과 검증

- 코드 변경은 인접 *.test.ts 또는 *.test.tsx에서 실패 조건을 먼저 재현하고 구현합니다. 기존 Vitest·React Testing Library·jsdom 설정과 server-only 모킹을 재사용합니다.
- 정적 검증은 pnpm lint와 pnpm exec tsc --noEmit입니다. package.json에 typecheck/type-check 스크립트는 없습니다. pnpm test는 Vitest 기본 실행이므로 일회성 검증에는 pnpm exec vitest run을 사용합니다. 커버리지는 pnpm coverage입니다.
- API·서비스 변경은 관련 테스트부터 실행하고 시세 실패, 토큰 캐시·동시 갱신, 분석 스트림·JSON 파싱과 추천 대체 경로를 변경 범위에 맞게 확인합니다. 빌드가 필요하면 pnpm build로 검증하며 외부 설정 부재와 코드 실패를 구분합니다.
- E2E는 pnpm e2e이며 playwright.config.ts가 http://127.0.0.1:3000 서버를 재사용하거나 실행합니다. 기존 서버가 이 프로젝트인지 확인합니다. e2e/stock-flows.spec.ts는 API를 모킹하므로 성공을 실제 KIS·Groq 연동 성공으로 보고하지 않습니다.
- .github/workflows/e2e.yml은 workflow_dispatch 전용입니다. PR·push만으로 E2E가 자동 실행되거나 필수 CI가 모두 통과한다고 가정하지 않습니다.
- 문서만 수정하면 실행 테스트를 억지로 추가하지 않고 경로·명령·실제 구현 근거·민감정보 포함 여부와 Hub 관리 구간 보존을 검증합니다. 실행하지 않은 검증은 통과로 보고하지 않습니다.
- 작업 브랜치에서 요청 범위만 변경하고 사용자 요청 없이 커밋·푸시·PR·머지를 수행하지 않습니다. PROJECT.md 작성·갱신 시 아래 계약을 따르며 기존 slug stock-ai-platform을 유지합니다.

<!-- portfolio-hub:project-guidance:start -->
## 포트폴리오 문서 입력 계약

- 기본 브랜치 루트 PROJECT.md를 프로젝트 소개의 원본으로 유지합니다. 프로젝트 초안 작업에서 이 저장소의 AI가 아래 규격으로 생성하고 내용 변경 시 함께 갱신합니다. Hub는 내용을 AI로 다시 생성하지 않고 검증한 필드를 그대로 전송합니다.
- PROJECT.md는 첫 번째 JSON 코드 블록에 아래 메타데이터를 작성하고, 블록 뒤 빈 줄 다음에 마크다운 본문을 작성합니다. 본문의 제목·목차·내용 구성은 자유입니다.

```json
{
  "schemaVersion": 1,
  "slug": "project-slug",
  "title": "프로젝트 제목",
  "description": "프로젝트 설명",
  "stacks": ["TypeScript"],
  "image": "https://example.com/public-project.png",
  "linkDemo": null,
  "linkGithub": null
}
```

- schemaVersion, slug, title, description, stacks는 필수입니다. slug는 기존 프로젝트 식별자로 유지하며 변경하면 다른 프로젝트가 생성될 수 있습니다. stacks는 문자열 배열입니다. JSON에 계약 밖 필드를 추가하지 않습니다.
- image는 선택적인 공개 HTTP(S) 이미지 URL입니다. URL이 제공된 경우에만 기존 이미지를 교체합니다. 생략/null/빈 문자열이면 기존 이미지를 유지하며, 신규 프로젝트는 이미지 없이 생성합니다. 로컬 파일 경로·비공개 GitHub 원본 URL·자격 증명이 필요한 URL을 넣지 않습니다.
- 대표 이미지 파일을 저장소에 보관할 때는 루트 PROJECT.png를 사용합니다. 적절한 배포 화면이 있으면 활용하고 없으면 이 저장소에서 생성형 썸네일을 준비합니다. 필요한 경우 갱신하며 일반 코드 변경마다 재생성하지 않습니다.
- 공개 이미지 업로드와 공개 URL 준비는 이 저장소가 담당합니다. Hub는 이미지 파일을 읽거나 업로드하지 않습니다. 도구를 사용할 수 없거나 URL을 제공하지 못하면 image를 생략할 수 있습니다.
- linkDemo와 linkGithub는 선택적인 HTTP(S) 링크이며 생략/null/빈 문자열이면 기존 링크 값을 지웁니다. JSON 뒤 마크다운 전체를 API의 content로 사용하므로 중복 생성하지 않습니다.
- 이 규격 문서가 있으면 Hub가 프로젝트를 생성·업데이트하며 즉시 공개하고 관리자 편집 내용도 교체합니다. 공개 가능한 소개만 작성하고 비밀값·내부 정보는 넣지 않습니다. 이 지침은 다른 비공개 파일을 공개할 권한을 부여하지 않습니다.
- 기존 프로젝트 규칙과 함께 적용하며 충돌은 담당자에게 확인합니다. Hub는 이 구간 밖의 기존 규칙과 원본 PROJECT.md를 변경하지 않습니다. 지침 PR은 담당자가 직접 머지합니다.
<!-- portfolio-hub:project-guidance:end -->
