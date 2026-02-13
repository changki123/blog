+++
title = "AWS EC2로 마인크래프트 서버 구축하기"
date = 2026-01-27
description = "AWS EC2 인스턴스를 활용한 마인크래프트 Java Edition 서버 구축 및 운영 가이드"

[taxonomies]
tags = ["minecraft", "aws", "ec2", "gaming", "server", "cloud"]

[extra]
toc = true
+++

## 개요

AWS EC2를 활용해 마인크래프트 Java Edition 1.21.11 서버를 구축하는 과정을 정리했습니다. 친구들과 함께 플레이할 수 있는 안정적인 멀티플레이 환경을 만들어봅시다.

<!-- more -->

## 사전 준비

### 필요한 것들
- AWS 계정
- EC2 인스턴스 (권장: t3.medium 이상, 2GB RAM 이상)
- 보안 그룹 설정 (25565 포트 개방)
- 기본적인 Linux 명령어 지식

### 사양
- **최소**?: t3.small (2GB RAM)
- **적용**: t3.medium (4GB RAM) - 5~10명 동시 접속

## 1단계: 시스템 환경 설정

먼저 EC2 인스턴스에 접속한 후 시스템을 업데이트하고 필요한 패키지를 설치합니다.
```bash
# 시스템 업데이트
sudo dnf update -y

# Java 21 설치 (마인크래프트 1.21+ 필수)
sudo dnf install java-21-amazon-corretto-headless -y

# 설치 확인
java -version
# openjdk version "21.0.x" 출력되면 성공

# 필수 도구 설치
sudo dnf install wget screen -y
```

> **참고**: 마인크래프트 1.21 이상 버전은 Java 21이 필수입니다. 이전 버전의 Java를 사용하면 서버가 시작되지 않습니다.

## 2단계: 마인크래프트 서버 파일 다운로드
```bash
# 서버 디렉토리 생성
sudo mkdir -p /opt/minecraft
sudo chown -R ec2-user:ec2-user /opt/minecraft/
cd /opt/minecraft

# 마인크래프트 1.21.11 서버 파일 다운로드 (최신버전은 알아서 변경해서 다운필요합니다.)
wget https://piston-data.mojang.com/v1/objects/64bb6d763bed0a9f1d632ec347938594144943ed/server.jar -O minecraft_server.1.21.11.jar

# EULA 동의 (필수)
echo "eula=true" > eula.txt

# 첫 실행으로 설정 파일 생성
java -Xmx2G -Xms1G -jar minecraft_server.1.21.11.jar nogui
# 월드 생성이 완료되면 Ctrl+C로 중지
```

## 3단계: 서버 설정 커스터마이징 (선택)

생성된 `server.properties` 파일을 수정해서 서버를 커스터마이징할 수 있습니다.
```bash
vi server.properties
```

주요 설정 항목:
```properties
# 서버 이름
motd=changki123's Minecraft Server

# 최대 플레이어 수
max-players=10

# 난이도 (peaceful, easy, normal, hard)
difficulty=normal

# 게임 모드 (survival, creative, adventure)
gamemode=survival

# PvP 활성화
pvp=true

# 화이트리스트 사용
white-list=false

# 뷰 거리 (청크 단위, 높을수록 부하 증가)
view-distance=10
```

## 4단계: systemd 서비스로 자동화

서버를 안정적으로 운영하기 위해 systemd 서비스로 등록합니다.
```bash
sudo vi /etc/systemd/system/minecraft.service
```

서비스 파일 내용:
```ini
[Unit]
Description=Minecraft Server 1.21.11
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/opt/minecraft
ExecStart=/usr/bin/java -Xmx3G -Xms1G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -jar minecraft_server.1.21.11.jar nogui
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**JVM 옵션 설명**:
- `-Xmx3G`: 최대 메모리 3GB (인스턴스 메모리의 75% 정도 권장)
- `-Xms1G`: 초기 메모리 1GB
- `-XX:+UseG1GC`: G1 가비지 컬렉터 사용 (성능 최적화)
- `-XX:MaxGCPauseMillis=200`: GC 일시정지 시간 최대 200ms로 제한

서비스 등록 및 시작:
```bash
# systemd 데몬 리로드
sudo systemctl daemon-reload

# 서버 시작
sudo systemctl start minecraft

# 부팅 시 자동 시작 설정
sudo systemctl enable minecraft

# 서버 상태 확인
sudo systemctl status minecraft
```

## 5단계: 로그 및 관리

### 로그 확인
```bash
# 실시간 로그 보기 (실행시키고 정상작동 하는지 확인 필요)
sudo journalctl -u minecraft -f

# 최근 100줄 보기
sudo journalctl -u minecraft -n 100
```

### 서버 관리 명령어
```bash
# 서버 중지
sudo systemctl stop minecraft

# 서버 재시작
sudo systemctl restart minecraft

# 서버 상태 확인
sudo systemctl status minecraft
```

## 보안 그룹 설정

AWS 콘솔에서 EC2 보안 그룹에 다음 규칙을 추가해야 합니다:

| 타입 | 프로토콜 | 포트 범위 | 소스 |
|------|----------|----------|------|
| 사용자 지정 TCP | TCP | 25565 | myip/0 |

> **보안 팁**: 특정 IP만 접속하도록 제한하려면 소스에 허용할 IP 주소를 입력하세요.

## 접속 방법

마인크래프트 클라이언트에서 멀티플레이 → 서버 추가 → 서버 주소에 EC2의 퍼블릭 IP 입력:
```
your-ec2-public-ip:25565
```

도메인이 있다면 A 레코드로 연결해서 사용할 수 있습니다.

service 파일에서 -Xmx3G를 -Xmx2G로 변경 해서 적용도 가능해보이는데 테스트 필요해보입니다.

```

## 간단한 백업 방법


## 마무리

비용 최적화나 백업, 자동화 등등은 다음에 ... 즐마 ⛏️