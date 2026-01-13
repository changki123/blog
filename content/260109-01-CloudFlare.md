+++
title = "Cloudflare DNS로 커스텀 도메인 구매하고 GitHub 블로그 연결하기"
date = 2026-01-12
+++

기술 블로그를 위해 `.click` 도메인을 구매하고 Cloudflare DNS를 통해 GitHub Pages에 연결한 과정을 기록합니다.

## 왜 커스텀 도메인인가?

GitHub Pages는 기본적으로 `username.github.io` 형태의 도메인을 제공하지만, 커스텀 도메인을 사용하면:

- 전문적인 브랜딩
- 기억하기 쉬운 URL
- 포트폴리오 가치 상승
- 실무 DNS 관리 경험

## 도메인 선택 과정

### TLD 비교

처음엔 `.com`을 고려했지만:
```
changki123.com - $15/년
changki123.org - $15/년

vs

changki123.click - $3/년 (5배 저렴!)
```

`.click` 도메인을 선택한 이유:
- 저렴한 비용 (연 $3)
- 등록 제약 없음 (누구나 등록 가능)
- 기술/블로그 용도로 적합
- 다양한 레지스트라에서 구매 가능

### 도메인 구조 설계 예시
```
changki123.click              → 메인 홈페이지 (향후 포트폴리오)
blog.changki123.click         → 기술 블로그 (현재)
lab.changki123.click          → 실습/테스트용 (계획)
monitor.changki123.click      → 모니터링 대시보드 (계획)
```

서브도메인으로 분리하면 용도별 관리가 쉽고 나중에 확장하기 좋습니다.

## Cloudflare DNS 설정

### 1단계: Cloudflare 계정 생성 및 사이트 추가
```
1. cloudflare.com 접속
2. 계정 가입 (무료 플랜 선택)
3. 대시보드에서 "Add a Site" 클릭
4. 도메인 입력: changki123.click
5. Free 플랜 선택
```

### 2단계: DNS 레코드 설정

Cloudflare가 기존 DNS 레코드를 자동 스캔한 후:
```
DNS → Records 메뉴

블로그용 CNAME 레코드 추가:
Type: CNAME
Name: blog
Target: changki123.github.io
Proxy status: Proxied (주황색 구름) 또는 DNS only
TTL: Auto

→ Save
```

**Proxy 설정 참고:**
- **Proxied (주황색 구름)**: Cloudflare CDN 사용, DDoS 보호, SSL 자동
- **DNS only (회색 구름)**: 단순 DNS만 사용, GitHub 직접 연결

### 3단계: 네임서버 변경

Cloudflare가 제공하는 네임서버 정보를 도메인 레지스트라에 등록합니다.
```
Cloudflare가 제공하는 네임서버 예시:
aldo.ns.cloudflare.com
raquel.ns.cloudflare.com
```

도메인 구매한 레지스트라(AWS Route 53, GoDaddy 등) 관리 페이지에서:
```
1. 도메인 관리 메뉴 접속
2. 네임서버(Name Server) 설정 찾기
3. Custom 네임서버로 변경
4. Cloudflare 네임서버 2개 입력
5. 저장
```

**전파 시간:** 최대 24~48시간 소요 (보통 몇 시간 내 완료)

### 4단계: DNS 전파 확인
```bash
# 터미널에서 확인
dig blog.changki123.click

# 또는 nslookup
nslookup blog.changki123.click

# 출력 예시:
# blog.changki123.click. 300 IN CNAME changki123.github.io.
```

## GitHub Pages 설정

### 1단계: Custom Domain 설정
```
GitHub 저장소 접속
→ Settings 탭
→ Pages 메뉴
→ Custom domain에 입력: blog.changki123.click
→ Save
```

GitHub이 자동으로:
- DNS 체크 수행
- CNAME 파일 생성 (루트 디렉토리)
- HTTPS 인증서 발급 시작

### 2단계: CNAME 파일 추가 (Zola 전용)

Zola는 빌드 시 `public` 디렉토리를 생성하므로, CNAME 파일이 포함되도록 설정해야 합니다.
```bash
# 프로젝트 루트에서
echo "blog.changki123.click" > static/CNAME
```

디렉토리 구조:
```
your-blog/
├── config.toml
├── content/
├── static/
│   └── CNAME          ← 여기에 생성
└── templates/
```

**왜 static 폴더인가?**
- Zola는 `static/` 폴더의 내용을 빌드 시 `public/`으로 그대로 복사
- GitHub Pages는 루트의 CNAME 파일을 찾음
- `static/CNAME` → 빌드 후 → `public/CNAME`
```bash
# 빌드 및 배포
git add static/CNAME
git commit -m "Add CNAME for custom domain"
git push origin main
```

### 3단계: HTTPS 활성화

GitHub Pages Settings에서:
```
☑ Enforce HTTPS (체크)
```

## (선택) 기존 경로 변경

도메인 변경으로 인해 기존 콘텐츠의 경로가 바뀐 경우:

### 이미지/미디어 파일 경로 수정
```markdown
# 변경 전
![이미지](/blog/images/sample.png)
<video src="/blog/videos/sample.mp4"></video>

# 변경 후
![이미지](/images/sample.png)
<video src="/videos/sample.mp4"></video>
```

### 링크 경로 확인
```markdown
# 내부 링크
[다른 포스트](../other-post/)  ← 상대 경로 권장

# 절대 경로 사용 시
[홈으로](https://blog.changki123.click/)
```

## 최종 확인

### 접속 테스트
```
http://blog.changki123.click   → https로 자동 리다이렉트
https://blog.changki123.click  → 정상 접속
```

### DNS 레코드 확인
```bash
dig blog.changki123.click

# Cloudflare 사용 시 출력 예시:
# blog.changki123.click. 300 IN CNAME changki123.github.io.
```

### SSL 인증서 확인

브라우저 주소창에서:
```
자물쇠 아이콘 클릭
→ 인증서 정보 확인
→ "Issued by: GitHub" 또는 "Cloudflare"
```

## 정리

### 전체 과정 요약
```
1. 도메인 구매 (.click, $3/년)
2. Cloudflare 계정 생성 및 사이트 추가 (무료 플랜)
3. Cloudflare DNS 레코드 설정 (CNAME: blog → changki123.github.io)
4. 도메인 레지스트라에서 네임서버를 Cloudflare로 변경
5. GitHub Settings → Pages → Custom domain 설정
6. static/CNAME 파일 생성 및 푸시
7. (선택) 기존 경로 수정
```

### Cloudflare 사용의 장점

- **무료 SSL/TLS**: 자동 HTTPS 적용
- **CDN**: 전 세계 빠른 접속 속도
- **DDoS 보호**: 기본 보안 제공
- **분석 도구**: 트래픽 모니터링
- **유연한 DNS 관리**: 쉬운 레코드 추가/수정

### 비용 정리
```
도메인 비용: $3/년 (.click)
Cloudflare DNS: 무료
GitHub Pages: 무료
HTTPS 인증서: 무료

총 비용: $3/년
```

## 참고 자료

- [GitHub Pages 공식 문서](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)
- [Cloudflare DNS 가이드](https://developers.cloudflare.com/dns/)
- [Zola 공식 문서](https://www.getzola.org/documentation/deployment/github-pages/)