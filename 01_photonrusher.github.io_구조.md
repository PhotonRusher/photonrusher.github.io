# photonrusher.github.io — 시작페이지(북마크) 구조

## 개요

GitHub Pages로 호스팅되는 개인 웹 북마크 페이지.
브라우저 시작 페이지로 사용하며, URL 파라미터(`?topic=`)로 다양한 주제별 북마크 묶음을 전환할 수 있다.

접속 URL: `https://photonrusher.github.io/`

---

## 파일 구조

```
photonrusher.github.io/
├── index.html            ← 메인 페이지 (북마크 렌더러)
└── siteinfo/
    ├── main.json         ← "main" 주제 북마크 데이터
    └── yk.json           ← "yk" 주제 북마크 데이터 (기본값)
```

---

## 동작 흐름

```
① 브라우저가 index.html 로드
       ↓
② JS가 URL 파라미터 ?topic= 값 읽기
   - 파라미터 없음       → siteinfo/yk.json 로드 (기본)
   - ?topic=main         → siteinfo/main.json 로드
   - ?topic=test         → 페이지 내 내장 JSON 사용 (로컬 테스트용)
       ↓
③ XMLHttpRequest로 siteinfo/{topic}.json 비동기 요청
       ↓
④ JSON 파싱 후 카테고리 → 그룹 → 사이트 순서로 HTML 생성
       ↓
⑤ #siteList div에 삽입하여 화면 표시
```

---

## JSON 데이터 구조 (`siteinfo/*.json`)

```json
{
  "name": "yk",           // 주제 ID (URL 파라미터 값과 일치)
  "ename": "photonrusher",
  "hname": "6071",
  "category": [
    {
      "categoryname": "카테고리명",
      "group": [
        {
          "groupname": "그룹명",
          "sites": [
            { "url": "https://example.com", "name": "표시이름" }
          ]
        }
      ]
    }
  ]
}
```

계층 구조: **카테고리 → 그룹 → 사이트 (링크)**

---

## 새 주제(topic) 추가 방법

1. `siteinfo/` 폴더에 `새주제.json` 파일 추가 (위 JSON 구조 형식 따름)
2. 접속: `https://photonrusher.github.io/?topic=새주제`

---

## 화면 구성

- 검정 배경 + VS Code 계열 색상 테마 (파랑 `#569CD6`, 노랑 `#D7BA7D`, 초록 `#6A9955`)
- 폰트: Noto Sans KR
- jQuery + Bootstrap 4 사용
- 카테고리는 `<h4>` 제목, 그룹은 `<li>` 항목, 링크는 `<a>` 태그로 렌더링

---

## 로컬 테스트

```
file:///C:/MyWorkPrj/04_worksplace/photonrusher.github.io/index.html?topic=test
```

`index.html` 하단의 `<script id="localjson" type="application/json">` 블록 안의 JSON을 수정하면  
서버 없이 로컬에서 바로 테스트 가능.
