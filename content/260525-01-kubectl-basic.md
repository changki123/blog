+++
title = "CKA 연습 - kubectl 기본 명령어 (Deployment, Service, Rolling Update)"
date = 2026-05-25
description = "VirtualBox 환경에서 CKA 시험 대비 kubectl 기본 명령어 연습 - Deployment 생성, NodePort Service, Scale, Rolling Update, Rollback"
[taxonomies]
tags = ["CKA", "Kubernetes", "kubectl", "DevOps"]
+++

VirtualBox에서 CKA 시험 대비 kubectl 기본 명령어를 연습했다.

<!-- more -->

## 환경

- VirtualBox VM (로컬)
- kubeadm으로 구성한 단일 노드 클러스터

## 1. Deployment 확인

```bash
kubectl get pods
```

```
NAME                   READY   STATUS    RESTARTS      AGE
web-76fd95c67-dvrgg    1/1     Running   4 (2m59s ago)  9d
web-76fd95c67-6ad92    1/1     Running   4 (2m59s ago)  9d
web-76fd95c67-22ncc    1/1     Running   4 (2m59s ago)  9d
```

web deployment가 3개 pod로 정상 동작 중인 상태.

## 2. NodePort Service 생성

```bash
kubectl expose deployment web --type=NodePort --port=80 --name=web-node
kubectl get svc web-node
```

외부에서 NodeIP:NodePort로 접근할 수 있도록 Service를 생성한다.

## 3. Replicas 스케일링

```bash
kubectl scale deployment web --replicas=5
kubectl get pods -w
```

`-w` 옵션으로 pod가 올라오는 것을 실시간으로 확인할 수 있다. 3개 → 5개로 늘어난 것을 확인했다.

## 4. Rolling Update

```bash
kubectl set image deployment/web nginx=nginx:1.25
kubectl rollout status deployment/web
```

> **주의:** `set image` 명령어에서 컨테이너 이름을 정확히 입력해야 한다.
> Deployment 이름(`web`)과 컨테이너 이름(`nginx`)은 다르다.

컨테이너 이름 확인 방법:

```bash
kubectl describe deployment web | grep -A2 "Containers:"
```

```
Containers:
  nginx:
    Image: nginx
```

이미지 업데이트 완료 후 확인:

```bash
kubectl describe deployment web | grep Image
# Image: nginx:1.25
```

## 5. Rollback

```bash
kubectl rollout undo deployment/web
kubectl describe deployment web | grep Image
# Image: nginx
```

이전 버전으로 정상 롤백되었다.

## 핵심 정리

| 명령어 | 설명 |
|--------|------|
| `kubectl scale deployment <name> --replicas=N` | pod 개수 조정 |
| `kubectl set image deployment/<name> <컨테이너>=<이미지>` | 이미지 업데이트 |
| `kubectl rollout status deployment/<name>` | 롤아웃 진행 상황 확인 |
| `kubectl rollout undo deployment/<name>` | 이전 버전으로 롤백 |

## 다음 단계

- ConfigMap / Secret
- Namespace + RBAC
- Static Pod / Taint & Toleration
+++