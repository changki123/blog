+++
title = "마인크래프트 서버 구축 - Amazon Lightsail (AL2023)"
date = 2026-02-21
[taxonomies]
tags = ["minecraft", "lightsail", "AWS"]
+++

AWS Lightsail에서 마인크래프트 1.21.11 서버를 구축하는 과정을 정리한다.

<!-- more -->

# 1. Lightsail 인스턴스 생성

<img width="862" height="656" alt="Image" src="https://github.com/user-attachments/assets/e316b8a8-805c-439a-a152-f2359b13c155" />

<img width="862" height="656" alt="Image" src="https://github.com/user-attachments/assets/5f2c7633-73a9-40b8-ba3c-76d949c5700e" />

<img width="847" height="695" alt="Image" src="https://github.com/user-attachments/assets/40c1dfc4-2811-4be5-9c45-7249e40bc059" />

<img width="330" height="308" alt="Image" src="https://github.com/user-attachments/assets/8a9b1671-ac55-47a1-bd9e-8ebdba81f952" />

인스턴스 생성 완료 후 방화벽에서 TCP 25565 포트를 열어준다.

---

# 2. 기본 환경 설치

```bash
# 시스템 업데이트
sudo dnf update -y

# Java 21 설치 (마인크래프트 1.21+)
sudo dnf install java-21-amazon-corretto-headless -y

# 설치 확인
java -version
# openjdk version "21.0.x" 나와야 함

# 필수 도구
sudo dnf install wget screen -y
```

---

# 3. 마인크래프트 서버 설치

```bash
# 디렉토리 생성
sudo mkdir -p /opt/minecraft
sudo chown -R ec2-user:ec2-user /opt/minecraft/
cd /opt/minecraft

# 서버 파일 다운로드 (1.21.11)
wget https://piston-data.mojang.com/v1/objects/64bb6d763bed0a9f1d632ec347938594144943ed/server.jar

# EULA 동의
echo "eula=true" > eula.txt

# 첫 실행 (설정 파일 생성)
java -Xms512M -Xmx1536M -jar server.jar nogui
# "Done (xx.xxxs)! For help, type "help"" 메시지 확인 후 Ctrl+C

# 권한 재설정
sudo chown -R ec2-user:ec2-user /opt/minecraft
```

---

# 4. systemd 서비스 등록

```bash
sudo vi /etc/systemd/system/minecraft.service
```

```ini
[Unit]
Description=Minecraft Server 1.21.11
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/opt/minecraft
ExecStart=/usr/bin/java -Xmx1536M -Xms512M -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -jar server.jar nogui
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now minecraft
sudo systemctl status minecraft
```

---

# 5. 스왑 메모리 설정

메모리 부족 방지를 위해 2G 스왑을 추가한다.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# swappiness 조정
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

# 6. S3 백업

```bash
cd /opt/minecraft
aws s3 sync world/ s3://버킷이름/minecraft-backups/world/
```

---

# 7. Seed 확인 (mcrcon)

`server.properties`에 RCON을 활성화한다.

```bash
vi /opt/minecraft/server.properties
```

```properties
enable-rcon=true
rcon.password=PASSWORD123123
rcon.port=25575
```

```bash
sudo systemctl restart minecraft
```

mcrcon을 빌드해서 명령어를 실행한다.

```bash
cd /tmp
sudo yum install -y git gcc
git clone https://github.com/Tiiffi/mcrcon.git
cd mcrcon
gcc -std=gnu11 -pedantic -Wall -Wextra -O2 -s -o mcrcon mcrcon.c
sudo mv mcrcon /usr/local/bin/

# op 권한 부여
mcrcon -H localhost -P 25575 -p PASSWORD123123 "op 유저명"

# 시드 확인
mcrcon -H localhost -P 25575 -p PASSWORD123123 "seed"
# Seed: [-23897777777787991]
```

---

# 8. Discord OOM 알림

메모리 부족으로 서버가 강제 종료될 경우 Discord로 알림을 받는다.

```bash
cat > /opt/minecraft/discord-oom.sh << 'EOF'
#!/bin/bash
WEBHOOK_URL="웹훅URL"
SERVER_NAME="마인크래프트"

journalctl -fu minecraft --output=cat | while read line; do
    TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
    if echo "$line" | grep -q "oom-kill"; then
        curl -s -X POST "$WEBHOOK_URL" \
            -H "Content-Type: application/json" \
            -d "{\"embeds\": [{
                \"title\": \"💀 [${SERVER_NAME}] OOM Kill 발생\",
                \"color\": 15548997,
                \"timestamp\": \"${TIMESTAMP}\",
                \"fields\": [
                    {\"name\": \"원인\", \"value\": \"메모리 부족으로 서버가 강제 종료됨\", \"inline\": false}
                ]
            }]}"
    fi
done
EOF

chmod +x /opt/minecraft/discord-oom.sh
```

```bash
cat > /etc/systemd/system/minecraft-oom-notify.service << 'EOF'
[Unit]
Description=Minecraft OOM Discord Notifier
After=minecraft.service

[Service]
ExecStart=/opt/minecraft/discord-oom.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now minecraft-oom-notify
```


---
도메인을 사용하는경우 포트를 기본값(25565)에서 변경한 경우 도메인 뒤에 포트를 명시해야 한다.
예: minecraft.game.com:포트번호