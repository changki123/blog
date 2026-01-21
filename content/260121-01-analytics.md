+++
title = "Zola 정적 사이트에 Google Analytics 적용하기"
date = 2025-01-21
[taxonomies]
tags = ["zola", "blog", "analytics", "monitoring"]
+++

## 배경

블로그 방문자 통계를 확인하고 싶어서 Google Analytics를 적용하기로 했다.
Zola 정적 사이트 생성기와 Terminimal 테마 환경에서 GA4를 설정한 과정을 기록한다.

## Google Analytics 설정

1. Google Analytics 계정 생성
2. 속성 추가 및 데이터 스트림 설정
3. 측정 ID 확인

## Zola 블로그 설정

Terminimal 테마는 GA 지원이 내장되어 있어 `config.toml`에 측정 ID만 추가하면 된다.

\`\`\`toml
[extra]
google_analytics = "G-XXXXXXXXXX"
\`\`\`

## 배포 및 확인

- GitHub Pages 배포 후 실시간 보고서에서 트래킹 확인
- 개발자 도구 Network 탭에서 gtag.js 로드 확인

## 트러블슈팅

(발생한 이슈가 있다면 여기에 추가)

## 참고

- [Zola 공식 문서]
- [Google Analytics 설정 가이드]