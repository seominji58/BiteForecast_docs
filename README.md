# BiteForecast_docs

입질예보(BiteForecast) 프로젝트 문서 저장소

## 발표 자료 바로가기

처음 보는 사람은 아래 순서로 확인하면 됩니다.

1. [발표용 요구사항 요약](01_기획/2026-11-11_발표용_요구사항_요약.md)
2. [시스템 아키텍처·주요 플로우차트](06_개발환경/2026-11-11_시스템_아키텍처_및_플로우차트.md)
3. [API 명세 요약](02_명세/2026-11-11_발표용_API_명세.md)
4. [기능명세서 전체](02_명세/입질예보_기능명세서.md)
5. [iOS UI 검증 결과](05_UX/2026-11-10_iOS_UI검증_결과.md)
6. [내일 작업·특보와 추천 정확성](06_개발환경/2026-11-10_내일작업_특보·추천정확성.md)
7. [저비용 출시용 기술 아키텍처](06_개발환경/2026-11-11_저비용_출시용_기술아키텍처.md)

## 현재 구현 상태

- Backend: [BiteForecast_backend](https://github.com/seominji58/BiteForecast_backend)
- Frontend: [BiteForecast_frontend](https://github.com/seominji58/BiteForecast_frontend)
- API 서버·수집기: 개발 Mac 로컬 실행
- DB: SQLite(`biteforecast.db`)
- 외부 운영 배포: 아직 하지 않음(저비용 배포안 검토 중)
- 공식 데이터: 기상청 단기예보·해상 특보, 국립해양조사원 조석·부이·바다낚시지수
- 원칙: 자료 없음은 0으로 채우지 않으며, 해상 조건을 실제 조황·어획 확률로 해석하지 않음

| 폴더 | 내용 |
|---|---|
| `01_기획/` | 기획안, 기능요구사항 정의서 (원본 docx) |
| `02_명세/` | 기능명세서, API 명세 |
| `03_데이터/` | 공공 API 검증 결과, 이용허락 조건, 규제 고시 조사 |
| `04_회의록/` | 회의록, 결정 사항 기록 |
| `05_UX/` | UX·기능 검토, 화면 설계 |
| `06_개발환경/` | **서버·앱 실행 가이드**, 공공데이터 자동 수집 운영 설계안(검토용) |

관련 레포
- Backend (FastAPI): https://github.com/seominji58/BiteForecast_backend
- Frontend (React Native + Expo, iOS·Android): https://github.com/seominji58/BiteForecast_frontend
