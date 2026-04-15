# sitecheck — 웹사이트 헬스체크 대시보드 구조

## 개요

관리 중인 웹사이트들의 HTTP 응답 상태를 주기적으로 자동 체크하고,  
결과를 대시보드 페이지로 보여주는 시스템.

접속 URL: `https://photonrusher.github.io/sitecheck`

---

## 파일 구조

```
photonrusher.github.io/
├── sitecheck/
│   ├── sites.json        ← ✏️  체크할 사이트 목록 (직접 수정하는 파일)
│   ├── status.json       ← 🤖  GitHub Actions가 자동으로 덮어씀 (수정 불필요)
│   └── index.html        ← 🌐  대시보드 UI (브라우저에서 접속하는 페이지)
└── .github/workflows/
    └── health-check.yml  ← ⏱️  자동 체크 스케줄 정의
```

---

## 동작 흐름

### ① 자동 체크 (GitHub Actions 러너가 수행)

```
GitHub Actions 스케줄러 (10분마다 실행)
  └── ubuntu-latest 러너 VM 생성
  └── 저장소 checkout
  └── sitecheck/sites.json 읽기
  └── 각 URL에 curl 요청 → HTTP 상태코드 + 응답시간(ms) 수집
  └── 결과를 sitecheck/status.json으로 저장
  └── 저장소에 자동 커밋 & push
```

> **핵심**: curl을 실행하는 주체는 GitHub Pages 웹서버가 아니다.  
> GitHub Actions 러너(Ubuntu VM)가 직접 외부 사이트에 curl 요청을 보낸다.  
> GitHub Pages는 정적 파일만 서빙하며 서버 측 코드를 실행하지 않는다.

### ② 대시보드 조회 (브라우저)

```
브라우저에서 sitecheck/index.html 접속
  └── JS가 fetch()로 sitecheck/status.json 읽기
  └── 테이블 형태로 결과 렌더링
      - 200번대 응답 → 🟢 정상
      - 300번대 응답 → 🟡 리다이렉트
      - 그 외 / 연결실패 → 🔴 오류
  └── 마지막 체크 시간 표시
```

---

## 각 파일 역할 상세

### `sitecheck/sites.json` — 체크 대상 목록

체크할 사이트를 여기에 추가/삭제한다. Actions가 이 파일을 읽어 순서대로 curl 요청.

```json
{
  "sites": [
    {"name": "표시할 이름",  "url": "https://체크할주소.com"},
    {"name": "사이트 2",     "url": "https://example.org"}
  ]
}
```

### `sitecheck/status.json` — 체크 결과 (자동 생성)

Actions가 체크 후 이 파일을 덮어쓴다. 손으로 수정할 필요 없음.

```json
{
  "updated": "2026-04-15 10:00 UTC",
  "sites": [
    {"name": "사이트명", "url": "https://...", "status": 200, "ms": 312},
    {"name": "사이트명", "url": "https://...", "status": 0,   "ms": 0}
  ]
}
```

- `status`: HTTP 응답 코드. 연결 실패 시 `0`
- `ms`: 응답 시간 (밀리초). `-L` 플래그로 리다이렉트 포함한 최종 응답 시간

### `sitecheck/index.html` — 대시보드 UI

- 순수 HTML + JS (외부 라이브러리 없음)
- `fetch('./status.json?t=타임스탬프)`로 캐시 무력화 후 최신 결과 로드
- 새로고침 버튼 / 60초 자동 새로고침 체크박스 제공
- 기존 시작페이지와 동일한 VS Code 계열 다크 테마

### `.github/workflows/health-check.yml` — 자동 실행 워크플로우

- `schedule: cron: '*/10 * * * *'` → 10분마다 자동 실행
- `workflow_dispatch` → GitHub Actions 탭에서 수동 실행 버튼 제공
- Python으로 sites.json 읽고 curl 결과 파싱 후 status.json 작성
- `[skip ci]` 태그로 커밋 후 재귀적 Actions 실행 방지

---

## 사이트 추가/제거 방법

`sitecheck/sites.json`만 수정하고 push하면 다음 실행부터 반영된다.

```json
{
  "sites": [
    {"name": "기존 사이트",    "url": "https://existing.com"},
    {"name": "새로 추가",      "url": "https://new-site.com"},
    {"name": "여기 삭제하면",  "url": "https://remove-me.com"}
  ]
}
```

---

## 최초 설정 시 필요한 GitHub 저장소 설정

Actions가 `status.json`을 저장소에 커밋하려면 쓰기 권한이 필요하다.

```
GitHub 저장소 → Settings → Actions → General
→ Workflow permissions → "Read and write permissions" 선택 → Save
```

이 설정 없이는 curl 체크는 되지만, status.json 커밋 단계에서 실패한다.

---

## 수동 즉시 실행

```
GitHub 저장소 → Actions 탭
→ "Site Health Check" 워크플로우 선택
→ "Run workflow" 버튼 클릭
```

push 후 10분을 기다리지 않고 바로 결과를 확인할 때 사용.
