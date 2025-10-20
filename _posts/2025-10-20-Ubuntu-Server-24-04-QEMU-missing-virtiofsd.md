---
title: "Ubuntu Server 24.04에서 'operation failed: Unable to find a satisfying virtiofsd' 오류 해결"
date: 2025-10-20 00:00:00 +0900
categories: [Server]
tags: [ubuntu, qemu, kvm, virt-manager, virtiofs, libvirt]
---

> **참고:** 이 게시물은 AI의 도움을 받아 작성되었습니다. 하지만 여기에 제시된 해결 방법은 블로그 운영자가 직접 겪은 문제를 해결한 실제 경험에 바탕을 두고 있습니다. AI를 활용하여 정보를 신속하고 효과적으로 전달하되, 모든 내용은 운영자가 직접 검수하였음을 알려드립니다.

## 문제 상황

`virt-manager` 또는 `libvirt`를 사용하여 KVM/QEMU 가상 머신을 관리할 때, 호스트와 게스트 운영체제 간에 파일 공유를 위해 **Virtio-FS**를 설정하고 가상 머신을 시작하려고 하면 다음과 같은 오류 메시지와 함께 실행이 실패하는 경우가 있습니다.

```
Error starting domain: internal error: qemu unexpectedly closed the monitor: ...
operation failed: Unable to find a satisfying virtiofsd
```

이 오류는 호스트 시스템에서 가상 머신을 위한 파일 공유 데몬인 `virtiofsd`를 찾을 수 없거나 실행할 수 없을 때 발생합니다.

## Virtio-FS란?

**Virtio-FS**는 가상 머신과 호스트 간에 파일 시스템을 공유하기 위해 설계된 고성능 기술입니다. 기존의 공유 방식(예: Samba, NFS, Virtio-9P)에 비해 더 나은 성능과 POSIX 파일 시스템 시맨틱 호환성을 제공하여, 특히 I/O 집약적인 작업을 할 때 유리합니다.

## 근본 원인

오류 메시지가 명확하게 알려주듯이, 문제의 핵심은 `libvirt`가 가상 머신을 시작하기 위해 필요한 `virtiofsd` 데몬을 찾지 못하는 것입니다.

Ubuntu Server 24.04에 QEMU/KVM 및 `libvirt`를 설치했더라도, Virtio-FS 기능에 필요한 `virtiofsd` 패키지는 **기본적으로 함께 설치되지 않습니다.** 따라서 사용자가 Virtio-FS 공유를 설정하더라도, 이를 지원하는 데몬이 시스템에 없기 때문에 오류가 발생하는 것입니다.

## 해결 과정

해결책은 간단합니다. 호스트 시스템에 `virtiofsd` 패키지를 직접 설치해주면 됩니다.

### 1단계: virtiofsd 패키지 설치

터미널을 열고 다음 명령어를 실행하여 `virtiofsd`를 설치합니다.

```bash
sudo apt update
sudo apt install virtiofsd
```

이 명령어는 `virtiofsd` 패키지와 관련 의존성을 시스템에 설치합니다.

### 2단계: 설치 확인 (선택 사항)

패키지가 올바르게 설치되었는지 확인하려면 다음 명령어를 사용할 수 있습니다.

```bash
which virtiofsd
```

이 명령어를 실행했을 때 `/usr/bin/virtiofsd` 또는 유사한 경로가 출력된다면 성공적으로 설치된 것입니다.

### 3단계: 가상 머신 재시작

`virtiofsd` 설치가 완료된 후, 이전에 오류가 발생했던 가상 머신을 다시 시작합니다. 이제 `libvirt`가 `virtiofsd` 데몬을 정상적으로 찾아 실행할 수 있으므로, Virtio-FS 파일 시스템이 올바르게 마운트되고 가상 머신이 성공적으로 부팅됩니다.

## 결론

"Unable to find a satisfying virtiofsd" 오류는 Ubuntu Server 24.04에서 Virtio-FS를 사용하려고 할 때 흔히 발생할 수 있는 문제입니다. 이는 QEMU/KVM을 설치할 때 모든 관련 기능이 함께 설치되지 않기 때문입니다. `virtiofsd` 패키지를 수동으로 설치하는 간단한 조치를 통해 이 문제를 해결하고, 호스트와 게스트 간의 효율적인 파일 공유를 원활하게 구성할 수 있습니다.