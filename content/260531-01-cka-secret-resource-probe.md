+++
title = "CKA 실습 - Secret / Resource 제한 / Probe"
date = 2026-05-31
description = "CKA 준비 실습: Secret 주입, Resource requests/limits, Liveness/Readiness Probe"
[taxonomies]
tags = ["CKA", "Kubernetes", "k8s", "실습"]
+++

ConfigMap 이후 다음 단계로 Secret, Resource 제한, Probe 실습을 정리한다.

<!-- more -->

## Secret

ConfigMap과 구조는 동일하지만 **base64 인코딩**해서 저장한다.

- 용도: 비밀번호, API 키, 토큰
- `secretKeyRef` 로 환경변수 주입
- `kubectl get secret -o jsonpath ... | base64 -d` 로 디코딩 확인

### Secret 생성

```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### 확인

```bash
kubectl get secret my-secret
kubectl describe secret my-secret

# base64 디코딩 확인
kubectl get secret my-secret -o jsonpath='{.data.username}' | base64 -d
```
<img width="427" height="191" alt="Image" src="https://github.com/user-attachments/assets/13c18ca7-d063-44cd-bcc0-cc77110ecf83" />

### Pod에 환경변수로 주입

```yaml
# secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

```bash
kubectl apply -f secret-pod.yaml
kubectl exec -it secret-pod -- env | grep DB
```
<img width="532" height="49" alt="Image" src="https://github.com/user-attachments/assets/3d099e54-5b47-4f10-80bf-0e05501a8d8c" />

---

## Resource 제한

Pod이 노드 자원을 과도하게 사용하는 것을 방지한다.

- `requests` - 스케줄링 기준 (이 만큼은 보장)
- `limits` - 최대 사용량 (이 이상은 사용 불가)
- limits 초과 시 CPU는 **throttling**, 메모리는 **OOMKill** (Pod 재시작)

```yaml
# resource-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```bash
kubectl apply -f resource-pod.yaml
kubectl describe pod resource-pod | grep -A 5 Limits
```
<img width="595" height="114" alt="Image" src="https://github.com/user-attachments/assets/9abb35d9-fc1a-40d9-acaf-49d917958ffd" />

---

## Probe

컨테이너 상태를 주기적으로 체크한다.

- `livenessProbe` - 실패하면 **컨테이너 재시작**
- `readinessProbe` - 실패하면 **Service에서 제외** (트래픽 차단)
- 체크 방식: `httpGet` / `exec` / `tcpSocket`
- `initialDelaySeconds` - 첫 체크 전 대기 시간 (앱 기동 시간 고려)

```yaml
# probe-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-pod
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
```

```bash
kubectl apply -f probe-pod.yaml
kubectl describe pod probe-pod | grep -A 10 Liveness

# 상태 변화 관찰 (0/1 → 1/1 되는 과정)
kubectl get pod probe-pod -w
```
<img width="750" height="194" alt="Image" src="https://github.com/user-attachments/assets/2c670e65-eede-485c-9e7e-61063fb6a552" />

`-w` 로 보면 readinessProbe 통과하면서 `0/1 → 1/1` 로 Ready 되는 걸 확인할 수 있다.

---

## 정리

| 항목 | 핵심 |
|------|------|
| Secret | base64 인코딩, `secretKeyRef` |
| Resource | requests=보장, limits=최대, 초과 시 OOMKill |
| livenessProbe | 실패 → 재시작 |
| readinessProbe | 실패 → Service 제외 |