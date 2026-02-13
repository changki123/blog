+++
title = "AWS Auto Scaling 인스턴스 종료 시 S3 자동 백업 시스템 구축"
date = 2026-02-07
description = "Lifecycle Hook과 systemd를 활용한 Scale-in 이벤트 데이터 백업 자동화"

[taxonomies]
tags = ["AWS", "Auto Scaling", "S3", "Backup", "Lifecycle Hook", "systemd"]
+++

## 프로젝트 배경

Auto Scaling 환경에서 Scale-in(인스턴스 축소) 발생 시 **인스턴스가 삭제되면서 로그와 데이터가 함께 소실**되는 문제가 있었습니다. 

요구사항:
- 인스턴스 종료 전 로그 및 웹 패키지 자동 백업
- S3에 안전하게 저장
- 백업 완료 후 정상 종료
- 사람 개입 없이 완전 자동화

<!-- more -->

---

## 아키텍처 개요
```
Auto Scaling Group
    ↓
Lifecycle Hook (Terminating:Wait)
    ↓
systemd watcher 감지
    ↓
백업 스크립트 실행
    ↓
S3 업로드 완료
    ↓
Lifecycle Action Complete
    ↓
인스턴스 종료
```

---

## 핵심 구성 요소

### 1. ASG Lifecycle Watcher (systemd 서비스)

인스턴스가 부팅되면 자동으로 시작되어 **5초마다 Auto Scaling 상태를 체크**합니다.
```bash
# /usr/local/bin/asg-lifecycle-watcher.sh
while true; do
    RESULT=$(aws autoscaling describe-auto-scaling-instances \
        --instance-ids $INSTANCE_ID \
        --region $REGION \
        --query 'AutoScalingInstances[0].[LifecycleState,AutoScalingGroupName]' \
        --output text 2>/dev/null)
    
    LIFECYCLE_STATE=$(echo "$RESULT" | awk '{print $1}')
    
    if [ "$LIFECYCLE_STATE" == "Terminating:Wait" ]; then
        # 백업 실행!
        /usr/local/bin/s3-backup-shutdown.sh
        
        # 백업 완료 신호
        aws autoscaling complete-lifecycle-action \
            --lifecycle-action-result CONTINUE \
            --lifecycle-hook-name backup-on-terminate \
            --auto-scaling-group-name $ASG_NAME \
            --instance-id $INSTANCE_ID \
            --region $REGION
        
        break
    fi
    
    sleep 5
done
```

**핵심 로직:**
- `Terminating:Wait` 상태 감지 시 백업 스크립트 실행
- 백업 완료 후 `complete-lifecycle-action` 호출
- Auto Scaling에 "백업 끝났으니 종료해도 됨" 신호 전송

---

### 2. 로그 아카이브 스크립트

각 애플리케이션별 로그를 압축하여 S3에 업로드합니다.
```bash
# /data/backup/scripts/log_archive_to_s3.sh
DATE=$(date +%Y%m%d)
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
BACKUP_DIR=/data/backup/LOG_ARCHIVED
S3_BUCKET="s3://my-backup-bucket/web-servers/$INSTANCE_ID/LOG/$DATE/"
LOG_SOURCE=/data/backup/LOG

mkdir -p $BACKUP_DIR/$DATE
cd $LOG_SOURCE

# 각 서비스별 로그 압축
tar czf $BACKUP_DIR/$DATE/nginx_$DATE.tar.gz nginx/
tar czf $BACKUP_DIR/$DATE/app1_$DATE.tar.gz app1/
tar czf $BACKUP_DIR/$DATE/app2_$DATE.tar.gz app2/
# ... (기타 애플리케이션)

# S3 업로드
/usr/local/bin/aws s3 cp $BACKUP_DIR/$DATE/ $S3_BUCKET --recursive

# 15일 이상 된 로컬 백업 삭제
CUTOFF_DATE=$(date -d "15 days ago" "+%Y%m%d")
cd $BACKUP_DIR

for DIR in */; do
    DIR_NAME=$(basename "$DIR")
    if [[ "$DIR_NAME" =~ ^[0-9]{8}$ ]] && [[ "$DIR_NAME" -lt "$CUTOFF_DATE" ]]; then
        rm -rf "$DIR_NAME"
    fi
done
```

**특징:**
- 인스턴스 ID 기반 경로로 구분 (여러 인스턴스 백업 혼선 방지)
- 날짜별 디렉토리 생성
- 15일 이상 로컬 백업 자동 삭제 (디스크 관리)

---

### 3. systemd 서비스 등록

부팅 시 자동으로 watcher 시작:
```ini
[Unit]
Description=ASG Lifecycle State Watcher
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/asg-lifecycle-watcher.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

`Restart=always` 덕분에 스크립트가 비정상 종료되어도 자동 재시작됩니다.

---

## 백업 대상 예시

- nginx 로그
- 각 애플리케이션 서버 로그
- 압축된 오래된 로그
- 웹 패키지 파일

---

## 동작 흐름

### 정상 운영 시
```
인스턴스 부팅
    ↓
systemd가 asg-watcher.service 시작
    ↓
5초마다 Lifecycle 상태 체크 (InService 유지)
    ↓
애플리케이션 서비스들 정상 운영
```

### Scale-in 발생 시
```
Auto Scaling: Scale-in 결정
    ↓
Lifecycle Hook 발동 (Terminating:Wait)
    ↓
Watcher 감지
    ↓
백업 스크립트 실행
  - 로그 압축
  - 웹 패키지 백업
  - S3 업로드
    ↓
complete-lifecycle-action 호출
    ↓
인스턴스 종료
```

---

## 핵심 포인트

### 1. Lifecycle Hook의 중요성
일반적인 Auto Scaling 종료는 **즉시 강제 종료**됩니다.  
Lifecycle Hook을 사용하면 **종료 전 대기 시간 확보** 가능!

### 2. 인스턴스 ID 기반 경로
```bash
S3_BUCKET="s3://my-backup-bucket/servers/$INSTANCE_ID/LOG/$DATE/"
```
하드코딩된 경로 대신 **EC2 메타데이터에서 인스턴스 ID를 동적으로 가져와** 사용.  
→ 어떤 인스턴스가 종료되든 자동으로 올바른 경로에 백업!

### 3. systemd의 안정성
- `Restart=always`: 스크립트 오류 시 자동 재시작
- `After=network.target`: 네트워크 준비 후 실행
- 부팅 시 자동 시작으로 사람 개입 불필요

---

## 적용 효과

| 항목 | Before | After |
|------|--------|-------|
| Scale-in 시 데이터 | 소실 | S3 보관 |
| 백업 방식 | 수동 | 완전 자동 |
| 복구 가능 여부 | 불가능 | 가능 |
| 운영 부담 | 높음 | 거의 없음 |

---

## 트러블슈팅 팁

### Watcher가 동작하지 않을 때
```bash
# 서비스 상태 확인
systemctl status asg-watcher.service

# 로그 확인
journalctl -u asg-watcher.service -f
```

### IAM 권한 확인
EC2 인스턴스 Role에 필요한 권한:
- `autoscaling:DescribeAutoScalingInstances`
- `autoscaling:CompleteLifecycleAction`
- `s3:PutObject` (백업 버킷)

### 백업 실패 시
```bash
# 수동으로 백업 스크립트 실행해보기
/usr/local/bin/s3-backup-shutdown.sh

# S3 접근 테스트
aws s3 ls s3://my-backup-bucket/
```

---

## 참고 자료

- [AWS Auto Scaling Lifecycle Hooks 공식 문서](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html)
- [EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [systemd 서비스 작성 가이드](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

---