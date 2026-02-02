+++
title = "Minecraft 서버를 위한 SRV 레코드 완벽 가이드"
date = 2026-01-31
description = "Minecraft 서버에 SRV DNS 레코드를 설정하여 포트 없이 깔끔하게 접속하는 방법"
[taxonomies]
tags = ["DNS", "Minecraft", "SRV레코드", "Cloudflare", "AWS"]
categories = ["Infrastructure", "Gaming"]
+++

## SRV 레코드란?

SRV(Service) 레코드는 특정 서비스의 위치 정보를 제공하는 DNS 레코드 타입입니다. 일반적인 A 레코드가 도메인을 IP 주소로 연결한다면, SRV 레코드는 서비스의 호스트명과 포트 정보까지 함께 제공할 수 있습니다.

<!-- more -->

## 왜 Minecraft 서버에서 SRV 레코드를 사용할까?

Minecraft Java Edition 서버는 기본적으로 25565 포트를 사용합니다. 일반적으로 서버에 접속하려면 다음과 같이 입력해야 합니다:
```
mc.example.com:25565
```

하지만 SRV 레코드를 설정하면 포트 번호 없이 깔끔하게 접속할 수 있습니다:
```
play.example.com
```

특히 다음과 같은 상황에서 유용합니다:
- 비표준 포트(예: 25566, 25567)를 사용하는 경우
- 여러 Minecraft 서버를 운영하면서 각각 다른 서브도메인으로 접속하고 싶은 경우
- 사용자 친화적인 접속 주소를 제공하고 싶은 경우

## SRV 레코드의 구조

SRV 레코드는 다음과 같은 형식을 가집니다:
```
_service._protocol.name TTL class SRV priority weight port target
```

각 필드의 의미:
- **_service**: 서비스 이름 (Minecraft는 `_minecraft`)
- **_protocol**: 프로토콜 (TCP 사용, `_tcp`)
- **name**: 서비스 도메인 이름
- **priority**: 우선순위 (낮을수록 우선, 0-65535)
- **weight**: 같은 우선순위 내에서의 가중치 (부하 분산용)
- **port**: 서비스 포트 번호
- **target**: 실제 서버 주소

## Minecraft 서버 SRV 레코드 설정 예시

### 시나리오 1: 기본 설정

실제 서버 주소가 `mc.example.com`이고 포트 `25565`를 사용하는 경우:
```
_minecraft._tcp.play.example.com. 300 IN SRV 0 5 25565 mc.example.com.
```

이제 사용자는 `play.example.com`만 입력하면 자동으로 `mc.example.com:25565`로 연결됩니다.

### 시나리오 2: 비표준 포트 사용

AWS EC2에서 포트 `25570`을 사용하는 경우:
```
_minecraft._tcp.survival.example.com. 300 IN SRV 0 5 25570 ec2-server.example.com.
```

사용자는 `survival.example.com` 입력만으로 포트 25570으로 자동 접속됩니다.

### 시나리오 3: 여러 서버 운영

생존 서버와 크리에이티브 서버를 각각 다른 포트로 운영하는 경우:
```
# 생존 서버 (포트 25565)
_minecraft._tcp.survival.example.com. 300 IN SRV 0 5 25565 mc1.example.com.

# 크리에이티브 서버 (포트 25566)
_minecraft._tcp.creative.example.com. 300 IN SRV 0 5 25566 mc2.example.com.
```

## Cloudflare에서 SRV 레코드 설정하기

Cloudflare DNS 관리 페이지에서 다음과 같이 설정합니다:

1. **Type**: SRV
2. **Name**: `_minecraft._tcp.play` (play는 원하는 서브도메인)
3. **Service**: `_minecraft`
4. **Protocol**: `TCP`
5. **Priority**: `0`
6. **Weight**: `5`
7. **Port**: `25565` (실제 서버 포트)
8. **Target**: `mc.example.com` (실제 서버 주소)
9. **TTL**: Auto 또는 300

**주의사항**: Cloudflare의 프록시 기능(주황색 구름)은 SRV 레코드에 적용되지 않으므로 회색(DNS only)으로 설정됩니다.

## 설정 확인 방법

### Windows에서 확인
```powershell
nslookup -type=SRV _minecraft._tcp.play.example.com
```

### Linux/macOS에서 확인
```bash
dig SRV _minecraft._tcp.play.example.com
```

올바르게 설정되었다면 다음과 같은 결과가 나옵니다:
```
_minecraft._tcp.play.example.com. 300 IN SRV 0 5 25565 mc.example.com.
```

## 실전 적용 팁

### DNS 전파 시간 고려

SRV 레코드 설정 후 전파까지 최대 24-48시간이 걸릴 수 있습니다. TTL을 낮게 설정하면(300초 권장) 변경사항이 더 빠르게 반영됩니다.

### 여러 서버 간 부하 분산

동일한 Priority에서 Weight 값을 조정하여 트래픽을 분산할 수 있습니다:
```
_minecraft._tcp.play.example.com. 300 IN SRV 0 70 25565 mc1.example.com.
_minecraft._tcp.play.example.com. 300 IN SRV 0 30 25565 mc2.example.com.
```

이 경우 약 70%는 mc1으로, 30%는 mc2로 분산됩니다.

### 장애 조치(Failover) 구성

Priority를 다르게 설정하여 메인 서버 장애 시 백업 서버로 자동 전환:
```
_minecraft._tcp.play.example.com. 300 IN SRV 0 5 25565 mc-main.example.com.
_minecraft._tcp.play.example.com. 300 IN SRV 10 5 25565 mc-backup.example.com.
```

## 문제 해결

### SRV 레코드가 작동하지 않을 때

#### DNS 캐시 초기화
```bash
# Windows
ipconfig /flushdns

# Linux
sudo systemd-resolve --flush-caches

# macOS
sudo dscacheutil -flushcache
```

#### 방화벽 확인

- AWS Security Group에서 해당 포트 개방 확인
- 서버 방화벽(firewalld, ufw)에서 포트 허용 확인

## 마치며

SRV 레코드를 활용하면 Minecraft 서버를 더 전문적이고 사용자 친화적으로 운영할 수 있습니다. 특히 여러 서버를 운영하거나 비표준 포트를 사용하는 경우 필수적인 설정입니다.

실제 운영 중인 AWS EC2 Minecraft 서버에 적용해보시고, 더 나은 게임 경험을 만들어보세요!