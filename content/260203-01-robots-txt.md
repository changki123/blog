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

## 실제 사용 예시
```txt
# 구글만 허용
User-agent: Googlebot
Allow: /

# 나머지 봇들은 특정 경로만 차단
User-agent: *
Disallow: /draft/
Disallow: /temp/
Disallow: /*.json$

Sitemap: https://blog.changki123.click/sitemap.xml
```

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

## 확인 방법

### 커맨드라인
```bash
curl https://blog.changki123.click/robots.txt
```

### 브라우저

주소창에 `https://blog.changki123.click/robots.txt` 입력

### Google Search Console

제대로 설정됐는지 구글 서치 콘솔에서도 테스트 가능함.

## 내 블로그 설정 예시
```txt
User-agent: *
Allow: /

Sitemap: https://blog.changki123.click/sitemap.xml
```

단순하게 모든 봇 허용하고 사이트맵만 알려주는 설정. 개인 블로그는 이 정도면 충분함.

## 결론

robots.txt는 검색엔진 최적화(SEO)의 기본. 크롤러한테 친절하게 가이드 제공하는 거라고 보면 됨. Zola는 `static/` 폴더에 넣기만 하면 되니까 진짜 쉬움.