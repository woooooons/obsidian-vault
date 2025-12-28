---
title: Obsidian + Quartz 블로그 Google Analytics + Google Search 등록
date: 2025-11-30
tags:
  - it
  - 가이드
---
# Google Analytics

## STEP 1: 구글 계정 설정
https://marketingplatform.google.com/about/analytics/

[참고 블로그](https://hel-p.tistory.com/57#%EA%B5%AC%EA%B8%80%20%EA%B3%84%EC%A0%95%20%EB%93%B1%EB%A1%9D-1)

참고블로그 따라서 등록하기

## STEP 2: 사이트 등록하기 (Quartz)

Quartz 폴더의 quartz.config.ts 수정

```
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
  .....,
      analytics: QUARTZ_GOOGLE_TAG_ID ? { provider: "google", tagId: QUARTZ_GOOGLE_TAG_ID } : null,
      baseUrl: QUARTZ_BASE_URL,
  }

```

quartz/bootstrap-cli.mjs
```
import "dotenv/config"
```

package.json‎ (install 해서 lock 과 맞추기)
```
"dotenv": "^16.4.7",
```

‎.github/workflows/deploy.yml
```
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - main
  repository_dispatch:
    types: [content-update]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # obsidian-vault 저장소에서 content 가져오기
      - name: Clone Obsidian Vault
        run: |
          git clone https://github.com/woooooons/obsidian-vault.git temp-vault
          rm -rf content
          mkdir -p content

          # .git, .github, node_modules 등 제외하고 복사
          rsync -av --exclude='.git' --exclude='.github' --exclude='node_modules' --exclude='.obsidian' temp-vault/ content/

          rm -rf temp-vault

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install Dependencies
        run: npm ci

      - name: Build Quartz
        run: npx quartz build
        env:
          # Repo Settings → Secrets and variables → Actions 에 등록
          QUARTZ_BASE_URL: ${{ vars.QUARTZ_BASE_URL }}
          QUARTZ_GOOGLE_TAG_ID: ${{ secrets.QUARTZ_GOOGLE_TAG_ID }}
          QUARTZ_GOOGLE_SITE_VERIFICATION: ${{ vars.QUARTZ_GOOGLE_SITE_VERIFICATION }}

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

위처럼 하고 add, commit, push

## STEP 3: 사이트 등록하기 (Github)

repo → Settings → Secrets and variables → Actions

- Secrets 탭
- New repository secret 클릭
- Name: QUARTZ_GOOGLE_TAG_ID
- Secret: GA4 측정 ID (예: G-XXXXXXXXXX)

- Variables 탭
- New repository variable 클릭
- Name: QUARTZ_BASE_URL
- Value: woooooons.github.io (또는 프로젝트 페이지면 woooooons.github.io/<repo>)




## STEP 4: 확인

![[Pasted image 20251228151637.png]]