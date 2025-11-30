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
3. **Appearance 탭:**
    - Base color scheme: 원하는 테마 선택
    - 한글 폰트 설정 가능
    - 한글 설정 가능
## STEP 2: Quartz 설치
### 2-1. 터미널에서 작업 폴더로 이동
```
# Mac
cd ~/Documents

# Windows (PowerShell)
cd C:\Users\YourName\Documents
```
### 2-2. Quartz 다운로드
```
git clone https://github.com/jackyzha0/quartz.git
cd quartz
```
### 2-3. 의존성 설치
```
npm install
```
### 2-4. Quartz 초기화
```
npx quartz create
```
선택지 나오면 Empty Quartz, Treat links as relative paths 선택

## STEP 3: 첫 글 작성
### 3-1. Obsidian으로 돌아가기

Obsidian 화면에서 작업

### 3-2. 폴더 구조 만들기

좌측 파일 탐색기에서:

1. 우클릭 → "New folder" → `물리학`
2. 같은 방식으로 `수학`, `화학`, `IT` 폴더 생성

### 3-3. 첫 글 작성

1. `물리학` 폴더 우클릭 → "New note"
2. 파일 이름: `양자역학-기초`
3. 내용 작성:
```
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

'''python
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
'''

## 참고 자료

- [[상대성이론]] - 관련 글 링크
- [[양자장론]]
```
### 3-4. 더 많은 글 작성 (선택)

같은 방식으로:
- `수학/미적분학.md`
- `화학/화학결합.md`
- `IT/Python-기초.md`

## STEP 4: 로컬에서 미리보기
### 4-1. 터미널에서 Quartz 폴더로 이동
```
cd ~/Documents/quartz # 또는 본인의 quartz 경로
```

### 4-2. 개발 서버 실행
옵시디언 폴더를 quartz 에 링크 걸기
```
# quartz 폴더에서 기존 content 폴더 이름 변경 (백업)
ren content content_backup

# quartz 폴더에서 심볼릭 링크 생성 (관리자권한) 
mklink /D content "옵시디언\폴더root\경로"
```

```
npx quartz build --serve
```

### 4-3. 브라우저에서 확인

1. 브라우저 열기
2. 주소창에 `localhost:8080` 입력
3. 블로그 확인!

**서버 종료:** 터미널에서 `Ctrl + C`

## STEP 5: GitHub에 배포
### 5-1. GitHub 레포지토리 생성

1. [https://github.com](https://github.com) 로그인
2. 우측 상단 `+` → "New repository"
3. 설정:
    - Repository name: `username.github.io`_(username을 본인 GitHub 아이디로 변경)_ 예: `gildong.github.io`
    - Public 선택
    - **다른 것들 체크 안 함** (README, .gitignore 등)
4. "Create repository" 클릭
### 5-2. GitHub 인증 설정
윈도우에서 SSH 키 생성 
```
# SSH 키 생성
ssh-keygen -t ed25519 -C "your_email@example.com"
# Enter 3번 누르기 (비밀번호 설정 안 함)

type C:\Users\YourName\.ssh\id_ed25519.pub | clip
```

**GitHub에 SSH 키 등록:**

1. GitHub → 우측 상단 프로필 → Settings
2. 좌측 "SSH and GPG keys" 클릭
3. type은 Authentication Key
4. "New SSH key" 클릭
5. Title: `My Computer`
6. Key: 복사한 내용 붙여넣기
7. "Add SSH key" 클릭
### 5-3. Quartz 배포 설정
#### root 에 .nojekyll 파일 생성 (actions 에러 막기 위함)
#### quartz.config.ts 파일 수정:
수정할 부분:
```
const config: QuartzConfig = {
  configuration: {
    pageTitle: "나의 학습 블로그",  // 블로그 제목
    enableSPA: true,
    enablePopovers: true,
    analytics: {
      provider: "plausible",  // 또는 null
    },
    locale: "ko-KR",  // 한국어 설정
    baseUrl: "username.github.io",  // 본인 GitHub 주소로 변경!
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

### 5-4. 배포 실행
```
# 1. 빌드 (quartz 폴더에서!)
cd quartz
npx quartz build

# 2. 실행 (내 리포로 origin을 바꿔줘야 함)
git remote remove origin
git remote add origin https://github.com/name/name.github.io.git
git push -u origin v4
npx quartz sync
```
### 5-5. GitHub Pages 활성화
1. GitHub 레포지토리 페이지 이동
2. "Settings" 탭 클릭
3. 좌측 "Pages" 클릭
4. Source: `Deploy from a branch` 선택
5. Branch: gh-pages 선택, `/ (root)` 선택
6. "Save" 클릭

### 5-6. GitHub Actions 활성화

Quartz v4는 GitHub Actions가 대신 빌드해서 배포 브랜치(`gh-pages`)를 만들어줌. 이를 위해 권한을 열어줘야 함.

1. GitHub 저장소 페이지로 이동.
    
2. 상단 탭의 **Settings** 클릭.
    
3. 좌측 사이드바에서 **Actions** -> **General** 클릭.
    
4. 스크롤을 내려 **Workflow permissions** 섹션을 찾기.
    
5. **Read and write permissions**를 선택하고 **Save**를 누르기.

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

### Jekyll 오류 발생 시

```bash
# .nojekyll 파일이 있는지 확인
cd \blog\quartz\public
dir .nojekyll

# 없으면 생성
echo. > .nojekyll
git add .nojekyll
git commit -m "Add .nojekyll"
git push
```

### 파일이 안 보일 때

```bash
# content 폴더 확인
cd \blog\quartz
dir content\*.md /s /b

# index.md가 있는지 확인 (필수!)
dir content\index.md
```

### 빌드 오류 시

````bash
# 의존성 재설치
cd \blog\quartz
rmdir /s node_modules
del package-lock.json
npm install
```

---

## 11. 폴더 구조 (최종)
```
D:blog\
├── quartz\                    # Quartz 소스 코드
│   ├── content\              # Obsidian vault (글 작성)
│   │   ├── index.md         # 홈페이지 (필수!)
│   │   ├── 물리학\
│   │   │   └── 양자역학-기초.md
│   │   ├── 수학\
│   │   ├── 화학\
│   │   └── IT\
│   ├── public\               # 빌드 결과 (배포용)
│   │   ├── .nojekyll        # Jekyll 비활성화
│   │   ├── index.html
│   │   └── ...
│   ├── quartz\               # Quartz 엔진
│   ├── package.json
│   └── quartz.config.ts      # 설정 파일
└── my-blog\                  # (삭제 가능, content로 통합됨)
````

---

## 핵심 명령어 요약


```bash
# 로컬 미리보기
cd \blog\quartz
npx quartz build --serve
# → http://localhost:8080

# 배포
cd \blog\quartz
npx quartz build
npx quartz sync
```

---

## 참고 링크

- **Quartz 공식 문서:** [https://quartz.jzhao.xyz/](https://quartz.jzhao.xyz/)
- **Obsidian 공식 사이트:** [https://obsidian.md](https://obsidian.md)
- **GitHub Pages 문서:** [https://docs.github.com/pages](https://docs.github.com/pages)
- **KaTeX 수식 문법:** [https://katex.org/docs/supported.html](https://katex.org/docs/supported.html)
- **Markdown 문법:** [https://www.markdownguide.org/](https://www.markdownguide.org/)


