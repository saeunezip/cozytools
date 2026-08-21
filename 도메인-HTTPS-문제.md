# cozytools.kro.kr HTTPS 인증서가 발급되지 않는 이유

조사일: 2026-08-21

## 증상

`https://cozytools.kro.kr` 접속 시 브라우저가 "안전하지 않음" 경고를 띄운다.
며칠을 기다려도 GitHub Pages 가 인증서를 발급하지 못한다.

## 확인한 것 (설정은 전부 정상)

| 항목 | 결과 |
|---|---|
| DNS | `cozytools.kro.kr` → CNAME `saeunezip.github.io` → GitHub Pages IP (185.199.108~111.153) — 정상 |
| 와일드카드 DNS | 없음 (`*.kro.kr` 안 잡힘) — GitHub 이 싫어하는 조건 아님 |
| CAA 레코드 | CNAME 을 따라가 `issue "letsencrypt.org"` 포함 — 차단 아님 |
| CNAME 파일 | 저장소에 `cozytools.kro.kr` 로 정상 존재 |
| 실제 TLS | `certificate is not valid for 'cozytools.kro.kr'` — GitHub 이 기본 인증서를 내보내는 중 = **인증서가 아예 발급된 적 없음** |

즉 우리 쪽 설정 문제가 아니다.

### 정정: `saeunezip.github.io/cozytools` 도 지금은 안전하지 않다

저장소에 `CNAME` 파일이 있으면 GitHub Pages 는 `.github.io` 주소를 커스텀 도메인으로 **302 리다이렉트**시킨다.
실제로 `https://saeunezip.github.io/cozytools/` → `http://cozytools.kro.kr/` 로 넘어간다 (https 도 아닌 **http**).
**현재 이 사이트에는 정상 동작하는 HTTPS 주소가 하나도 없다.**

## 진짜 원인: kro.kr 이 Public Suffix List 에 없다

- Let's Encrypt 는 **Public Suffix List(PSL)** 로 "등록 도메인" 을 판정하고,
  **등록 도메인당 7일에 인증서 50장** 이라는 전역 제한을 건다.
  (전 세계 모든 계정이 합쳐서 세는 값이다)
- `.kr` 은 PSL 에 있지만 **`kro.kr` 은 없다.** (PSL `.kr` 항목 31개 중 사설 등재는 `blogspot.kr` 뿐)
- 그래서 Let's Encrypt 입장에서 `cozytools.kro.kr` 의 등록 도메인은 **`kro.kr`** 이고,
  전 세계 kro.kr 하위 도메인 전부가 **주당 50장짜리 양동이 하나를 나눠 쓴다.**
- kro.kr 은 "내도메인.한국" 이라는 한국 최대급 무료 도메인 서비스라 그 양동이가 늘 비어 있다.
  → GitHub 이 요청해도 계속 거절당하고, 며칠을 기다려도 안 된다.

참고로 `github.io`, `pages.dev`, `netlify.app`, `vercel.app` 는 전부 PSL 에 등재돼 있어서
하위 도메인마다 자기 몫의 제한을 받는다. 그래서 그쪽은 HTTPS 가 즉시 붙는다.

같은 원리로 GitLab 도 `nip.io` 에서 똑같은 문제를 겪었다:
https://gitlab.com/gitlab-org/gitlab/-/issues/28056

## 우리가 할 수 있는 것이 아니다

CNAME 파일을 지웠다 다시 만드는 재시도는 도움이 안 된다 (저장소 이력상 이미 3번 했다).
kro.kr 운영자가 PSL 등재를 신청하지 않는 한 해결되지 않는다.

## 선택지

1. **Cloudflare Pages 로 옮기기** (무료, 확실함)
   - `pages.dev` 는 PSL 등재 → HTTPS 즉시 발급
   - 같은 GitHub 저장소를 연결하면 push 할 때마다 자동 배포
   - 주소: `cozytools.pages.dev`
2. **실제 도메인 구입** (연 1~2만원)
   - GitHub Pages 그대로 쓰면서 HTTPS 정상 동작
   - 커뮤니티 홍보용이면 kro.kr 보다 신뢰감도 낫다
3. **당장 급하면** `https://saeunezip.github.io/cozytools/` — 지금도 HTTPS 정상

## 근거 자료

- Let's Encrypt Rate Limits: https://letsencrypt.org/docs/rate-limits/
- Public Suffix List: https://publicsuffix.org/list/
- GitHub Pages 커스텀 도메인 문제 해결: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages


---

## 결정 (2026-08-21): Cloudflare Pages 로 이전

`pages.dev` 는 PSL 에 등재돼 있어 HTTPS 가 확실히 붙는다. 이미 Cloudflare 계정
(`kyunga72765@gmail.com`)으로 `cozytools-backend` Worker 를 쓰고 있으므로 계정도 그대로 쓴다.

### 진행 순서

1. **Pages 프로젝트 생성** — Workers & Pages → Create → Pages → Connect to Git → `saeunezip/cozytools`
   - Production branch: `main`
   - Framework preset: `None`
   - Build command: `mkdir -p dist && cp *.html dist/`
   - Build output directory: `dist`
   - 빌드 명령을 쓰는 이유: 저장소 최상위의 `.md` 문서(`CLAUDE.md` 등)가 그대로 공개되는 걸 막는다.
     지금 GitHub Pages 는 이걸 그대로 서빙하고 있다.
2. **`cozytools.pages.dev` 로 HTTPS 확인**
3. **`cozytools.kro.kr` 살리기 시도** (안 되면 포기해도 됨)
   - Pages → Custom domains → `cozytools.kro.kr` 등록
   - kro.kr 관리 페이지에서 CNAME 을 `saeunezip.github.io` → `cozytools.pages.dev` 로 변경
   - **될 가능성이 있는 이유**: Cloudflare 는 Let's Encrypt 외에 Google Trust Services / SSL.com 도
     쓴다. GTS 는 PSL 기반 50장/주 제한을 쓰지 않으므로 kro.kr 병목을 피할 수 있다.
     (보장은 아님 — 안 되면 `pages.dev` 로 간다)
4. **GitHub Pages 정리** — Cloudflare 가 뜬 뒤에 `CNAME` 파일 삭제 + Pages 설정에서 custom domain 제거.
   그래야 `saeunezip.github.io/cozytools` 가 리다이렉트 없이 예비 주소로 쓸 수 있다.
   **순서 주의: Cloudflare 가 정상 동작하기 전에 지우면 안 된다.**
5. **og:url 갱신** — 최종 주소가 정해지면 `index.html` / `calc.html` / `calendar.html` 12번째 줄의
   `og:url` 을 바꾼다. 카카오톡·디스코드 링크 미리보기에 쓰이는 값이라 커뮤니티 공유 시 중요하다.
   내부 링크는 전부 상대경로(`href="calc.html"`)라 도메인이 바뀌어도 그대로 동작한다.
