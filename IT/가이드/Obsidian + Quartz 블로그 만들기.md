---
title: Obsidian + Quartz 블로그 만들기
date: 2025-11-30
tags:
  - it
  - 가이드
---
## 사전 준비

1. nodejs 설치
2. git 설치

---

## STEP 1: Obsidian 설치 및 설정

### 1-1. Obsidian 다운로드

https://obsidian.md/ 여기서 옵시디언 설치

### 1-2. Vault(저장소) 생성

1. Obsidian 실행
2. "Create new vault" 클릭
3. Vault name: `my-blog` 입력
4. Location: 원하는 위치 선택 (예: `Documents/my-blog`)
5. "Create" 클릭

### 1-3. 기본 설정

1. **Editor 탭:**
    - Readable line length: OFF (전체 너비 사용)
    - Strict line breaks: OFF
2. **Files & Links 탭:**
    - Default location for new attachments: `In subfolder under current folder`
    - Subfolder name: `attachments`
    - New link format: `Relative path to file` 선택
	    - `[양자역학 기초](물리학/양자역학-기초.md)` 이런식으로 링크 생김. 블로그의 링크 싱크 맞추기 위함.
    - Use [[Wikilinks]]: OFF (Markdown 링크 사용)
3. **Appearance 탭:**
    - Base color scheme: 원하는 테마 선택
    - 한글 폰트 설정 가능
    - 한글 설정 가능


---

## STEP 2: Quartz 설치

### 2-1. 터미널에서 작업 폴더로 이동

```bash
# Mac
cd ~/Documents

# Windows (PowerShell)
cd C:\Users\YourName\Documents
```

### 2-2. Quartz 다운로드

```bash
git clone https://github.com/jackyzha0/quartz.git
cd quartz
```

### 2-3. 의존성 설치

```bash
npm install
```

### 2-4. Quartz 초기화

```bash
npx quartz create
```

선택지 나오면 Empty Quartz, Treat links as relative paths 선택

---

## STEP 3: 첫 글 작성

### 3-1. Obsidian으로 돌아가기

Obsidian 화면에서 작업

### 3-2. 폴더 구조 만들기

좌측 파일 탐색기에서:

1. 우클릭 → "New folder" → `물리학`
2. 같은 방식으로 `수학`, `화학`, `IT` 폴더 생성

### 3-3. 홈페이지 작성 (필수!)

**중요:** 루트에 `index.md` 파일을 반드시 만들어야 합니다!

1. 루트에서 "New note" 클릭
2. 파일 이름: `index`
3. 내용 작성:

```markdown
### ㅎㅇㅎㅇ
공부하면서 배운 것들을 기록하고 정리하는 블로그입니다.
지식을 쌓아가는 여정을 공유합니다.

---

*이 블로그는 Obsidian과 Quartz로 만들어졌습니다.*
```

### 3-4. 첫 글 작성

1. `물리학` 폴더 우클릭 → "New note"
2. 파일 이름: `양자역학-기초`
3. 내용 작성:

````markdown
---
title: 양자역학 기초
date: 2025-11-30
tags:
  - 물리학
  - 양자역학
---

# 양자역학이란?

양자역학은 미시세계를 설명하는 물리학의 한 분야입니다.

## 핵심 개념

### 1. 파동-입자 이중성
빛과 물질은 파동과 입자의 성질을 동시에 가집니다.

### 2. 슈뢰딩거 방정식
시간에 따른 양자 상태의 변화를 기술합니다:

$$
i\hbar\frac{\partial\Psi}{\partial t} = \hat{H}\Psi
$$

여기서:
- $\Psi$: 파동함수
- $\hbar$: 플랑크 상수
- $\hat{H}$: 해밀토니안 연산자

## 코드 예시


import numpy as np

def wave_function(x, t):
    """간단한 파동함수"""
    k = 2 * np.pi
    omega = 1.0
    return np.exp(1j * (k*x - omega*t))

# 확률 밀도 계산
psi = wave_function(1.0, 0.0)
probability = np.abs(psi)**2
print(f"확률 밀도: {probability}")

## 참고 자료

- [[상대성이론]] - 관련 글 링크
- [[양자장론]]
````

