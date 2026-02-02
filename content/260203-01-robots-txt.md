+++
title = "robots.txt가 뭔지 알아보자"
date = 2025-02-03
[taxonomies]
tags = ["web", "seo", "robots-txt"]
+++

웹사이트 루트에 놓는 텍스트 파일인데, 검색엔진 크롤러한테 "여기는 긁어가도 되고, 여기는 긁어가지 마" 하고 알려주는 용도임.

<!-- more -->

## 기본 구조
```txt
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

Sitemap: https://blog.changki123.click/sitemap.xml
```

각 항목의 의미:

- **User-agent**: 어떤 봇한테 규칙 적용할건지 (Googlebot, Bingbot 등)
- **Disallow**: 크롤링 금지 경로
- **Allow**: 크롤링 허용 경로 (Disallow 안에 있어도 예외처리)
- **Sitemap**: 사이트맵 위치 알려주기

## 주의사항

### 보안 수단 아님

robots.txt는 "부탁"이지 강제가 아님. 악의적인 봇은 무시함.

### 민감정보 노출 주의

`/secret/` 이런 거 쓰면 오히려 "여기 뭔가 있구나" 하고 타겟팅됨.

### 위치 중요

반드시 루트 디렉토리에 위치해야 함 (`https://example.com/robots.txt`)

## Zola 블로그에 적용하기

Zola 같은 정적 사이트 생성기 쓸 때는 `static/` 폴더에 넣으면 빌드할 때 자동으로 루트로 복사됨:
```
myblog/
├── content/
├── static/
│   └── robots.txt  <- 여기에 넣기
├── templates/
└── config.toml
```

빌드하면 `public/robots.txt`로 복사되고, GitHub Pages에 배포하면 `https://blog.changki123.click/robots.txt`로 접근 가능.

## Cloudflare 자동 설정 예시

Cloudflare를 사용하면 AI 봇 차단 설정이 자동으로 추가됨:
```txt
# BEGIN Cloudflare Managed content
User-agent: *
Content-Signal: search=yes,ai-train=no
Allow: /

User-agent: Amazonbot
Disallow: /

User-agent: Applebot-Extended
Disallow: /

User-agent: Bytespider
Disallow: /

User-agent: CCBot
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: Google-Extended
Disallow: /

User-agent: GPTBot
Disallow: /

User-agent: meta-externalagent
Disallow: /
# END Cloudflare Managed Content

Sitemap: https://changki123.github.io/blog/sitemap.xml
```

이 설정의 의미:
- **검색엔진 크롤링은 허용** (`search=yes`)
- **AI 학습용 크롤링은 차단** (`ai-train=no`)
- OpenAI GPTBot, Anthropic ClaudeBot, Google Extended 등 AI 봇들 개별 차단

Cloudflare 대시보드에서 **Security > Bots** 메뉴의 "AI Scrapers and Crawlers" 옵션을 켜면 자동으로 관리됨.

## 확인 방법

### 커맨드라인
```bash
curl https://blog.changki123.click/robots.txt
```

### 브라우저

주소창에 `https://blog.changki123.click/robots.txt` 입력

### Google Search Console

제대로 설정됐는지 구글 서치 콘솔에서도 테스트 가능함.

## 결론

robots.txt는 검색엔진 최적화(SEO)의 기본. 크롤러한테 친절하게 가이드 제공하는 거라고 보면 됨. Cloudflare 사용하면 AI 봇 차단까지 자동으로 해주니까 편함.