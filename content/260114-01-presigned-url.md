+++
title = "S3 Pre-signed URL 완벽 가이드: 보안과 편의성을 동시에"
date = 2025-01-15
description = "AWS S3 Pre-signed URL의 개념부터 실무 활용까지, 안전한 파일 공유 방법을 알아봅니다"
[taxonomies]
tags = ["AWS", "S3", "Security", "Infrastructure"]
+++

## Pre-signed URL이란?

Pre-signed URL은 AWS S3 버킷의 객체에 대해 **임시 접근 권한**을 부여하는 URL입니다. S3 버킷을 private으로 유지하면서도 특정 사용자에게 제한된 시간 동안만 파일 접근을 허용할 수 있는 강력한 기능입니다.

## 왜 Pre-signed URL을 사용해야 할까?

실무에서 다음과 같은 상황에 자주 마주칩니다:

- 고객에게 대용량 로그 파일을 공유해야 하는데, S3 버킷을 public으로 열고 싶지 않을 때
- 사용자가 업로드한 파일을 일정 시간 동안만 다운로드 가능하게 하고 싶을 때
- 백업 파일을 외부 협력사와 안전하게 공유해야 할 때

저는 3,000대 이상의 서버를 관리하면서 자동 백업 파일 공유, 모니터링 리포트 배포 등에 Pre-signed URL을 활용하고 있습니다.

## Pre-signed URL의 동작 원리
```
1. 애플리케이션이 AWS SDK를 통해 Pre-signed URL 생성 요청
2. AWS IAM 자격 증명으로 서명된 URL 생성
3. 생성된 URL에는 인증 정보가 쿼리 파라미터로 포함됨
4. 사용자는 AWS 자격 증명 없이도 해당 URL로 접근 가능
5. 만료 시간이 지나면 URL은 자동으로 무효화됨
```

## 실전 코드 예제

### Python (Boto3)로 Pre-signed URL 생성하기
```python
import boto3
from botocore.exceptions import ClientError

def create_presigned_url(bucket_name, object_name, expiration=3600):
    """
    S3 객체에 대한 Pre-signed URL 생성
    
    Args:
        bucket_name: S3 버킷 이름
        object_name: S3 객체 키
        expiration: URL 만료 시간(초), 기본값 1시간
    
    Returns:
        Pre-signed URL 문자열, 에러 발생 시 None
    """
    s3_client = boto3.client('s3')
    
    try:
        response = s3_client.generate_presigned_url(
            'get_object',
            Params={
                'Bucket': bucket_name,
                'Key': object_name
            },
            ExpiresIn=expiration
        )
    except ClientError as e:
        print(f"Error: {e}")
        return None
    
    return response

# 사용 예시
url = create_presigned_url('my-backup-bucket', 'logs/server-20250115.tar.gz', 7200)
print(f"다운로드 링크(2시간 유효): {url}")
```

### 업로드용 Pre-signed URL 생성
```python
def create_presigned_post(bucket_name, object_name, 
                          fields=None, conditions=None, expiration=3600):
    """
    파일 업로드를 위한 Pre-signed POST URL 생성
    """
    s3_client = boto3.client('s3')
    
    try:
        response = s3_client.generate_presigned_post(
            bucket_name,
            object_name,
            Fields=fields,
            Conditions=conditions,
            ExpiresIn=expiration
        )
    except ClientError as e:
        print(f"Error: {e}")
        return None
    
    return response

# 사용 예시 - 최대 10MB 파일만 업로드 가능
response = create_presigned_post(
    'my-upload-bucket',
    'uploads/${filename}',
    conditions=[
        ['content-length-range', 1, 10485760]  # 1 byte ~ 10MB
    ],
    expiration=1800  # 30분
)
```

## AWS CLI로 빠르게 생성하기

터미널에서 간단하게 Pre-signed URL을 생성할 수 있습니다:
```bash
# 다운로드용 URL (기본 1시간 유효)
aws s3 presign s3://bucket-name/path/to/file.zip

# 만료 시간 지정 (초 단위)
aws s3 presign s3://bucket-name/backup.sql --expires-in 604800  # 7일

# 특정 프로파일 사용
aws s3 presign s3://bucket-name/report.pdf --profile production --expires-in 3600
```

## 실무 활용 시나리오

### 1. 자동 백업 파일 공유 시스템

Lambda 함수와 결합하여 백업 완료 시 자동으로 Pre-signed URL을 생성하고 Slack으로 전송:
```python
import boto3
import json
import requests

def lambda_handler(event, context):
    # S3 이벤트에서 정보 추출
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Pre-signed URL 생성 (24시간 유효)
    s3_client = boto3.client('s3')
    url = s3_client.generate_presigned_url(
        'get_object',
        Params={'Bucket': bucket, 'Key': key},
        ExpiresIn=86400
    )
    
    # Slack 알림
    slack_webhook = "YOUR_SLACK_WEBHOOK_URL"
    message = {
        "text": f"백업 완료: {key}\n다운로드 링크(24시간 유효): {url}"
    }
    requests.post(slack_webhook, json=message)
    
    return {'statusCode': 200}
```

