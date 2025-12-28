---
title: Google Analytics + Google Search 등록
date: 2025-12-28
tags:
  - it
  - 가이드
---
# Google Analytics

## STEP 1: 구글 계정/GA4 속성 생성
https://marketingplatform.google.com/about/analytics/
https://search.google.com/search-console/performance/

[Github blog를 Google 검색 엔진에 노출시키기](https://burningfalls.github.io/blog/google-search-console/)
[Google Analytics 등록하는법](https://hel-p.tistory.com/57#%EA%B5%AC%EA%B8%80%20%EA%B3%84%EC%A0%95%20%EB%93%B1%EB%A1%9D-1)

- Google Analytics 등록하는법 따라서 GA4에서 속성/데이터 스트림 만들고 **Measurement ID**를 받기. (예: `G-XXXXXXXXXX`)

## STEP 2: Quartz 코드에 GA 설정(환경변수 기반)
Quartz는 `quartz.config.ts`에서 `process.env`를 읽어 **GA를 켜거나 끄게** 구성.

```1:30:quartz.config.ts
import { QuartzConfig } from "./quartz/cfg"
import * as Plugin from "./quartz/plugins"

const env = (key: string) => {
  const v = process.env[key]
  if (typeof v !== "string") return undefined
  const t = v.trim()
  return t.length > 0 ? t : undefined
}

// Keep sensitive values out of git by setting them via environment variables.
// Local: put them in `.env` (ignored by git). GitHub Actions: set Repo Secrets/Variables.
const QUARTZ_BASE_URL = env("QUARTZ_BASE_URL") ?? "woooooons.github.io"
const QUARTZ_GOOGLE_TAG_ID = env("QUARTZ_GOOGLE_TAG_ID")

const config: QuartzConfig = {
  configuration: {
    pageTitle: "Woooooon's Blog",
    pageTitleSuffix: "",
    enableSPA: true,
    enablePopovers: true,
    analytics: QUARTZ_GOOGLE_TAG_ID ? { provider: "google", tagId: QUARTZ_GOOGLE_TAG_ID } : null,
    locale: "ko-KR",
    baseUrl: QUARTZ_BASE_URL,
```

## STEP 3: 로컬 `.env` 자동 로딩(doten v)
로컬에서 `.env`를 자동으로 읽도록 CLI 진입점에 `dotenv/config`를 import 해둔 상태.

```1:6:quartz/bootstrap-cli.mjs
#!/usr/bin/env -S node --no-deprecation
import "dotenv/config"
import yargs from "yargs"
import { hideBin } from "yargs/helpers"
import {
  handleBuild,
```

그리고 `dotenv` 의존성이 들어가 있음.

```37:48:package.json
  "dependencies": {
    "@clack/prompts": "^0.11.0",
    "@floating-ui/dom": "^1.7.4",
    // ...
    "chokidar": "^4.0.3",
    "dotenv": "^16.4.7",
    "cli-spinner": "^0.2.10",
    "d3": "^7.9.0",
```

> 로컬에서 `.env` 파일은 커밋되지 않게 `.gitignore`에 포함되어 있음.

```1:6:.gitignore
.DS_Store
.gitignore
.env
.env.*
node_modules
public
```

---

## Google Search (Search Console)

## STEP 4: 소유권 확인(HTML 태그) — 코드 적용 방식
Search Console에서 제공하는:

```html
<meta name="google-site-verification" content="...토큰..." />
```

을 Quartz의 `Head` 컴포넌트에서 **환경변수로 주입**.

```15:50:quartz/components/Head.tsx
  }: QuartzComponentProps) => {
    // Google Search Console HTML 태그 검증용 (보안상 민감정보 아님)
    // Local: `.env`에 QUARTZ_GOOGLE_SITE_VERIFICATION 추가
    // GitHub Actions: Repo Settings → Secrets/Variables에 등록
    const googleSiteVerification = process.env.QUARTZ_GOOGLE_SITE_VERIFICATION?.trim()

    // ...

    return (
      <head>
        <title>{title}</title>
        <meta charSet="utf-8" />
        {googleSiteVerification && (
          <meta name="google-site-verification" content={googleSiteVerification} />
        )}
```

## STEP 5: GitHub Actions 배포에서 환경변수 주입(실제 워크플로우)
`npx quartz build` 실행 시점에 env를 주입.

```1:54:.github/workflows/deploy.yml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - main
  repository_dispatch:
    types: [content-update]
  workflow_dispatch:

# ...

      - name: Build Quartz
        run: npx quartz build
        env:
          # Repo Settings → Secrets and variables → Actions 에 등록
          QUARTZ_BASE_URL: ${{ vars.QUARTZ_BASE_URL }}
          QUARTZ_GOOGLE_TAG_ID: ${{ secrets.QUARTZ_GOOGLE_TAG_ID }}
          QUARTZ_GOOGLE_SITE_VERIFICATION: ${{ vars.QUARTZ_GOOGLE_SITE_VERIFICATION }}
```

## STEP 6: GitHub에 값 등록(Secrets/Variables)
repo → **Settings → Secrets and variables → Actions**
- **Secrets**
  - `QUARTZ_GOOGLE_TAG_ID` = `G-...`
- **Variables**
  - `QUARTZ_BASE_URL` = `woooooons.github.io`
  - `QUARTZ_GOOGLE_SITE_VERIFICATION` = Search Console 메타태그의 `content` 값

---

# Sitemap / robots / feed (SEO 크롤링용)

## STEP 7: sitemap.xml 자동 생성(Quartz ContentIndex emitter)
Quartz의 `ContentIndex` 플러그인이 `sitemap.xml`을 생성. 그리고 Search Console 파서가 까다롭게 보는 문자를 피하려고 **경로 세그먼트 단위로 인코딩**.

```42:64:quartz/plugins/emitters/contentIndex.tsx
function generateSiteMap(cfg: GlobalConfiguration, idx: ContentIndexMap): string {
  const base = cfg.baseUrl ?? ""
  // Encode each path segment to avoid leaving reserved characters (e.g. (), +, ,) unescaped.
  // Some sitemap consumers are stricter than browsers.
  const encodeSitemapSlug = (slug: SimpleSlug) => {
    if (slug === "/" || slug === "") return ""
    const trimmed = slug.replace(/^\/+/, "")
    return trimmed
      .split("/")
      .filter((s) => s.length > 0)
      .map((segment) => encodeURIComponent(segment))
      .join("/")
  }

  const createURLEntry = (slug: SimpleSlug, content: ContentDetails): string => `<url>
    <loc>https://${joinSegments(base, encodeSitemapSlug(slug))}</loc>
    ${content.date && `<lastmod>${content.date.toISOString()}</lastmod>`}
  </url>`
  // ...
  return `<?xml version="1.0" encoding="UTF-8"?>\n<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">${urls}</urlset>`
}
```

## STEP 8: robots.txt 자동 생성(루트 `/robots.txt`)
`robots.txt`는 Quartz 기본 기능이 아니라, emitter를 추가해서 **루트에 생성**.

```1:28:quartz/plugins/emitters/robots.ts
function generateRobotsTxt(baseUrl?: string) {
  const lines = ["User-agent: *", "Allow: /"]

  if (baseUrl) {
    lines.push(`Sitemap: https://${baseUrl}/sitemap.xml`)
  }

  return lines.join("\n") + "\n"
}

export const Robots: QuartzEmitterPlugin = () => ({
  name: "Robots",
  async emit(ctx) {
    const content = generateRobotsTxt(ctx.cfg.configuration.baseUrl)
    const path = await write({
      ctx,
      content,
      slug: "robots" as FullSlug,
      ext: ".txt",
    })
```

이 emitter는 플러그인으로 export되고:

```1:13:quartz/plugins/emitters/index.ts
export { ContentIndex as ContentIndex } from "./contentIndex"
export { AliasRedirects } from "./aliases"
export { Assets } from "./assets"
export { Static } from "./static"
export { Favicon } from "./favicon"
export { ComponentResources } from "./componentResources"
export { NotFoundPage } from "./404"
export { CNAME } from "./cname"
export { CustomOgImages } from "./ogImage"
export { Robots } from "./robots"
```

`quartz.config.ts`의 emitters에 포함되어 실행됨.

```87:104:quartz.config.ts
    emitters: [
      Plugin.AliasRedirects(),
      Plugin.ComponentResources(),
      Plugin.ContentPage(),
      Plugin.FolderPage(),
      Plugin.TagPage(),
      Plugin.ContentIndex({
        enableSiteMap: true,
        enableRSS: true,
      }),
      Plugin.Robots(),
      Plugin.Assets(),
      Plugin.Static(),
      Plugin.Favicon(),
      Plugin.NotFoundPage(),
      Plugin.CustomOgImages(),
    ],
```

## STEP 9: 색인 생성 요청(가속)

Sitemap 제출만으로도 결국 크롤링되지만, 초기에 빠르게 잡히려면 아래를 권장.

1) Search Console → URL 검사

2) 아래 URL들로 각각 실시간 테스트 → 색인 생성 요청

- 홈: https://woooooons.github.io/

- 대표 글 3~10개 (최신글/핵심글 위주)

> 모든 글을 하나하나 할 필요는 없고, 홈 + 핵심 글 몇 개만 해도 충분.

---

## STEP 10: 정상/지연 기준(체크리스트)

- 정상(기술적으로 OK) 기준

- URL 검사 “실시간 테스트”에서 가져오기 성공, 크롤링 허용: 예

- 브라우저에서 sitemap.xml이 열림

- 지연(기다리면 되는 케이스)

- Sitemaps에 “읽을 수 없음”이 수시간~수일 남아있을 수 있음

- 색인 반영은 보통 수시간~수일, 길면 1~2주도 가능

---

## STEP 11: 트러블슈팅

- Sitemaps에 계속 “읽을 수 없음”만 뜰 때

- 기존 제출 항목 삭제 → sitemap.xml로 재제출

- 1~6시간 후 다시 확인 (Search Console 배치 처리 지연이 흔함)

- URL 검사에서 실시간 테스트가 실패할 때

- “가져오기 실패 이유”(차단/리다이렉트/404/5xx)를 확인해야 함

- 광고차단/네트워크 차단이 있을 수 있으니 다른 네트워크에서도 테스트

- sitemap.xml을 ‘색인’시키려고 할 때

- 사이트맵은 검색 노출 대상 페이지가 아니라 데이터 파일이라 URL 검사에서 “미등록”처럼 보여도 정상.

- 색인 요청은 홈/글 URL에 해야 의미가 있음.


## STEP 12: 확인

아직 Google Analytics 만 확인가능.

![[Pasted image 20251228151637.png]]

