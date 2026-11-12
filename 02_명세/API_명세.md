# API 명세

기본 주소(배포): `https://biteforecast-api.fly.dev/api/v1` · 로컬 개발: `http://127.0.0.1:8001/api/v1`  
인증: 로그인 API는 Bearer 토큰, 비로그인 기능은 `X-Device-Id`를 사용한다.

## 공개 조회 API

| 메서드 | 경로 | 용도 |
|---|---|---|
| GET | `/health` | 서버 상태 확인 |
| GET | `/regions` | 지원 지역 목록 |
| GET | `/places` | 낚시터 목록·검색 |
| GET | `/places/{place_id}` | 낚시터 기본 상세 |
| GET | `/places-summary?date=YYYY-MM-DD` | 지도·홈용 장소별 요약 |
| GET | `/fishing-index?date=YYYY-MM-DD` | 그날 공식 바다낚시지수 — 지점별 어종·오전/오후 등급·물때, 공식 지점/근처(10km) 낚시터 구분 (홈 어종 탐색) |
| GET | `/places/{place_id}/conditions?date=YYYY-MM-DD` | 장소 상세 해황·특보·낚시지수 |
| GET | `/compare?place_a=&place_b=&date=` | 두 장소 비교 |
| GET | `/meta/date-range` | 조회 가능한 날짜 범위 |
| GET | `/meta/data-status` | 자료별 최신성·수집 시각 |
| GET | `/meta/sources` | 데이터 출처 목록 |
| GET | `/places/{place_id}/regulation` | 규제·출입 정보 |

## 사용자·기록 API

| 메서드 | 경로 | 용도 |
|---|---|---|
| POST | `/auth/signup` | 회원가입 |
| POST | `/auth/login` | 로그인 |
| GET/POST/PUT/DELETE | `/journals` | 낚시일지 조회·작성·수정·삭제 |
| GET | `/me/trips` | 출조계획 목록 |
| POST/PATCH/DELETE | `/me/trips/{trip_id}` | 출조계획 수정 |
| POST/PATCH/DELETE | `/me/trips/{trip_id}/stops` | 계획 장소 관리 |
| GET | `/me/trips/{trip_id}/outlook` | 계획 장소 해황 요약 |
| GET/PUT/DELETE | `/me/favorites/{place_id}` | 즐겨찾기 |
| GET/POST | `/community/posts` | 커뮤니티 |

## 주요 응답 원칙

- `wave_obs`와 `wave_fc_max`, `wind_obs`와 `wind_fc_max`를 섞지 않는다.
- `warning.level`은 `warning`, `nearby`, `preliminary`, `none`, `unknown`으로 구분한다.
- `warning.active[].areas`에는 기상청 원문의 적용 해역이 들어간다.
- `fishing_index`는 국립해양조사원 원본 등급과 범위를 그대로 전달한다.
- 최신성 부족 시 `stale`로 표시하며 안전 또는 조건 충족으로 해석하지 않는다.

## 데이터 출처

[외부 API](../03_데이터/외부_API.md) 참고.
