+++
title = "Rocky Linux root 비밀번호 까먹었을 때 - GRUB으로 재설정하기"
date = 2026-05-19
[taxonomies]
tags = ["Rocky Linux", "Linux", "GRUB", "SELinux", "트러블슈팅"]
+++

운영하던 VirtualBox VM에 root로 로그인하려는데 `Login incorrect`가 떴다. 비밀번호를 바꾼 기억이 없는데... 당황하지 말고 GRUB emergency mode로 재설정하면 된다.

<!-- more -->

## 상황

VirtualBox 콘솔에서 root 계정으로 로그인 시도 → `Login incorrect` 반복 발생.  
SSH 접속도 안 되는 상황이라 GRUB에서 직접 처리해야 한다.

---

## 1. GRUB 편집 모드 진입

재부팅 후 GRUB 화면이 뜨는 순간 **`e`** 를 눌러 편집 모드로 진입한다.

커널 라인(`linux` 로 시작하는 줄)을 찾아서 맨 끝에 다음을 추가:

```
rd.break
```

그 다음 **`Ctrl + X`** 로 부팅 시작. emergency mode로 진입된다.

---

## 2. sysroot remount + chroot

emergency shell에 진입하면 `/sysroot`가 read-only로 마운트되어 있다. 쓰기 가능하게 remount 먼저:

```bash
mount -o remount,rw /sysroot
```

실제 시스템 루트로 chroot 진입:

```bash
chroot /sysroot
```

---

## 3. root 비밀번호 변경

```bash
passwd root
```

새 비밀번호 두 번 입력하면 끝.

---

## 4. SELinux relabel 예약

비밀번호 변경 후 `/etc/shadow`의 SELinux 컨텍스트가 바뀌었기 때문에, 다음 부팅 시 전체 relabel이 필요하다. 안 하면 로그인이 또 막힐 수 있다.

```bash
touch /.autorelabel
```

이 파일이 존재하면 부팅 시 자동으로 SELinux relabel을 수행하고 파일을 삭제한다.

---

## 5. 정상 부팅

```bash
exit   # chroot 탈출
exit   # emergency shell 탈출
```

이후 자동으로 relabel 진행되면서 정상 부팅. 시간이 좀 걸린다.

---

## 6. 결과

새 비밀번호로 root 로그인 성공 ✓

---

## 요약

| 단계 | 명령어 |
|------|--------|
| remount | `mount -o remount,rw /sysroot` |
| chroot | `chroot /sysroot` |
| 비밀번호 변경 | `passwd root` |
| SELinux relabel 예약 | `touch /.autorelabel` |
| 종료 | `exit` × 2 |

> **핵심**: `touch /.autorelabel` 빠뜨리면 SELinux가 변경된 shadow 파일을 거부해서 또 로그인 안 된다. 반드시 해줄 것.