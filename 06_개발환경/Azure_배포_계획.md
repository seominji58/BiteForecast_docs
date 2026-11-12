# Azure 배포 계획 (검토용, 2026-11-09)

> ⚠️ **이전 검토안** — 실제 배포 전에 작성한 문서입니다. 현재 구성은 Fly.io(서버) + Supabase(DB) + GitHub Actions(수집) 입니다 → [발표 자료](../06_개발환경/시스템_아키텍처.md) · [Fly 배포](../03_데이터/서버_배포.md)


> 상태: **계획만 — 서버 생성·과금·배포는 하지 않았다.** 승인 후 진행한다.
> 전제(사용자 결정 2026-11-09): Linux VM 1대에서 API 서버와 수집기를 함께 실행, 클라우드는 Azure 우선, systemd timer 자동 수집, 이메일 알림. 개발 Mac 에는 자동 수집을 등록하지 않는다.

---

## 1. 서버 사양 (권장)

| 항목 | 권장 | 근거 |
|---|---|---|
| VM | **B2s** (2 vCPU · 4GB, 버스트형) | API(FastAPI)는 가볍지만, 사진 업로드 시 이미지 처리(Pillow)와 매시 수집(약 4분, 수천 건 저장)이 겹칠 수 있음. 1GB·2GB 메모리는 여유가 적음 |
| 최소 대안 | B1ms (1 vCPU · 2GB) | 초기 사용자 적을 때. 수집 중 응답이 느려질 수 있음 |
| OS | Ubuntu Server 24.04 LTS | 장기 지원, systemd 기본 |
| 지역 | Korea Central (서울) | 공공데이터 API·사용자와 가까움, 개인정보(기록·사진) 국내 보관 |
| 디스크 | OS 30GB + 데이터 디스크 **Standard SSD 64GB** | DB(SQLite, 현재 약 7MB)·사진(1장 약 0.3MB, 1만 장 ≈ 3GB)·로그·로컬 백업 |
| 네트워크 | 고정 공인 IP 1개, 인바운드 22(관리자 IP만)·80·443 | 앱은 HTTPS 로만 접속 |
| 웹 서버 | nginx (HTTPS 종단, Let's Encrypt 무료 인증서) → uvicorn | |
| 런타임 | Python 3.13 (uv 또는 deadsnakes) + 가상환경 | 개발 환경과 같은 버전 |

## 2. 월 운영비 구성 (대략값 — 실제 금액은 Azure 가격 계산기로 확인 필요)

| 항목 | B2s 구성 | B1ms 구성 | 비고 |
|---|---|---|---|
| VM (종량제, Linux) | 약 $35~45 | 약 $15~20 | 1년 예약 시 30~40% 절감 |
| 데이터 디스크 Standard SSD 64GB | 약 $5~6 | 약 $5~6 | |
| OS 디스크 | 약 $2~5 | 약 $2~5 | |
| 공인 IP (Standard, 고정) | 약 $3~4 | 약 $3~4 | |
| 아웃바운드 트래픽 | 대부분 $0 | 대부분 $0 | 월 100GB 까지 무료 범위 안 예상 (사진 조회량에 따라 증가) |
| 백업 (Azure Backup 또는 Blob 저장소) | 약 $5~10 | 약 $5~10 | DB·사진 매일 백업 |
| 이메일 (Azure Communication Services Email) | 1달러 미만 | 1달러 미만 | 알림 메일만 (월 수십 통) |
| 도메인 | 연 $10~20 | 연 $10~20 | 앱이 부를 고정 주소 (예: api.도메인) |
| **합계 (월)** | **약 $50~70** | **약 $30~45** | 환율·가격 변동, 세금 별도 |

> 비용 알림: Azure 「비용 관리 → 예산」에서 월 예산(예: $80)과 80% 도달 알림을 먼저 걸어 둔다.

## 3. 자동 수집 구성 (systemd)

### 3-1. 서비스 단위
| 단위 | 역할 |
|---|---|
| `biteforecast-api.service` | API 서버 (uvicorn, 실패 시 자동 재시작) |
| `biteforecast-collect.service` | 수집 1회 실행 (Type=oneshot). 실패하면 종료 코드 ≠ 0 |
| `biteforecast-collect.timer` | **매시 5분** 실행, `Persistent=true` → 서버가 꺼져 있던 동안 놓친 실행은 켜지자마자 1회 실행 |
| `biteforecast-collect-retry` | 수집 실패 시 **10분 뒤 1회 재실행** (`Restart=on-failure`, `RestartSec=10min`, 1시간에 1회로 제한) |
| `biteforecast-alert@.service` | 수집 실패(`OnFailure=`) 시 관리자에게 이메일 |
| `biteforecast-watchdog.timer` | 30분마다 「마지막 성공」 확인 → 예보 발표 4시간 10분·특보 확인 3시간 넘게 없으면 이메일 (수집기가 아예 안 도는 경우까지 잡음) |

```ini
# /etc/systemd/system/biteforecast-collect.timer (초안)
[Timer]
OnCalendar=*-*-* *:05:00
Persistent=true
RandomizedDelaySec=60

# /etc/systemd/system/biteforecast-collect.service (초안)
[Service]
Type=oneshot
User=biteforecast
WorkingDirectory=/opt/biteforecast/backend
EnvironmentFile=/etc/biteforecast/backend.env      # 권한 600, 인증키는 여기에만
ExecStart=/opt/biteforecast/backend/.venv/bin/python -m app.jobs.collect
TimeoutStartSec=50min
Restart=on-failure
RestartSec=10min
OnFailure=biteforecast-alert@%n.service
```

### 3-2. 마지막 성공 시각·실패 기록 (코드 변경 — 승인 후 구현)
- `collect_run`: 실행마다 시작·종료 시각, 상태(성공 / 일부 실패 / 실패), 자료별 건수·오류 수, 오류 요약(인증키 가림)
- `dataset_status`: 자료별 **마지막 시도·마지막 성공 시각**, 연속 실패 횟수, 마지막 오류, **최근 발표 시각, 예보 유효 범위**
  - 수집 성공 시각과 예보 발표 시각·유효 범위를 따로 관리 (5번 검토 결과)
- 수집기 종료 코드: 전부 실패 → 2, 일부 실패 → 0(기록만), 성공 → 0
- 중복 실행 방지: DB 잠금 행(만료 50분) — 재시도와 정규 실행이 겹치지 않게
- 관리자 화면에 「수집 상태」 표 (자료별 마지막 성공, 연속 실패)

### 3-3. 이메일 알림
- 발송: **Azure Communication Services Email** (Azure VM 은 25번 포트 외부 메일 발송이 막혀 있어 직접 SMTP 대신 사용)
- 받는 사람·발신 주소는 환경변수(`ALERT_EMAIL_TO`, `ACS_EMAIL_CONNECTION` 등)로 — 저장소에 넣지 않음
- 알림 조건: 수집 실행 실패 / 같은 자료 3회 연속 실패 / 예보·특보 오래됨(위 기준) / 디스크 85% 이상
- 같은 알림은 6시간에 1번만 (메일 폭주 방지)

## 4. 배포 절차 (승인 후)

1. **Azure 준비**: 리소스 그룹, 예산 알림, VM(B2s, Ubuntu 24.04, Korea Central), 데이터 디스크, 고정 IP, 네트워크 보안 그룹(22는 관리자 IP만)
2. **서버 기본 설정**: 관리자 계정·SSH 키만 허용, 자동 보안 업데이트, 방화벽(ufw), 시간대 Asia/Seoul, 데이터 디스크 `/srv/biteforecast` 마운트
3. **앱 설치**: `biteforecast` 시스템 사용자, 백엔드 코드 배포(git clone 특정 태그), 가상환경·의존성 설치
4. **비밀값**: `/etc/biteforecast/backend.env` (권한 600) — `DATA_GO_KR_API_KEY`, `ADMIN_TOKEN`, `GOOGLE_CLIENT_IDS`, `MEDIA_DIR=/srv/biteforecast/media`, `DATABASE_URL=sqlite:////srv/biteforecast/biteforecast.db`, 이메일 설정
5. **데이터 옮기기**: 개발 DB 를 그대로 쓸지, 장소·관측소만 새로 적재(`python -m seeds.load`)할지 결정 → 운영은 **새로 적재 + 테스트 데이터 제외** 권장
6. **서비스 등록**: api.service, collect.service/timer, alert, watchdog → `systemctl enable --now`
7. **nginx + HTTPS**: 도메인 연결, certbot 으로 인증서, 업로드 크기 제한 12MB
8. **확인**: `/api/v1/health`, `/api/v1/meta/data-status`(stale=false), 관리자 화면, 수집 타이머 다음 실행 시각(`systemctl list-timers`)
9. **앱 연결**: 앱 `.env` 의 `EXPO_PUBLIC_API_URL=https://<도메인>/api/v1` 로 바꿔 빌드(EAS)
10. **백업**: 매일 새벽 DB(`sqlite3 .backup`)·사진을 Blob 저장소로, 30일 보관. 복구 연습 1회
11. **운영 전환 후**: Alembic 마이그레이션 도입 (지금의 「새 컬럼 자동 추가」는 개발용)

## 5. 남은 결정
1. VM 크기: B2s(권장) / B1ms
2. 도메인 이름
3. 알림 받을 이메일 주소
4. 운영 DB: 새로 적재(권장) / 개발 DB 복사
