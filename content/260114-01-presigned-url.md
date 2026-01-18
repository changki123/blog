+++
title = "S3 Pre-signed URL 실전 실습 가이드: 10분 만에 마스터하기"
date = 2025-01-18
description = "AWS 콘솔부터 자동화까지, Pre-signed URL을 직접 만들고 활용하는 실습 가이드"
[taxonomies]
tags = ["AWS", "S3", "Security", "Hands-on"]
+++

## Pre-signed URL이란?

S3 버킷을 **private으로 유지**하면서도, 특정 파일에 대해 **임시 접근 권한**을 부여하는 URL입니다. 

**핵심 포인트:**
- S3 버킷은 비공개 상태 유지
- URL을 아는 사람만 제한 시간 동안 접근 가능
- 시간 지나면 자동으로 차단

## 왜 필요한가?

실무에서 이런 상황 자주 마주치지 않나요?

```
❌ 문제 상황:
- 고객에게 10GB 백업 파일 전달해야 하는데 이메일 첨부 불가
- S3 버킷을 public으로 열면 보안 위험
- 특정 사람에게만 일시적으로 파일 공유 필요

✅ Pre-signed URL 해결:
- S3는 private 유지
- URL만 전달하면 다운로드 가능
- 12시간 후 자동 만료
```

## AWS 콘솔에서 Pre-signed URL 만들기 (실습)

오늘 직접 실습해본 내용입니다. 생각보다 정말 간단합니다!

### 실습 과정

**1. S3 버킷에서 파일 선택**
- AWS Console → S3 → 버킷 선택
- 공유할 파일 체크박스 클릭

**2. 객체 작업 메뉴**
- 우측 상단 **"객체 작업(Object actions)"** 버튼 클릭
- **"미리 서명된 URL로 공유"** 선택

**3. 만료 시간 설정**
- Minutes (분) 또는 Hours (시간) 선택
- 원하는 시간 입력
- "미리 서명된 URL 생성" 버튼 클릭

**4. URL 복사 및 테스트**
- "클립보드에 복사" 버튼 클릭
- 브라우저 새 탭에 붙여넣기
- Enter → 파일 다운로드 확인! ✅

### 실습 결과

```
✅ 버킷은 private 상태 유지
✅ URL로 AWS 자격증명 없이 다운로드 가능
✅ 설정한 시간 후 자동 만료
```

### 콘솔 사용 시 참고사항

**만료 시간 제한:**
- 콘솔에서는 **Minutes와 Hours만** 선택 가능
- **최대 12시간(720분)**까지만 설정 가능
- 그 이상 필요하면 AWS CLI 사용

**언제 콘솔 사용?**
- 빠른 일회성 파일 공유
- 고객/협력사에 임시 다운로드 링크 전달
- 복잡한 설정 없이 간단하게 공유할 때

## AWS CLI로 만들기 (더 긴 만료 시간 필요할 때)

### CloudShell 사용 (설치 불필요)

AWS 콘솔 우측 상단 CloudShell 아이콘 클릭:

```bash
# 버킷 목록 확인
aws s3 ls

# 특정 버킷 내용 보기
aws s3 ls s3://your-bucket-name/

# Pre-signed URL 생성 (1시간)
aws s3 presign s3://your-bucket-name/test.txt --expires-in 3600

# 7일짜리 URL 생성
aws s3 presign s3://your-bucket-name/backup.tar.gz --expires-in 604800
```

**생성된 URL 예시:**
```
https://your-bucket.s3.ap-northeast-2.amazonaws.com/test.txt?
X-Amz-Algorithm=AWS4-HMAC-SHA256&
X-Amz-Credential=AKIA...&
X-Amz-Date=20250118T120000Z&
X-Amz-Expires=3600&
X-Amz-SignedHeaders=host&
X-Amz-Signature=abc123...
```

## 실전 활용 시나리오

### 시나리오 1: 백업 파일 Slack 공유

```python
import boto3
import requests
from datetime import datetime

SLACK_WEBHOOK = "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

def share_backup_to_slack(bucket, backup_file):
    """백업 완료 시 Slack으로 다운로드 링크 전송"""
    
    # 1. Pre-signed URL 생성 (24시간)
    s3_client = boto3.client('s3')
    url = s3_client.generate_presigned_url(
        'get_object',
        Params={'Bucket': bucket, 'Key': backup_file},
        ExpiresIn=86400  # 24시간
    )
    
    # 2. Slack 메시지 작성
    message = {
        "text": f"📦 백업 완료: {backup_file}",
        "attachments": [{
            "color": "good",
            "fields": [
                {"title": "다운로드 링크", "value": url, "short": False},
                {"title": "유효 시간", "value": "24시간", "short": True},
                {"title": "생성 시각", "value": datetime.now().strftime("%Y-%m-%d %H:%M"), "short": True}
            ]
        }]
    }
    
    # 3. Slack 전송
    response = requests.post(SLACK_WEBHOOK, json=message)
    
    if response.status_code == 200:
        print("✅ Slack 알림 전송 완료")
    else:
        print(f"❌ Slack 전송 실패: {response.status_code}")

# 실행
share_backup_to_slack('backup-bucket', 'daily/db-backup-2025-01-18.sql.gz')
```

