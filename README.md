# BiteForecast_docs

입질예보(BiteForecast) 프로젝트 문서 저장소

## 발표 자료 (읽는 순서)

1. [기획 의도](01_기획/기획의도.md)
2. [차별성](01_기획/차별성.md)
3. [요구사항 요약](01_기획/요구사항_요약.md)
4. [화면 설계 — 홈·어종 탐색 (스크린샷)](05_UX/화면_설계.md)
5. [용어 풀이 — 바다낚시지수·파고·풍속·수온·물때·특보](01_기획/용어_풀이.md)
6. [외부 API](03_데이터/외부_API.md)
7. [시스템 아키텍처](06_개발환경/시스템_아키텍처.md)
8. [서버 배포 (Fly.io)](03_데이터/서버_배포.md)
9. [DB (Supabase)](03_데이터/DB_Supabase.md)
10. [자동 수집 (GitHub Actions)](06_개발환경/자동_수집.md)
11. [API 명세](02_명세/API_명세.md)
12. [기능명세서](02_명세/기능명세서.md)
13. [iOS 화면 검증](05_UX/iOS_화면_검증.md)

<details>
<summary>참고 자료</summary>

- 설계 과정: [홈 설계 과정 이력](05_UX/이력/어종_홈_설계_과정.md)
- 데이터 검증: [공공 API 검증 결과](03_데이터/공공API_검증결과.md) · [좌표 품질 검증](03_데이터/2026-11-09_좌표품질_검증보고서.md) · [주소 API 좌표 대조](03_데이터/2026-11-10_주소API_좌표대조_보고서.md)
- 이전 검토안 (실제 배포 전에 쓴 문서): [저비용 출시용 기술 아키텍처](06_개발환경/2026-11-11_저비용_출시용_기술아키텍처.md) · [Supabase·DigitalOcean 배포 전환 설계](06_개발환경/2026-11-11_Supabase_DigitalOcean_배포전환_설계.md) · [Azure 배포 계획](06_개발환경/Azure_배포_계획.md) · [작업 메모: 특보·추천 정확성](06_개발환경/2026-11-10_내일작업_특보·추천정확성.md)
- 실행: [실행 가이드](06_개발환경/실행_가이드.md)

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
