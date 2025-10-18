---
title: Ubuntu 24.04에서 Cockpit '관리되지 않음' 및 '오프라인' 오류 해결
date: 2025-10-18 00:00:00 +0900
categories: [Linux]
tags: [ubuntu, cockpit, networkmanager, netplan, boot]
---

> **참고:** 이 게시물은 AI의 도움을 받아 작성되었습니다. 하지만 여기에 제시된 해결 방법은 블로그 운영자가 직접 겪은 문제를 해결한 실제 경험에 바탕을 두고 있습니다. AI를 활용하여 정보를 신속하고 효과적으로 전달하되, 모든 내용은 운영자가 직접 검수하였음을 알려드립니다.

## 문제 상황

Ubuntu Server 24.04에 Cockpit을 새로 설치하면, 서로 다른 메뉴에서 두 가지 문제가 함께 발생하는 것을 발견했습니다.

1.  **네트워킹:** '네트워킹' 탭에서 네트워크 인터페이스(예: `enp1s0`)가 **'관리되지 않음(unmanaged)'** 상태로 표시됩니다.
2.  **소프트웨어 업데이트:** '소프트웨어 업데이트' 탭에서는 **"Cannot refresh cache whilst offline"** 라는 오류가 나타납니다.

## 근본 원인

이 두 문제는 별개의 오류처럼 보이지만, 사실 **동일한 근본 원인**을 공유합니다. 바로 Cockpit이 의존하는 **NetworkManager**가 Ubuntu Server의 기본 네트워크 관리자가 아니기 때문입니다.

Cockpit은 시스템의 네트워크 상태를 파악하고 제어하기 위해 NetworkManager와 통신합니다. NetworkManager가 인터페이스를 관리하지 않으면, Cockpit은 해당 인터페이스를 '관리되지 않음'으로 표시합니다. 더 나아가, Cockpit은 네트워크 연결 상태를 확인할 수 없으므로 시스템 전체가 '오프라인' 상태라고 판단하게 됩니다. 그 결과 소프트웨어 업데이트와 같이 인터넷 연결이 필요한 기능들이 실패하는 것입니다.

## 해결 과정

해결책은 시스템의 네트워크 관리를 `systemd-networkd`에서 `NetworkManager`로 전환하고, 이로 인해 발생하는 부수적인 문제를 해결하는 것입니다.

### 1단계: Netplan 렌더러를 NetworkManager로 변경

먼저, Ubuntu의 네트워크 설정 시스템인 Netplan이 NetworkManager를 사용하도록 설정합니다.

1.  Netplan 설정 파일을 엽니다. (파일은 시스템마다 다를 수 있음.)
    ```bash
    sudo nano /etc/netplan/50-cloud-init.yaml
    ```

2.  `renderer` 항목을 `NetworkManager`로 변경하거나 없는 경우 추가합니다.
    ```yaml
    network:
      version: 2
      renderer: NetworkManager # 이 부분을 추가/수정
      ethernets:
        enp1s0:
          dhcp4: true
    ```

3.  설정을 적용합니다.
    ```bash
    sudo netplan apply
    ```
    이 작업을 마치면 Cockpit의 '네트워킹' 탭에서 인터페이스가 정상적으로 표시되고, '소프트웨어 업데이트' 오류도 함께 해결된 것을 확인할 수 있습니다.

하지만 이 변경으로 인해 재부팅 시 새로운 문제가 발생합니다.

### 2단계: 부팅 지연 문제 해결

Netplan 렌더러를 변경한 후 시스템을 재부팅하면, `Wait for Network to be Configured` 메시지와 함께 부팅이 몇 분간 지연될 수 있습니다. 이는 시스템이 더 이상 사용하지 않는 `systemd-networkd`의 네트워크 온라인 상태를 기다리기 때문입니다.

`NetworkManager`용 부팅 대기 서비스를 활성화하여 이 문제를 해결합니다.

1.  기존 `wait-online` 서비스를 비활성화하고 마스킹합니다.
    ```bash
    sudo systemctl disable systemd-networkd-wait-online.service
    sudo systemctl mask systemd-networkd-wait-online.service
    ```

2.  `NetworkManager`의 `wait-online` 서비스를 활성화합니다.
    ```bash
    sudo systemctl enable NetworkManager-wait-online.service
    sudo systemctl start NetworkManager-wait-online.service
    ```

## 결론

이제 시스템은 부팅 지연 없이 시작되며, Cockpit은 네트워크 관리와 소프트웨어 업데이트 기능을 모두 완벽하게 수행합니다.