### 시나리오 2: 모니터링 리포트 자동 배포

```python
import boto3
from datetime import datetime, timedelta

def create_daily_report_url(bucket, report_date=None):
    """일일 리포트 URL 생성"""
    
    if report_date is None:
        report_date = datetime.now().strftime('%Y-%m-%d')
    
    report_key = f'reports/daily/monitoring-{report_date}.pdf'
    
    s3_client = boto3.client('s3')
    
    # 리포트가 존재하는지 확인
    try:
        s3_client.head_object(Bucket=bucket, Key=report_key)
    except:
        print(f"❌ 리포트가 없습니다: {report_key}")
        return None
    
    # URL 생성 (12시간 유효)
    url = s3_client.generate_presigned_url(
        'get_object',
        Params={'Bucket': bucket, 'Key': report_key},
        ExpiresIn=43200
    )
    
    print(f"✅ 리포트 URL 생성: {report_date}")
    print(f"URL: {url}")
    
    return url

# cron으로 매일 오전 9시 실행
# 0 9 * * * python3 /scripts/send_report.py
```

### 시나리오 3: 서버 로그 임시 공유

```python
import boto3
import sys

def share_server_logs(server_name, log_date, expires_minutes=30):
    """특정 서버의 로그 파일을 임시로 공유"""
    
    bucket = 'server-logs'
    log_file = f'{server_name}/access-{log_date}.log.gz'
    
    s3_client = boto3.client('s3')
    
    # 파일 존재 확인
    try:
        response = s3_client.head_object(Bucket=bucket, Key=log_file)
        file_size = response['ContentLength'] / (1024*1024)  # MB
        
        print(f"📄 로그 파일 찾음: {log_file}")
        print(f"📦 크기: {file_size:.2f} MB")
        
    except:
        print(f"❌ 로그 파일이 없습니다: {log_file}")
        return None
    
    # Pre-signed URL 생성
    url = s3_client.generate_presigned_url(
        'get_object',
        Params={'Bucket': bucket, 'Key': log_file},
        ExpiresIn=expires_minutes * 60
    )
    
    print(f"\n✅ 다운로드 링크 ({expires_minutes}분 유효):")
    print(url)
    print(f"\n다운로드 명령어:")
    print(f"curl -o {server_name}-{log_date}.log.gz '{url}'")
    
    return url

# 사용 예시
if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("사용법: python share_logs.py <서버명> <날짜>")
        print("예시: python share_logs.py web-01 2025-01-18")
        sys.exit(1)
    
    server = sys.argv[1]
    date = sys.argv[2]
    
    share_server_logs(server, date, expires_minutes=30)
```

## 보안 고려사항

### 1. 적절한 만료 시간 설정

```python
# 용도별 권장 만료 시간
EXPIRATION_TIMES = {
    'emergency_share': 900,      # 15분 - 긴급 공유
    'customer_download': 3600,   # 1시간 - 고객 다운로드
    'daily_report': 43200,       # 12시간 - 일일 리포트
    'backup_archive': 604800     # 7일 - 백업 아카이브
}

# 사용
url = create_presigned_url(
    bucket='my-bucket',
    key='sensitive-data.zip',
    expires_in=EXPIRATION_TIMES['emergency_share']  # 15분만
)
```

### 2. URL 노출 방지

```python
import logging

# ❌ 나쁜 예 - 로그에 URL 노출
logger.info(f"Generated URL: {presigned_url}")

# ✅ 좋은 예 - URL은 숨기고 메타데이터만
logger.info(f"Generated presigned URL for: {bucket}/{key}, expires in {expires_in}s")
```

## 성능 최적화

### 대용량 파일은 CloudFront 사용

```python
def create_cloudfront_url(s3_key, expires_hours=24):
    """CloudFront Signed URL 생성 (더 빠른 다운로드)"""
    
    # CloudFront 설정 (한 번만)
    cloudfront_domain = 'd1234567890.cloudfront.net'
    key_pair_id = 'APKAEXAMPLE'
    private_key_file = '/path/to/private-key.pem'
    
    from botocore.signers import CloudFrontSigner
    from cryptography.hazmat.backends import default_backend
    from cryptography.hazmat.primitives import hashes, serialization
    from cryptography.hazmat.primitives.asymmetric import padding
    from datetime import datetime, timedelta
    
    def rsa_signer(message):
        with open(private_key_file, 'rb') as key_file:
            private_key = serialization.load_pem_private_key(
                key_file.read(),
                password=None,
                backend=default_backend()
            )
        return private_key.sign(message, padding.PKCS1v15(), hashes.SHA1())
    
    cloudfront_signer = CloudFrontSigner(key_pair_id, rsa_signer)
    
    url = f"https://{cloudfront_domain}/{s3_key}"
    expire_date = datetime.now() + timedelta(hours=expires_hours)
    
    signed_url = cloudfront_signer.generate_presigned_url(
        url,
        date_less_than=expire_date
    )
    
    return signed_url
```