### 3-5. 더 많은 글 작성 (선택)
같은 방식으로:
- `수학/미적분학.md`
- `화학/화학결합.md`
- `IT/Python-기초.md`

---

## STEP 4: Obsidian Vault를 GitHub에 올리기

### 4-1. GitHub Token 생성
1. GitHub 로그인 → Settings (우측 상단 프로필)
2. Developer settings (맨 아래) → Personal access tokens → Tokens (classic)
3. "Generate new token (classic)" 클릭
4. Note: `Quartz Trigger Token`
5. Expiration: `No expiration` (또는 원하는 기간)
6. 권한: **`repo` 전체 체크**
7. "Generate token" 클릭
8. **생성된 토큰을 복사해두기** (다시 볼 수 없음!)

### 4-2. Obsidian Vault 레포지토리 생성
1. GitHub에서 "New repository" 클릭
2. Repository name: `obsidian-vault`
3. Public 선택
4. "Create repository" 클릭

### 4-3. Obsidian Vault에 자동 트리거 설정
옵시디언 vault 폴더에서:

```bash
# .github/workflows 폴더 생성
mkdir -p .github/workflows
````

`.github/workflows/trigger-quartz.yml` 파일 생성:

```yaml
name: Trigger Quartz Rebuild

on:
  push:
    branches:
      - main

jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Quartz Deploy
        run: |
          curl -X POST \
            -H "Accept: application/vnd.github.v3+json" \
            -H "Authorization: token ${{ secrets.QUARTZ_TRIGGER_TOKEN }}" \
            https://api.github.com/repos/username/username.github.io/dispatches \
            -d '{"event_type":"content-update"}'
```

**주의:** `username/username.github.io` 부분을 본인의 GitHub 아이디와 블로그 레포지토리 이름으로 변경!

### 4-4. GitHub Secret 설정

1. `obsidian-vault` 레포지토리 페이지 이동
2. Settings → Secrets and variables → Actions
3. "New repository secret" 클릭
4. Name: `QUARTZ_TRIGGER_TOKEN`
5. Secret: 위에서 복사한 토큰 붙여넣기
6. "Add secret" 클릭

### 4-5. Git 초기화 및 푸시

Obsidian vault 폴더에서:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/obsidian-vault.git
git push -u origin main
```

**주의:** `username`을 본인의 GitHub 아이디로 변경!

---

## STEP 5: 로컬에서 미리보기

### 5-1. 터미널에서 Quartz 폴더로 이동

```bash
cd ~/Documents/quartz  # 또는 본인의 quartz 경로
```

### 5-2. 옵시디언 폴더를 Quartz에 링크 걸기

**Windows (관리자 권한 PowerShell):**

```powershell
# quartz 폴더에서 기존 content 폴더 이름 변경 (백업)
ren content content_backup

# 심볼릭 링크 생성
mklink /D content "C:\Users\YourName\Documents\my-blog"
```

**Mac/Linux:**

```bash
# 기존 content 폴더 백업
mv content content_backup

# 심볼릭 링크 생성
ln -s ~/Documents/my-blog content
```

### 5-3. 개발 서버 실행

```bash
npx quartz build --serve
```

### 5-4. 브라우저에서 확인

1. 브라우저 열기
2. 주소창에 `localhost:8080` 입력
3. 블로그 확인!

**서버 종료:** 터미널에서 `Ctrl + C`

---

## STEP 6: GitHub Pages에 자동 배포 설정

### 6-1. GitHub 블로그 레포지토리 생성

1. GitHub에서 "New repository" 클릭
2. Repository name: `username.github.io` (username을 본인 GitHub 아이디로 변경!)
3. Public 선택
4. **다른 것들 체크 안 함**
5. "Create repository" 클릭

### 6-2. Quartz 설정 파일 수정

**`quartz.config.ts` 파일 수정:**

