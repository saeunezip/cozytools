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
