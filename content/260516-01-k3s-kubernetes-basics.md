+++
title = "k3s로 시작하는 쿠버네티스 기본 실습"
date = 2026-05-16
[taxonomies]
tags = ["kubernetes", "k3s", "devops", "infra"]
+++

VirtualBox 위에 k3s를 올리고 Pod, Deployment, Service를 직접 띄워보며 쿠버네티스 기본 흐름을 체감한 기록이다.  
마지막엔 NodePort로 외부에서 nginx에 curl까지 해봤다.

<!-- more -->

---

## 네트워크 설정

VirtualBox 기본 설정에선 인터페이스가 자동 연결이 안 되는 경우가 있다.  
`nmcli`로 수동으로 올리고, 재부팅 후에도 유지되도록 `autoconnect`를 설정해준다.

```bash
# 인터페이스 상태 확인 후 연결
sudo nmcli device status
sudo nmcli device connect enp0s3

# 재부팅 후에도 자동 연결
nmcli connection modify "enp0s3" connection.autoconnect yes
nmcli connection show enp0s3 | grep autoconnect
```

> `autoconnect` 안 해두면 VM 재시작할 때마다 수동으로 연결해야 한다. 꼭 해두자.

---

## k3s 설치

기본 latest로 설치하면 cgroup v2만 지원하는 버전이 올라오는 경우가 있다.  
VirtualBox 기본 커널은 **cgroup v1**이라 버전을 고정해서 설치했다.

```bash
# cgroup v1 지원 확인된 v1.28.8로 고정 설치
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.28.8+k3s1" sh -
```

> 최신 버전(v1.30+)은 cgroup v2 required라 컨테이너가 `CrashLoopBackOff`로 뜰 수 있다. VM 커널 버전 먼저 확인하자.

---

## 첫 Pod 띄우기

클러스터가 올라왔으면 nginx Pod 하나 띄워서 기본 동작을 확인해본다.  
`Ready 1/1`이 나오면 성공이다.

```bash
kubectl run nginx --image=nginx

kubectl get pods
# NAME    READY   STATUS    RESTARTS   AGE
# nginx   1/1     Running   0          23s
```

Pod 안에 직접 들어가서 nginx가 응답하는지도 확인해봤다.

```bash
kubectl describe pod nginx
kubectl logs nginx

# Pod 안으로 진입
kubectl exec -it nginx -- /bin/bash
curl localhost    # Welcome to nginx! 응답 확인
exit

kubectl delete pod nginx
```

> Pod는 삭제하면 그냥 없어진다. 이게 Deployment와의 차이다.

---

## Deployment & ReplicaSet 체감

Deployment를 쓰면 Pod가 죽어도 자동으로 다시 살아난다.  
`--replicas=3`으로 띄우고 Pod 하나 강제로 지워보면 바로 체감된다.

```bash
kubectl create deployment web --image=nginx --replicas=3

<img width="435" height="125" alt="Image" src="https://github.com/user-attachments/assets/6196aa3f-eeca-42e5-8e54-fc08f2ba3986" />

kubectl get all
# NAME                       READY   STATUS
# pod/web-5d87b8d4f5-4xkzp   1/1     Running
# pod/web-5d87b8d4f5-9v2rp   1/1     Running
# pod/web-5d87b8d4f5-n8qmx   1/1     Running

<img width="592" height="222" alt="Image" src="https://github.com/user-attachments/assets/70e708b7-e1ec-4661-89a7-941751bb07a5" />

# 전체 네임스페이스 확인
kubectl get all -A

<img width="1014" height="624" alt="Image" src="https://github.com/user-attachments/assets/09355656-6c5e-4b5d-8d90-3c7fac45b007" />

# Pod IP 확인
kubectl get pods -o wide

<img width="1034" height="80" alt="Image" src="https://github.com/user-attachments/assets/3df54514-113c-4e1a-90ca-f6cd63e72946" />

# 정리 — RS, Pod 전부 같이 삭제됨
kubectl delete deployment web
```

Deployment가 ReplicaSet을 관리하고, ReplicaSet이 Pod 3개를 유지한다.  
Pod 하나를 지우면 ReplicaSet이 감지해서 새 Pod를 자동으로 다시 만든다.  
**"아 Deployment가 유지시키는구나"** 하고 바로 체감됐다.

---

## Service — 외부에서 접근하기

Pod끼리는 클러스터 내부 IP로 통신하는데, 외부에서 접근하려면 Service가 필요하다.  
ClusterIP → NodePort 순서로 만들어봤다.

```bash
kubectl create deployment web --image=nginx --replicas=3

# ClusterIP — 클러스터 내부 전용
kubectl expose deployment web --port=80 --type=ClusterIP
kubectl describe svc
# Endpoints: 10.42.0.12:80,10.42.0.13:80,10.42.0.14:80

<img width="627" height="97" alt="Image" src="https://github.com/user-attachments/assets/dc351159-c634-4305-b713-b81b2c5460c9" />

kubectl describe svc web

<img width="486" height="255" alt="Image" src="https://github.com/user-attachments/assets/6a2af0ef-9636-411b-abb5-88fdb006881a" />

# NodePort — 외부 접근 가능
kubectl expose deployment web --type=NodePort --port=80 --name=web-node
kubectl get svc
# NAME       TYPE       PORT(S)
# web        ClusterIP  80/TCP
# web-node   NodePort   80:30808/TCP

<img width="593" height="81" alt="Image" src="https://github.com/user-attachments/assets/a853cdc4-1563-4751-9b05-0bc1851b1974" />

# 노드 IP 확인 후 curl
hostname -I
<img width="271" height="34" alt="Image" src="https://github.com/user-attachments/assets/c7d15b87-a74f-4cfe-bf09-ce3614d16a6f" />

curl http://119.192.20.14:30808
<img width="564" height="447" alt="Image" src="https://github.com/user-attachments/assets/65a20fe7-b405-4e52-b5ac-c560f17e8873" />

```

> NodePort 범위는 기본 30000~32767. 포트 지정 안 하면 랜덤 할당된다.  
> 실서비스에선 LoadBalancer나 Ingress를 쓰는 게 일반적.

---

Pod → Deployment → Service 흐름을 직접 손으로 다 쳐봤다.  
다음엔 Ingress로 도메인 기반 라우팅이랑 ConfigMap/Secret으로 환경변수 주입하는 부분을 다뤄볼 예정이다.