+++

title = "Windows에서 Zola + GitHub Pages로 블로그 만들기"

date = 2026-01-02

\[taxonomies]

tags = \["rust", "zola", "github-pages", "blog", "tutorial"]

+++



Rust로 만든 정적 사이트 생성기 Zola를 사용해서 GitHub Pages 블로그를 만들어봤다.



\## 왜 Zola인가?



\- \*\*빠름\*\*: Rust로 작성되어 빌드 속도가 매우 빠르다

\- \*\*간단함\*\*: Hugo나 Jekyll보다 설정이 심플하다

\- \*\*GitHub 언어 통계\*\*: Zola 테마를 사용하면 레포에 Rust 비율이 나온다



\## 설치 과정



\### 1. Zola 설치 (Windows)



GitHub 릴리즈에서 직접 다운로드하는 방식을 사용했다.

```powershell

\# 1. https://github.com/getzola/zola/releases/latest 에서

\# zola-v0.21.0-x86\_64-pc-windows-msvc.zip 다운로드



\# 2. C:\\zola 폴더에 압축 해제



\# 3. 환경 변수 Path에 C:\\zola 추가

```



\### 2. 블로그 프로젝트 생성

```powershell

cd C:\\Users\\player\\Documents

zola init myblog



\# 질문에 답변:

\# URL: https://yourusername.github.io/blog

\# Sass: Y

\# Syntax highlighting: y

\# Search index: y

```



\### 3. 테마 설치



terminimal 테마를 선택했다. 터미널 스타일이 깔끔하고 마음에 들었다.

```powershell

cd myblog

git init

git submodule add https://github.com/pawroman/zola-theme-terminimal.git themes/terminimal

```



\### 4. 설정 파일 수정



`config.toml`:

```toml

base\_url = "https://changki123.github.io/blog"

title = "changki123's blog"

description = "Infrastructure Engineer's Tech Blog"

compile\_sass = true

build\_search\_index = true



theme = "terminimal"



\[markdown]

highlight\_code = true



\[extra]

accent\_color = "green"

logo\_text = "changki123"

```



\### 5. 로컬 테스트

```powershell

zola serve

\# http://127.0.0.1:1111 접속

```



완벽하게 작동했다!



\## GitHub Pages 배포



\### 1. GitHub 레포 생성



\- Repository name: `blog`

\- Public으로 생성



\### 2. Git 설정

```powershell

\# Git 사용자 정보

git config --global user.email "이메일@example.com"

git config --global user.name "changki123"



\# .gitignore 생성

echo "public/" > .gitignore



\# 커밋 \& 푸시

git add .

git commit -m "Initial commit"

git branch -M main

git remote add origin https://github.com/changki123/blog.git

git push -u origin main

```



\### 3. GitHub Actions 워크플로우



`.github/workflows/deploy.yml`:

```yaml

name: Deploy to GitHub Pages



on:

&nbsp; push:

&nbsp;   branches: \[main]



jobs:

&nbsp; build:

&nbsp;   runs-on: ubuntu-latest

&nbsp;   

&nbsp;   permissions:

&nbsp;     contents: write

&nbsp;   

&nbsp;   steps:

&nbsp;     - uses: actions/checkout@v4

&nbsp;       with:

&nbsp;         submodules: true

&nbsp;     

&nbsp;     - name: Install Zola

&nbsp;       uses: taiki-e/install-action@v2

&nbsp;       with:

&nbsp;         tool: zola@0.21.0

&nbsp;     

&nbsp;     - name: Build

&nbsp;       run: zola build --base-url https://changki123.github.io/blog

&nbsp;     

&nbsp;     - name: Deploy

&nbsp;       uses: peaceiris/actions-gh-pages@v3

&nbsp;       with:

&nbsp;         github\_token: ${{ secrets.GITHUB\_TOKEN }}

&nbsp;         publish\_dir: ./public

```



\### 4. Jekyll 비활성화



GitHub가 자동으로 Jekyll을 실행하려고 해서 `.nojekyll` 파일을 추가했다.

```powershell

echo $null > .nojekyll

echo $null > static\\.nojekyll

```



\### 5. GitHub 설정



\- Settings → Actions → Workflow permissions

\- "Read and write permissions" 선택

\- Settings → Pages

\- Source: Deploy from a branch

\- Branch: gh-pages, /root



\## 결과



https://changki123.github.io/blog



블로그가 성공적으로 배포되었다! 🎉



\## 트러블슈팅 경험



\### 문제 1: Jekyll 자동 실행

\*\*해결\*\*: `.nojekyll` 파일 추가



\### 문제 2: Git 권한 에러 (exit code 128)

\*\*해결\*\*: GitHub Actions 권한을 "Read and write"로 변경



\### 문제 3: 환경 변수 인식 안됨

\*\*해결\*\*: PowerShell 재시작 필요



---



\*\*참고 링크:\*\*

\- \[Zola 공식 문서](https://www.getzola.org/)

\- \[terminimal 테마](https://github.com/pawroman/zola-theme-terminimal)

