# Hugo + Stack 테마 + GitHub Pages(CI/CD) + AdSense + SEO 플랜

대상 레포: `https://github.com/LUKE-hyungjin/blog`  
배포 방식: **GitHub Pages + GitHub Actions (push 할 때마다 자동 배포)**  
첫 포스트: `Titanic_medium_en.md`

---

## 최종 목표

- **Hugo**로 정적 블로그 생성
- **Stack 테마** 적용
- `main` 브랜치에 **push** 될 때마다 **자동 빌드/배포**
- 첫 글로 `Titanic_medium_en.md` 게시
- **Google AdSense** 통합(광고 스크립트 삽입 + 정책 체크)
- 마지막으로 **SEO 최적화**(사이트맵/robots/canonical/OG/Twitter/구조화 데이터/성능/콘텐츠 규칙)

---

## 사전 체크(필수)

- GitHub Pages:
  - Repo → **Settings → Pages → Source: GitHub Actions**
- Pages URL(프로젝트 페이지):
  - `baseURL = "https://luke-hyungjin.github.io/blog/"`
- 로컬 환경:
  - Hugo **Extended** 설치(테마/SCSS 빌드 이슈 방지)

---

## 전체 작업 흐름(상세 Pseudocode)

```text
INPUTS:
  repo_url = "https://github.com/LUKE-hyungjin/blog"
  branch = "main"
  pages_base_url = "https://luke-hyungjin.github.io/blog/"
  first_post_source = "./Titanic_medium_en.md"

OUTPUTS:
  - Hugo 사이트 구조(루트)
  - Stack 테마 적용
  - GitHub Actions workflow: push → build → deploy to GitHub Pages
  - 첫 글: content/post/.../index.md
  - AdSense 삽입 포인트(head) + 운영 체크리스트
  - SEO 설정(hugo.toml + 콘텐츠 가이드)

PLAN:
  1) Hugo 스캐폴딩 생성 ✅ (완료)
     - run: hugo new site . --format toml
     - ensure: hugo.toml / content/ / layouts/ / static/ 생성됨

  2) Stack 테마 연결 ✅ (완료)
     - note: 로컬 환경에서 Git TLS 인증서 경로 이슈가 있으면 Modules 대신 themes/ 방식으로 설치해도 됨
     - run: hugo mod init github.com/LUKE-hyungjin/blog
     - configure: hugo.toml module.imports에 Stack module path 추가
     - run: hugo mod tidy
     - verify: hugo server로 테마가 정상 적용되는지 확인

  3) 사이트 기본 설정(운영 최소치) ✅ (완료)
     - set: baseURL = pages_base_url
     - set: languageCode, title, timeZone = "Asia/Seoul"
     - set: pagination, summaryLength, outputs(RSS 등)
     - set: enableRobotsTXT = true

  4) 첫 포스트로 Titanic_medium_en.md 등록 ✅ (완료)
     - create: content/post/titanic-foundry-pipeline/index.md
     - add: front matter(title/date/draft=false/slug/tags/categories)
     - copy: first_post_source 내용을 본문에 삽입
     - note: "Pasted image ..." 참조는 실제 이미지 파일로 교체 필요
            (해당 이미지가 있으면 같은 폴더에 저장 후 상대경로로 연결)

  5) GitHub Actions로 CI/CD 구성 (push마다 자동 배포)
     - create: .github/workflows/deploy.yml
     - on: push to main, workflow_dispatch
     - steps:
         - checkout
         - setup Hugo Extended
         - build: hugo --minify
         - upload-pages-artifact (public/)
         - deploy-pages
     - ensure: permissions pages:write, id-token:write
     - status: ✅ (완료)

  6) Google AdSense 통합(승인 전/후 모두 고려)
     - prerequisite:
         - AdSense 가입/승인
         - 사이트 소유권 확인(AdSense 요구사항)
     - integration approach:
         - Stack 테마의 head에 스크립트 삽입할 "오버라이드 partial" 생성
           (예: layouts/partials/head/custom.html)
         - head에 async script 삽입:
             <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXX"
                     crossorigin="anonymous"></script>
         - client id(ca-pub-XXXX)는 hugo.toml params로 분리해서 관리
     - policy checklist:
         - Privacy Policy / Terms / Contact 페이지 준비
         - 콘텐츠 품질(얇은 글/중복/자동 생성 콘텐츠 최소화)
         - 광고 과다 배치 금지

  7) SEO 최적화(기술 설정 + 콘텐츠 규칙)
     - technical:
         - sitemap 생성(enable + 설정)
         - robots.txt 활성화(enableRobotsTXT)
         - canonical URL 안정화(baseURL 일관성)
         - Open Graph / Twitter Card 메타 확인(테마 제공 여부 점검)
         - JSON-LD 구조화 데이터(필요 시 partial로 추가)
         - RSS/atom 활성화
     - content:
         - 글마다 title/description/tags/categories/cover image/alt text
         - H1은 1개, H2/H3 구조 유지
         - 내부 링크(관련 글)와 외부 링크(신뢰 출처)
         - 이미지 용량 최적화(webp 권장) + lazy-loading(테마 기능 확인)
     - performance:
         - minify on
         - 불필요한 JS/CSS 최소화(테마 기본 유지)
         - Lighthouse/Pagespeed로 재검증
     - status: ✅ (완료 - 기술 설정)
```