```typescript
const config: QuartzConfig = {
  configuration: {
    pageTitle: "나의 학습 블로그",  // 블로그 제목
    enableSPA: true,
    enablePopovers: true,
    analytics: {
      provider: "plausible",  // 또는 null
    },
    locale: "ko-KR",  // 한국어 설정
    baseUrl: "username.github.io",  // ⚠️ 본인 GitHub 주소로 변경!
    ignorePatterns: ["private", "templates", ".obsidian"],
    defaultDateType: "created",
    theme: {
      fontOrigin: "googleFonts",
      cdnCaching: true,
      typography: {
        header: "Noto Sans KR",
        body: "Noto Sans KR",
        code: "JetBrains Mono",
      },
      colors: {
        lightMode: {
          light: "#faf8f8",
          lightgray: "#e5e5e5",
          gray: "#b8b8b8",
          darkgray: "#4e4e4e",
          dark: "#2b2b2b",
          secondary: "#284b63",
          tertiary: "#84a59d",
          highlight: "rgba(143, 159, 169, 0.15)",
        },
        darkMode: {
          light: "#161618",
          lightgray: "#393639",
          gray: "#646464",
          darkgray: "#d4d4d4",
          dark: "#ebebec",
          secondary: "#7b97aa",
          tertiary: "#84a59d",
          highlight: "rgba(143, 159, 169, 0.15)",
        },
      },
    },
  },
  // ... 나머지는 그대로
}
```

### 6-3. .nojekyll 파일 생성

Quartz 루트 폴더에서:

```bash
# Windows
echo. > .nojekyll

# Mac/Linux
touch .nojekyll
```

### 6-4. GitHub Actions 워크플로우 생성

`.github/workflows/deploy.yml` 파일 생성:

```yaml
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      - main
  # obsidian-vault 저장소가 업데이트되면 자동으로 빌드
  repository_dispatch:
    types: [content-update]
 
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
          git clone https://github.com/username/obsidian-vault.git temp-vault
          rm -rf content
          mkdir -p content
          cp -r temp-vault/* content/ 2>/dev/null || true
          cp -r temp-vault/.* content/ 2>/dev/null || true
          rm -rf temp-vault
          rm -rf content/.git
      
      - name: List content directory
        run: ls -la content/
      
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      
      - name: Install Dependencies
        run: npm ci
      
      - name: Build Quartz
        run: npx quartz build
      
      - name: List public directory contents
        run: ls -la public/
      
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

**주의:** `username/obsidian-vault` 부분을 본인의 GitHub 아이디로 변경!

### 6-5. Quartz 저장소 업로드

Quartz 폴더에서:

```bash
# 기존 .git 폴더가 있다면 삭제
rm -rf .git  # Windows: rmdir /s .git

# Git 초기화
git init
git add .
git commit -m "Initial Quartz setup"
git branch -M main
git remote add origin https://github.com/username/username.github.io.git
git push -u origin main
```

### 6-6. GitHub Pages 활성화

1. `username.github.io` 레포지토리 페이지 이동
2. Settings → Pages
3. Source: **GitHub Actions** 선택
4. Save

### 6-7. GitHub Actions 권한 설정

1. `username.github.io` 레포지토리 Settings
2. Actions → General
3. Workflow permissions
4. **Read and write permissions** 선택
5. Save

---

## STEP 7: 자동 배포 테스트

### 7-1. Obsidian에서 글 작성

Obsidian에서 새 글을 작성하거나 기존 글 수정

### 7-2. Git으로 푸시

Obsidian vault 폴더에서:

```bash
git add .
git commit -m "새 글 추가"
git push
```

### 7-3. 자동 배포 확인

1. `obsidian-vault` 레포지토리 → Actions 탭: "Trigger Quartz Rebuild" 워크플로우 실행 확인
2. `username.github.io` 레포지토리 → Actions 탭: "Deploy Quartz site" 워크플로우 실행 확인
3. 5분 정도 후 `https://username.github.io` 접속하여 변경사항 확인

**이제 Obsidian에서 글을 작성하고 푸시만 하면 자동으로 블로그가 업데이트됩니다!**

---

## 유용한 팁

### 수식 작성

```markdown
인라인: $E = mc^2$

블록:
$$
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
$$

여러 줄:
$$
\begin{aligned}
a &= b + c \\
d &= e + f
\end{aligned}
$$
```

### 글 간 링크

