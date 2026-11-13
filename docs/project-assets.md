# 프로젝트 문서 이미지

- 원본은 2026-11-13 공개 배포 서비스 https://stock.ycseng.com 을 Playwright Chromium으로 연 화면입니다. 모의 응답이나 생성형 이미지를 사용하지 않았습니다.
- 브라우저 viewport는 1600×1100, device scale factor는 1입니다. 목록·검색 이미지는 Stock Overview 영역, 차트 이미지는 상세 페이지 콘텐츠 영역을 캡처했습니다.
- public/project/stock-list.png: /domestic 의 실제 종목 목록입니다.
- public/project/stock-search.png: /search?q=삼성전자 의 실제 검색 결과 목록입니다.
- public/project/stock-chart.png: /stock/kis-114800?market=domestic&ranking=volume 의 실제 상세 차트입니다.
- 캡처 속 가격은 촬영 시점의 조회 결과입니다. 차트는 현재 코드가 가격 이력으로 구성한 시각화이며 실제 거래소 OHLC 캔들로 설명하지 않습니다.
- PROJECT.md는 이미지가 저장된 커밋 71304c216b31a85a7f6022801e9d356caba703e3의 public/project 파일을 공개 raw GitHub URL로 참조합니다. 해당 커밋을 푸시한 뒤 PR과 공개 문서에서 이미지를 볼 수 있습니다. 로컬 검증은 URL에 대응하는 파일을 연결해 확인합니다.
- PROJECT.md에서 image 메타데이터는 생략해 포트폴리오에 이미 등록된 대표 이미지를 유지합니다. 본문 캡처 이미지와 대표 이미지의 역할을 구분합니다.

## 분석 패널 추가

- 2026-11-13 운영 배포 커밋 e993a99287df97de97fc463be0bef0609645ff73의 Vercel 성공 후 /search?q=삼성전자 화면에서 분석 API의 HTTP 200 텍스트 스트림과 결과 표시를 확인했습니다.
- public/project/stock-analysis.png는 실제 분석 패널 영역 캡처입니다. 분석 내용은 촬영 시점의 생성 결과이며 투자 성과나 사실 검증을 의미하지 않습니다.
- 요약·상승 요인·하락 요인·주요 리스크·뉴스 요약이 표시되는 상태를 캡처했습니다. 화면 내용 변경이나 모의 응답을 사용하지 않았습니다.
- 분석 패널 이미지는 고정 커밋 68311982080b1bdbebba50951db624e49b77152d의 공개 raw URL로 연결합니다. 기존 대표 이미지와 본문 이미지 3장은 유지합니다.
