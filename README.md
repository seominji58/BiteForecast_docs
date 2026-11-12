# BiteForecast_docs

입질예보(BiteForecast) 프로젝트 문서 저장소

## 발표 자료 (읽는 순서)

1. [발표 자료 — 한 문서 요약](발표_자료.md)
2. [개발 배경 및 기획 의도](01_기획/2026-11-12_개발배경_및_외부API.md#1-개발-배경-및-기획-의도)
3. [차별성](01_기획/2026-11-12_개발배경_및_외부API.md#4-차별성--입질예보만의-강점)
4. [주요 정보와 용어 풀이 (바다낚시지수·파고·풍속·수온·물때·특보)](01_기획/2026-11-12_개발배경_및_외부API.md#3-주요-정보와-용어-풀이)
5. [사용 중인 외부 API](01_기획/2026-11-12_개발배경_및_외부API.md#2-사용-중인-외부-api서비스)
6. [요구사항 요약](01_기획/2026-11-11_발표용_요구사항_요약.md)
7. [기능명세서](02_명세/입질예보_기능명세서.md)
8. [화면 설계·구현 — 어종 중심 홈](05_UX/2026-11-11_어종중심_홈_설계안.md)
9. [iOS 화면 검증 결과](05_UX/2026-11-10_iOS_UI검증_결과.md)
10. [API 명세](02_명세/2026-11-11_발표용_API_명세.md)
11. [시스템 아키텍처](06_개발환경/2026-11-11_시스템_아키텍처_및_플로우차트.md)
12. [서버 배포 (Fly.io)](03_데이터/2026-11-11_Fly_배포.md)
13. [DB 전환·성능 개선 (Supabase)](03_데이터/2026-11-11_Supabase_사본.md)

<details>
<summary>참고: 이전 검토안 (실제 배포 전에 쓴 문서)</summary>

- [저비용 출시용 기술 아키텍처](06_개발환경/2026-11-11_저비용_출시용_기술아키텍처.md)
- [Supabase·DigitalOcean 배포 전환 설계](06_개발환경/2026-11-11_Supabase_DigitalOcean_배포전환_설계.md)
- [Azure 배포 계획](06_개발환경/Azure_배포_계획.md)
- [작업 메모: 특보·추천 정확성](06_개발환경/2026-11-10_내일작업_특보·추천정확성.md)

</details>

## 현재 구현 상태

- Backend: [BiteForecast_backend](https://github.com/seominji58/BiteForecast_backend)
- Frontend: [BiteForecast_frontend](https://github.com/seominji58/BiteForecast_frontend)
- API 서버: Fly.io(도쿄) `https://biteforecast-api.fly.dev/api/v1`
- DB: Supabase PostgreSQL(서울) — 로컬 개발·테스트는 SQLite 도 가능
- 수집기: GitHub Actions 매시 17분 (개발 Mac 자동 수집은 등록하지 않음)
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
