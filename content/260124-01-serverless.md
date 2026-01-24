+++
title = "Zola 정적 사이트에 Google Analytics 적용하기"
date = 2026-01-21
[taxonomies]
tags = ["zola", "blog", "analytics", "monitoring"]
+++

## 배경

블로그 방문자 통계를 확인하고 싶어서 Google Analytics 4(GA4)를 적용하기로 했다.
Zola 정적 사이트 생성기와 Terminimal 테마 환경에서 GA4를 설정한 과정을 기록한다.

## Google Analytics 설정

### 1. GA4 속성 생성

1. [Google Analytics](https://analytics.google.com) 접속
2. 관리 → 계정 만들기
3. 속성 추가 → 속성 이름 입력
4. 데이터 스트림 → 웹 선택
5. 웹사이트 URL과 스트림 이름 입력
6. 측정 ID(G-XXXXXXXXXX) 복사

### 2. 데이터 수집 활성화 확인

관리 → 데이터 수집 및 수정 → 데이터 수집에서 토글이 활성화되어 있는지 확인한다.

## Zola 블로그에 GA 코드 추가

Terminimal 테마는 기본적으로 Google Analytics를 지원하지 않아 템플릿 오버라이드를 통해 직접 추가해야 한다.

### 1. config.toml 설정

먼저 `config.toml` 파일의 `[extra]` 섹션에 측정 ID를 추가한다.
```toml
[extra]
accent_color = "orange"
logo_text = "changki123"
google_analytics = "G-XXXXXXXXXX"  # 실제 측정 ID로 변경
```

### 2. 로컬 템플릿 오버라이드

테마 파일을 직접 수정하지 않고 로컬에 복사하여 커스터마이징한다.
```powershell
# 로컬 templates 디렉토리 생성
mkdir templates\macros -Force

# 테마의 head.html 파일 복사
copy themes\terminimal\templates\macros\head.html templates\macros\head.html
```

### 3. GA 코드 삽입

복사한 `templates\macros\head.html` 파일을 열어 `{% endmacro head %}` 바로 위에 다음 코드를 추가한다.
```html
{%- if config.extra.google_analytics %}
    <!-- Google tag (gtag.js) -->
    <script async src="https://www.googletagmanager.com/gtag/js?id={{ config.extra.google_analytics }}"></script>
    <script>
      window.dataLayer = window.dataLayer || [];
      function gtag(){dataLayer.push(arguments);}
      gtag('js', new Date());
      gtag('config', '{{ config.extra.google_analytics }}');
    </script>
{% endif -%}
```

## 배포 및 확인

### 1. 로컬 빌드 테스트
```powershell
zola build
Select-String -Path public\index.html -Pattern "gtag"
```

gtag 스크립트가 검색되면 정상적으로 삽입된 것이다.

### 2. Git 커밋 및 배포
```powershell
git add templates\macros\head.html config.toml
git commit -m "Add Google Analytics tracking"
git push
```

### 3. 실시간 데이터 확인

배포 후 5-10분 뒤 다음 방법으로 확인한다.

**브라우저 개발자 도구 확인:**
1. 블로그 접속 후 F12
2. Network 탭에서 `gtag` 검색
3. `gtag/js?id=G-XXXXXXXXXX` 요청 확인

**Google Analytics 실시간 보고서:**
1. GA 관리 페이지 → 실시간 메뉴
2. 블로그 접속 상태로 대기
3. 활성 사용자 1명 표시 확인

## 트러블슈팅

### 데이터 수집이 활성화되지 않음

GA 관리 → 데이터 수집 및 수정 → 데이터 수집에서 토글을 활성화한다.

### 브라우저에서 gtag 스크립트가 로드되지 않음

- GitHub Actions에서 배포가 정상 완료되었는지 확인
- 브라우저 캐시를 지우고 시크릿 모드로 재접속

## 참고

- [Zola 공식 문서](https://www.getzola.org/documentation/)
- [Google Analytics 설정 가이드](https://support.google.com/analytics/)