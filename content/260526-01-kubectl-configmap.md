+++
title = "CKA 실습 - ConfigMap 기초"
date = 2026-05-26
description = "ConfigMap 생성 및 Pod 환경변수 주입 실습"

[taxonomies]
tags = ["kubernetes", "cka", "configmap", "k8s"]
+++

쿠버네티스에서 앱 설정값을 분리해서 관리하는 방법인 ConfigMap을 실습했다.

<!-- more -->

## ConfigMap이 뭔가

앱 설정값(DB 주소, 포트, 환경 이름 등)을 Pod 안에 하드코딩하지 않고 쿠버네티스 오브젝트로 분리해서 관리하는 것.

설정값을 코드에 박아두면 환경마다 이미지를 새로 빌드해야 하는데, ConfigMap에 빼두면 이미지는 그대로고 설정만 바꾸면 된다.

Pod에 주입하는 방법은 두 가지다.

- **환경변수 방식** — 앱이 `os.getenv("KEY")` 형태로 읽을 때
- **볼륨 마운트 방식** — 앱이 파일을 직접 읽을 때 (`/etc/config/app.properties` 등)

---

## 실습

### 1. ConfigMap 생성

```bash
kubectl create configmap myconfig --from-literal=KEY=VALUE
```

### 2. 확인

```bash
kubectl get configmap myconfig -o yaml
```

`data:` 아래에 `KEY: VALUE`가 들어있으면 성공.

```yaml
apiVersion: v1
data:
  KEY: VALUE
kind: ConfigMap
metadata:
  name: myconfig
  namespace: default
```

### 3. Pod yaml 작성

들여쓰기는 스페이스 2칸 기준. 계층이 하나 내려갈 때마다 2칸 추가. `-`는 칸 수에 포함해서 센다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: busybox
    command: ["sleep", "3600"]
    env:
    - name: MY_KEY
      valueFrom:
        configMapKeyRef:
          name: myconfig
          key: KEY
```

| 필드 | 설명 |
|------|------|
| `name: MY_KEY` | Pod 안에서 쓸 환경변수 이름 |
| `name: myconfig` | 참조할 ConfigMap 이름 |
| `key: KEY` | ConfigMap 안의 키 이름 |

### 4. 적용 및 확인

```bash
kubectl apply -f myapp.yaml

kubectl exec -it myapp -- env | grep MY_KEY
# MY_KEY=VALUE 출력되면 성공
# execPod 안에서 명령어 실행
# -it터미널 연결 (interactive + tty)
```

---

## 정리

| 명령어 | 설명 |
|--------|------|
| `kubectl create configmap myconfig --from-literal=KEY=VALUE` | ConfigMap 생성 |
| `kubectl get configmap myconfig -o yaml` | ConfigMap 내용 확인 |
| `kubectl apply -f myapp.yaml` | Pod 생성 |
| `kubectl exec -it myapp -- env \| grep MY_KEY` | 환경변수 주입 확인 |

**흐름 요약:** ConfigMap 생성 → yaml에 참조 추가 → apply → exec으로 확인