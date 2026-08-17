# 코지 도구 모음 (cozytools) — 프로젝트 안내

**더 코지 플로리스트** 유저를 위한 범용 계산기·도구 모음 사이트.
자세한 목표와 기획 배경은 같은 폴더의 `범용도구사이트-구상.md` 참고.

**이 프로젝트는 [guild-flower](../guild-flower) (길드 <백야> 길드전 계산기)와 완전히 별개의 프로젝트다.**
사이트 자체(`index.html`/`calc.html`/`calendar.html`/`register.html`)는 빌드 없는 순수 정적 웹페이지(HTML/CSS/JS)로, GitHub Pages로 배포한다.
guild-flower와 스택·리포지토리·배포 흐름 전부 다르다.

**단, 캘린더 일정 저장만은 예외** — [cozytools-worker](../cozytools-worker)라는 **cozytools 전용** Cloudflare Workers 백엔드가 따로 있다 (2026-08-18부터. 원래 Apps Script로 만들었던 [cozytools-backend](../cozytools-backend)는 너무 느려서 대체함 — 그 폴더는 폐기 상태로 보관만 함). guild-flower의 스크립트/시트와는 완전히 무관. 자세한 내용은 `cozytools-worker/CLAUDE.md` 참고.

---

## 이 폴더의 파일

| 파일 | 역할 |
|---|---|
| `index.html` | 허브 페이지 — 도구 목록 |
| `calc.html` | 길드전 임무 횟수 계산기 |
| `calendar.html` | 이벤트 캘린더 — `cozytools-worker`에서 fetch로 실시간 로드 |
| `register.html` | 일정 등록/삭제 (비밀번호 게이트). **사이트 어디에도 링크 안 걸어둠 — 주소를 아는 사람만 접근.** `index.html`에 카드 추가하지 말 것 |
| `범용도구사이트-구상.md` | 이 사이트의 목표·전략, 앞으로 만들 도구 아이디어 목록 |

모두 빌드 과정 없는 순수 HTML — 파일 열면 그대로 동작한다 (단, 로컬에서 열 때는 `file://`로 열지 말고 간단한 정적 서버로 띄워서 확인할 것. 브라우저에 따라 `file://`에서 스크립트 동작이 이상해질 수 있음. 예: `python -m http.server`).

---

## 배포

- **GitHub 리포지토리**: [saeunezip/cozytools](https://github.com/saeunezip/cozytools) — guild-flower가 쓰는 `cozy-flower`와는 별개.
- **배포 방식**: GitHub Pages, `main` 브랜치 `/ (root)`. 빌드 스텝 없음 — 리포지토리 루트가 곧 사이트 루트. `git push origin main`이면 반영됨 (재배포 절차 따로 없음 — guild-flower의 clasp/exec 방식과 다름).
- **도메인**: `cozytools.kro.kr` 연결 완료. 저장소 루트에 `CNAME` 파일로 관리됨 (GitHub Pages 커스텀 도메인 설정 시 자동 생성/갱신되니 직접 건드릴 일 거의 없음).
- **로컬 git 인증**: 이 리포지토리는 push 시 Git Credential Manager가 브라우저/모바일 앱 승인을 요구함 (사람이 한 번 승인해야 하는 단계 — Claude Code가 대신 못 함).

---

## calc.html — 임무 횟수 계산기

guild-flower의 `Calc.html`(Apps Script 템플릿, 길드 앱 안에서 `?page=calc`로 씀)을 **정적 페이지로 포팅**한 것.
로직(등급별 점수·확률·새로고침 기대비용 공식)은 완전히 동일하지만, 이 파일은 독립 사본이라 **한쪽을 고쳐도 다른 쪽엔 자동 반영 안 됨.** 게임 규칙이 바뀌면 두 파일 다 손봐야 할 수 있음.

설계 배경(공식 유도 과정 등)은 guild-flower 폴더의 `기능-임무횟수계산기.md`에 남아있음 — 참고용으로만, 이 프로젝트에서 직접 쓰는 파일은 아님.

---

## calendar.html / register.html — 실제로 저장됨

**2026-08-18부터 실사용 상태.** `calendar.html`은 더 이상 하드코딩 배열을 안 쓰고, `cozytools-worker`(Cloudflare Workers) `GET /`을 fetch해서 이벤트를 불러온다. `register.html`이 같은 백엔드에 `POST /`로 등록/삭제한다. (처음엔 Apps Script로 만들었다가 콜드스타트가 너무 느려서 같은 날 Cloudflare Workers로 옮김 — `cozytools-worker/CLAUDE.md` 참고.)

- 여러 날짜에 걸친 일정은 달력에서 이어진 막대로 표시됨 (주가 바뀌어도 같은 줄 유지)
- 단일/기간 일정, 매주 반복 일정(요일 범위, 예: 금~월) 둘 다 지원
- `register.html`은 비밀번호(Worker Secret에 저장, 코드엔 없음)로 게이트 — 링크를 아는 사람만 접근, 사이트 내 어디서도 링크 안 걸어둠
- 백엔드 API 주소(`API_URL`)는 `calendar.html`/`register.html` 양쪽에 하드코딩돼 있음 (`https://cozytools-backend.saeunezip.workers.dev`). Worker는 `wrangler deploy` 한 번이면 같은 주소로 갱신되니 이 두 파일을 손댈 일은 거의 없음

guild-flower 폴더의 `기능-이벤트캘린더.md`에는 guild-flower 자체에 Admin+시트로 통합하는 다른(더 예전) 설계가 남아있는데, **이 cozytools 버전으로 대체됐다고 보면 됨** — 굳이 guild-flower 쪽에 또 만들 필요 없음.

---

## 스타일 가이드

기존 두 페이지가 쓰는 톤 — 새 도구 추가할 때 맞출 것:

- 배경 `#f7f7f8`, 카드 흰 배경 + `border-radius: 14px` + 옅은 그림자
- 포인트 색 `#1f3b73`(버튼/제목), 진한 남색 `#152c57`(타이틀 텍스트)
- 보조 텍스트 `#90a4ae`
- 폰트 `"Malgun Gothic", sans-serif`
- 모바일(375~390px) 우선 — 다들 폰으로 봄. 최대 너비 `640px`로 가운데 정렬
- 각 도구 페이지 상단에 "도구 모음"으로 돌아가는 링크(`index.html`), 하단에 길드 크레딧 문구

---

## 아직 안 한 것

- `robots.txt` — 아직 없음(기본값: 전체 허용, 의도한 대로임 — 홍보 목적이라 검색엔진 차단 안 함). 단 `register.html`은 `<meta name="robots" content="noindex">`로 개별 차단해둠
- 카카오톡 등 링크 미리보기용 `og:image` — 아직 이미지 없음 (guild-flower의 `preview.png` 같은 것)
