+++
title = "AWS Route 53으로 커스텀 도메인 구매하고 GitHub 블로그 연결하기"
date = 2026-01-12
+++

기술 블로그를 위해 AWS Route 53에서 `.click` 도메인을 구매하고 GitHub Pages에 연결한 과정을 기록합니다.

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
- Route 53에서 구매 가능

### 도메인 구조 설계 예시
```
changki123.click              → 메인 홈페이지 (향후 포트폴리오)
blog.changki123.click         → 기술 블로그 (현재)
lab.changki123.click          → 실습/테스트용 (계획)
monitor.changki123.click      → 모니터링 대시보드 (계획)
```

서브도메인으로 분리하면 용도별 관리가 쉽고 나중에 확장하기 좋습니다.

## Route 53 도메인 구매

### 1단계: 도메인 검색
```
AWS Console 로그인
→ Route 53 서비스 접속
→ 왼쪽 메뉴 "Registered domains" 클릭
→ "Register Domain" 버튼
```

### 2단계: 도메인 선택
```
도메인 검색: changki123
→ .click 확장자 선택
→ Add to cart
→ Continue
```

### 3단계: 연락처 정보 입력
```
Contact Type: Person (개인)
First Name / Last Name: 영문 입력
Email: 본인 이메일 (중요!)
Phone: +82-10-xxxx-xxxx
Address: 영문 주소

Privacy Protection: 기본 활성화 (개인정보 보호)
Auto-renew: 체크 (자동 갱신)
```

**주의사항:**
- 이메일 주소는 반드시 확인 가능한 주소 사용
- 도메인 인증 이메일을 받게 됨

### 4단계: 결제 및 대기
```
Total cost: $3.00 USD
→ Complete Order

상태: 진행 중 (Pending)
예상 시간: 10분~24시간
```

## 도메인 인증

### 이메일 인증 필수

도메인 등록 후 Amazon Registrar로부터 이메일이 옵니다:
```
From: Amazon Registrar
Subject: [Action Required] Email Address Verification
```

**중요:** 15일 내 인증하지 않으면 도메인이 정지됩니다!
```
1. 받은편지함 확인 (스팸함도 체크!)
2. 이메일의 인증 링크 클릭
3. "Email address verified" 확인
```

### 등록 완료 확인
```
Route 53 → Registered domains
→ changki123.click

상태: 진행 중 → 등록 완료
Hosted Zone: 자동 생성됨
```

## DNS 설정 (Hosted Zone)

도메인이 등록되면 Hosted Zone이 자동으로 생성됩니다.

### Hosted Zone 확인
```
Route 53 → Hosted zones
→ changki123.click

기본 레코드:
- NS (Name Server) 레코드
- SOA (Start of Authority) 레코드
```

### 블로그용 CNAME 레코드 추가
```
Create record 클릭

Record name: blog
Record type: CNAME
Value: changki123.github.io
TTL: 300 (5분)

→ Create records
```

**중요:** Value에는 경로를 포함하지 않습니다!
```
✅ changki123.github.io
❌ changki123.github.io/blog/
```

### DNS 레코드 확인
```bash
# 터미널에서 확인
dig blog.changki123.click

# 출력 예시:
# blog.changki123.click. 300 IN CNAME changki123.github.io.
# changki123.github.io. 3600 IN A 185.199.108.153
```
---