```markdown
[[다른 글 제목]]
[[다른 글 제목|표시할 텍스트]]
```

### Callout (강조 상자)

```markdown
> [!note] 노트
> 중요한 내용

> [!warning] 경고
> 주의 사항

> [!tip] 팁
> 유용한 정보
```

### 코드 블록

````markdown
```python title="example.py"
def hello():
    print("Hello, World!")
```
````

---

## 커스터마이징

### 색상 변경

`quartz/styles/custom.scss` 파일 생성:

```scss
:root {
  --primary: #5b8dec;
  --secondary: #7b97aa;
}

h1 {
  color: var(--primary);
  border-bottom: 2px solid var(--primary);
}

a {
  color: var(--primary);
}

a:hover {
  border-bottom: 1px solid var(--primary);
}
```

### 레이아웃 변경

`quartz.layout.ts` 수정:

```typescript
export const defaultContentPageLayout: PageLayout = {
  beforeBody: [
    Component.Breadcrumbs(),
    Component.ArticleTitle(),
    Component.ContentMeta(),
    Component.TagList(),
  ],
  left: [
    Component.PageTitle(),
    Component.Search(),
    Component.Darkmode(),
    Component.DesktopOnly(Component.Explorer()),
  ],
  right: [
    Component.Graph(),
    Component.DesktopOnly(Component.TableOfContents()),
    Component.Backlinks(),
  ],
}
```

---

## 문제 해결

### 탐색기에 content랑 목차 중복
![[attachments/스크린샷 2025-11-30 173618.png]]
quartz.config.ts 에 
   ignorePatterns: ["private", "templates", ".obsidian", "README.md", "CODE_OF_CONDUCT.md", "LICENSE", "content"], 
 
### 옵시디언 clone 실패시 
private -> public

### 블로그가 안 보이고 RSS 피드만 보일 때

**원인:** `content/index.md` 파일이 없음

**해결:**

1. Obsidian vault 루트에 `index.md` 파일 생성
2. 홈페이지 내용 작성
3. Git에 푸시

### Actions 실행이 안 될 때

1. `username.github.io` → Settings → Actions → General
2. Workflow permissions → **Read and write permissions** 확인
3. `obsidian-vault` → Settings → Secrets → `QUARTZ_TRIGGER_TOKEN` 확인

### 자동 트리거가 작동하지 않을 때

1. GitHub Token이 만료되지 않았는지 확인
2. `.github/workflows/trigger-quartz.yml`에서 레포지토리 이름이 정확한지 확인
3. `obsidian-vault`의 Actions 탭에서 워크플로우 실행 로그 확인

### 빌드 오류 시

```bash
# 의존성 재설치
cd quartz
rm -rf node_modules
rm package-lock.json
npm install
```

---

## 폴더 구조 (최종)

```
~/Documents/
├── my-blog/                  # Obsidian vault
│   ├── .github/
│   │   └── workflows/
│   │       └── trigger-quartz.yml
│   ├── index.md             # 홈페이지 (필수!)
│   ├── 물리학/
│   │   └── 양자역학-기초.md
│   ├── 수학/
│   ├── 화학/
│   └── IT/
│
└── quartz/                   # Quartz 프로젝트
    ├── .github/
    │   └── workflows/
    │       └── deploy.yml
    ├── content/             # → my-blog로 심볼릭 링크
    ├── public/              # 빌드 결과
    ├── quartz/              # Quartz 엔진
    ├── .nojekyll
    ├── package.json
    └── quartz.config.ts
```

---

## 핵심 명령어 요약

```bash
# 로컬 미리보기
cd quartz
npx quartz build --serve
# → http://localhost:8080

# Obsidian에서 글 작성 후
cd my-blog
git add .
git commit -m "새 글 작성"
git push
# → 자동으로 블로그 업데이트!
```

---

## 참고 링크

- **Quartz 공식 문서:** https://quartz.jzhao.xyz/
- **Obsidian 공식 사이트:** https://obsidian.md
- **GitHub Pages 문서:** https://docs.github.com/pages
- **KaTeX 수식 문법:** https://katex.org/docs/supported.html
- **Markdown 문법:** https://www.markdownguide.org/