### 2. Jenkins CI/CD 빌드 아티팩트 배포
```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Upload to S3') {
            steps {
                script {
                    sh """
                        aws s3 cp build/artifact.tar.gz s3://artifacts-bucket/builds/
                        PRESIGNED_URL=\$(aws s3 presign s3://artifacts-bucket/builds/artifact.tar.gz --expires-in 7200)
                        echo "Artifact URL: \${PRESIGNED_URL}" > artifact_url.txt
                    """
                    archiveArtifacts 'artifact_url.txt'
                }
            }
        }
    }
}
```

## 보안 고려사항

Pre-signed URL 사용 시 주의할 점:

### 만료 시간 설정
- 최소 권한 원칙에 따라 **가능한 짧은 만료 시간** 설정
- 일반 파일 공유: 1-24시간
- 긴급 공유: 15-30분
- AWS 최대 제한: 7일 (STS 임시 자격 증명 사용 시)

### URL 노출 방지
```python
# ❌ 잘못된 예 - 로그에 URL 노출
logger.info(f"Generated URL: {presigned_url}")

# ✅ 올바른 예 - 민감 정보 마스킹
logger.info(f"Generated URL for object: {object_name}")
```

### CloudTrail 로깅
Pre-signed URL로 접근한 이력도 CloudTrail에 기록됩니다:
```json
{
  "eventName": "GetObject",
  "requestParameters": {
    "bucketName": "my-bucket",
    "key": "sensitive-data.json"
  },
  "sourceIPAddress": "203.0.113.42"
}
```

## 성능 최적화 팁

### 1. CloudFront와 함께 사용

대용량 파일 배포 시 CloudFront를 앞단에 두면 다운로드 속도가 크게 향상됩니다:
```python
def create_cloudfront_presigned_url(url, key_pair_id, private_key_file, expire_time):
    """CloudFront Signed URL 생성"""
    from botocore.signers import CloudFrontSigner
    
    def rsa_signer(message):
        from cryptography.hazmat.backends import default_backend
        from cryptography.hazmat.primitives import hashes, serialization
        from cryptography.hazmat.primitives.asymmetric import padding
        
        with open(private_key_file, 'rb') as key_file:
            private_key = serialization.load_pem_private_key(
                key_file.read(),
                password=None,
                backend=default_backend()
            )
        return private_key.sign(message, padding.PKCS1v15(), hashes.SHA1())
    
    cloudfront_signer = CloudFrontSigner(key_pair_id, rsa_signer)
    return cloudfront_signer.generate_presigned_url(url, date_less_than=expire_time)
```

### 2. 캐싱 전략

자주 요청되는 객체는 Pre-signed URL을 캐싱하여 API 호출 절감:
```python
from functools import lru_cache
from datetime import datetime, timedelta

@lru_cache(maxsize=100)
def get_cached_presigned_url(bucket, key, cache_key):
    """캐시 가능한 Pre-signed URL 생성"""
    s3_client = boto3.client('s3')
    return s3_client.generate_presigned_url(
        'get_object',
        Params={'Bucket': bucket, 'Key': key},
        ExpiresIn=3600
    )

# cache_key에 현재 시간의 시(hour)를 사용하여 매 시간 갱신
cache_key = datetime.now().strftime('%Y%m%d%H')
url = get_cached_presigned_url('bucket', 'file.zip', cache_key)
```

## 트러블슈팅

### 문제: "Access Denied" 에러
```python
# IAM 정책 확인
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::your-bucket/*"
        }
    ]
}
```

### 문제: URL이 만료되기 전에 접근 불가

- 서버 시간 동기화 확인 (NTP)
- AWS 리전 설정 확인
- S3 버킷 정책과 CORS 설정 검토

## 비용 최적화

Pre-signed URL 자체는 무료이지만, 실제 데이터 전송 비용이 발생합니다:

- GET 요청: $0.0004 per 1,000 requests
- 데이터 전송(아웃바운드): $0.09/GB (첫 10TB 기준)
- CloudFront 사용 시 더 저렴한 전송 비용

## 마무리

Pre-signed URL은 S3의 보안을 유지하면서도 유연한 파일 공유를 가능하게 하는 필수 기능입니다. 저는 실무에서 백업 시스템, 모니터링 리포트 배포, CI/CD 파이프라인 등에 적극 활용하고 있으며, 특히 멀티 클라우드 환경에서 AWS S3를 중앙 스토리지로 사용할 때 큰 도움이 됩니다.

여러분의 인프라 환경에 맞게 적절한 만료 시간과 보안 정책을 설정하여 안전하고 효율적인 파일 공유 시스템을 구축해보세요.

---

**관련 글**
- AWS S3 백업 자동화 구축하기
- Lambda@Edge로 동적 콘텐츠 배포하기
- 멀티 클라우드 스토리지 전략