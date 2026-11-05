# 공공 API 검증 결과

검증일: 2026-11-05 · 실제 인증키로 호출해 확인

## 국립해양조사원 (공공데이터포털, `DATA_GO_KR_API_KEY`)

기본 주소: `https://apis.data.go.kr/1192136/{서비스}/{오퍼레이션}`
공통 파라미터: `serviceKey`, `type=json`, `pageNo`, `numOfRows`(최대 300), `obsCode`, `reqDate`(yyyyMMdd)

| 데이터 | 경로 | 상태 | 주요 응답 필드 |
|---|---|---|---|
| 조석예보(고·저조) | `tideFcstHghLw/GetTideFcstHghLwApiService` | ✅ 사용 중 | `obsvtrNm`, `lat`, `lot`, `predcDt`, `predcTdlvVl`(cm), `extrSe` |
| 해양관측부이 최신 관측 | `twRecent/GetTWRecentApiService` (+`min=60`) | ✅ 사용 중 | `obsrvnDt`, `wvhgt`(파고 m), `wspd`(풍속 m/s), `wtem`(수온 ℃), `wndrct`, `artmp` 등 |
| 조석예보(시계열) | `tideFcstTime/GetTideFcstTimeApiService` (+`min`) | ⛔ 활용신청 필요 | 미검증 |

**확인된 동작**
- `extrSe`: 1·3 = 고조(만조), 2·4 = 저조(간조). 군산 2026-11-05 응답(2=164cm, 3=546cm, 4=157cm)으로 확인했고, 수집할 때마다 값으로 다시 검증해 경고를 남긴다.
- 시각(`predcDt`, `obsrvnDt`)은 KST이고 시간대 표기가 없다. 서버에서 KST로 해석한 뒤 UTC로 저장한다.
- 부이 값은 결측일 때 `null`로 온다(0과 구분). 항만 부이 중에는 파고·풍속이 없고 수온만 있는 곳이 있다.
- **짧은 시간에 연속 호출하면 HTTP 429가 난다.** 수집기는 호출 간격을 0.4초로 두고 429면 재시도한다.
- 미신청 API는 **HTTP 403 + JSON 오류 본문**(`returnReasonCode: 30`, `SERVICE_KEY_IS_NOT_REGISTERED_ERROR`)으로 응답한다.
- 관측소 목록 API는 따로 쓰지 않았다. 코드 범위(DT_0001~0099, TW_0001~0099, KG_0001~0029)를 직접 조회해서 응답한 72곳을 `backend/seeds/stations.json`에 저장했다.

## 기상청 API허브 (`KMA_API_KEY`)

| 데이터 | 경로 | 상태 |
|---|---|---|
| 단기예보 (파고 WAV·풍속 WSD) | `api/typ02/openApi/VilageFcstInfoService_2.0/getVilageFcst` | ⛔ 403 "활용신청이 필요한 API" |
| 해양기상부이 | `api/typ01/url/kma_buoy.php` | ⛔ 403 "활용신청이 필요한 API" |

인증키 자체는 유효하다. API허브는 **API마다 따로 활용신청**해야 한다.

## 지원 해역 (명세 D-3 제안)

조석 예보지점과, 파고·풍속을 모두 관측하는 부이가 가까운 곳을 기준으로 골랐다.

| 해역 | 장소 | 조석 지점 (거리) | 부이 (거리) |
|---|---|---|---|
| 속초 | 동명항 | DT_0012 속초 (0.7km) | TW_0093 속초해수욕장 (2.9km) |
| 속초 | 대포항 | DT_0048 속초등표 (3.1km) | TW_0093 속초해수욕장 (3.6km) |
| 보령 | 대천항 | DT_0025 보령 (8.7km) | TW_0069 대천해수욕장 (7.8km) |
| 보령 | 무창포항 | DT_0025 보령 (18.1km) | TW_0069 대천해수욕장 (7.6km) |
| 보령 | 오천항 | DT_0025 보령 (4.8km) | TW_0069 대천해수욕장 (19.2km) |
| 부산 | 미포 | DT_0005 부산 (14.3km) | TW_0062 해운대해수욕장 (1.0km) |
| 부산 | 청사포 | DT_0005 부산 (15.9km) | TW_0062 해운대해수욕장 (2.4km) |
| 부산 | 송정 | DT_0005 부산 (17.4km) | TW_0090 송정해수욕장 (2.4km) |

- 장소 좌표는 OpenStreetMap Nominatim에서 조회한 항구·지명의 대표 위치다. 정확한 낚시 지점이 아니다.
- 규제 상태는 공식 고시를 조사하기 전이라 모든 장소가 「확인되지 않음」이다 (FR-17).

## 남은 일

- [ ] 공공데이터포털: **조석예보(시계열)** 활용신청 → 시간별 조위 그래프
- [ ] 기상청 API허브: **단기예보** 활용신청 → 미래 날짜 파고·풍속 예보
- [ ] 이용허락범위 확인 (공공누리 제1유형: 출처표시) → 앱 설정 화면에 출처 표기
- [ ] 해역별 낚시통제구역 고시 수동 조사 (FR-17)