---

## 파일/디렉토리 설계(권장)

```text
/
  hugo.toml
  content/
    post/
      titanic-foundry-pipeline/
        index.md
        (images...)
  layouts/
    partials/
      head/
        custom.html        # AdSense/SEO 메타(추가 필요 시)
  static/
    robots.txt            # (기본 enableRobotsTXT로 자동 생성도 가능)
  .github/
    workflows/
      deploy.yml
```

---

## 첫 글 구성(권장 Front Matter)

- `Titanic_medium_en.md`를 아래 위치로 옮겨 게시:
  - `content/post/titanic-foundry-pipeline/index.md`
- Front matter 예시(실제 구현 시 날짜/슬러그 확정):

```toml
+++
title = "Titanic 데이터 전처리: Foundry Pipeline Builder로 결측치 처리 & 피처 엔지니어링"
date = 2026-01-03T00:00:00+09:00
draft = false
slug = "titanic-foundry-pipeline-builder"
tags = ["Foundry", "Titanic", "Data", "No-code"]
categories = ["Data"]
+++
```

---

## AdSense 통합 상세(구현 포인트)

- **삽입 위치**: `<head>` (전체 페이지 공통)
- **추천 방식**: 테마를 직접 수정하지 않고 **`layouts/partials/...` 오버라이드**
- **환경 분리(권장)**:
  - `hugo.toml`에 `params.adsenseClient = "ca-pub-XXXX"` 같이 저장
  - partial에서는 params 존재할 때만 스크립트 렌더(early return 개념)

---

## SEO 최적화 체크리스트(완료 기준)

- **인덱싱**
  - `sitemap.xml` 생성됨
  - `robots.txt` 접근 가능
  - canonical이 Pages URL 기준으로 일관됨
- **미리보기**
  - OG/Twitter 카드가 정상(타이틀/설명/이미지)
- **콘텐츠 품질**
  - 각 포스트에 title/description(요약)/태그/대표 이미지(가능 시)
  - 이미지에 대체텍스트(alt) 포함
- **기술**
  - RSS 제공
  - 빌드 결과가 `public/`에 생성되고 Actions가 Pages에 배포
- **검증**
  - Lighthouse/Pagespeed로 Core Web Vitals 기본 점검

---

## 다음 액션(진행 순서)

- [x] `PLAN.md` 기준으로 Hugo 프로젝트 생성 + Stack 테마 연결  
- [x] 사이트 기본 설정(운영 최소치) 적용  
- [x] `Titanic_medium_en.md`를 첫 포스트로 변환/이동  
- [x] GitHub Actions 배포 파이프라인 추가(푸시마다 자동 배포)  
- [ ] AdSense head 삽입 + 정책 페이지(Privacy/Contact) 추가  
- [x] SEO 설정/메타/사이트맵/robots/OG/구조화데이터까지 마무리


