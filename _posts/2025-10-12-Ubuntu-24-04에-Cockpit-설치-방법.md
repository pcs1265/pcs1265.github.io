---
title: Ubuntu 24.04에 Cockpit 설치 방법
date: 2025-10-12 00:00:00 +0900
categories: [Server]
tags: [ubuntu, cockpit]
---

## Cockpit이란?

Cockpit은 웹 브라우저를 통해 Linux 서버를 관리하고 모니터링할 수 있는 오픈 소스 도구입니다.  
시스템 리소스 사용량 확인, 서비스 관리, 로그 분석, 사용자 계정 관리 등 다양한 작업을 그래픽 인터페이스(GUI)로 편리하게 수행할 수 있도록 도와줍니다.

## 설치

[Cockpit 공식 설치가이드(Ubuntu)](https://cockpit-project.org/running.html#ubuntu) 사이트에 따르면 아래 명령어로 설치하도록 가이드하고 있습니다.

```bash
. /etc/os-release
sudo apt install -t ${VERSION_CODENAME}-backports cockpit
```

아래 명령어로 가이드하는 곳도 있지만, 최신버전이 아닌 구버전이 설치되는 방법이니 최신버전을 이용하기 위해서는 위의 명령어를 사용해야합니다.
```bash
sudo apt update
sudo apt install cockpit    #최신버전을 설치하지 않음!
```

## 방화벽 설정

Cockpit은 기본적으로 9090 포트를 사용합니다. 방화벽이 활성화된 경우 다음 명령어로 9090 포트를 허용해야 합니다.

```bash
sudo ufw allow 9090
sudo ufw reload
```

## Cockpit 접속

설치가 완료되면 웹 브라우저에서 다음 주소로 접속할 수 있습니다.
```
https://<서버-IP-주소>:9090
```

로그인 시 Ubuntu 서버의 사용자 계정과 비밀번호를 사용합니다.

## 마무리

이제 웹 브라우저에서 편리하게 Ubuntu 서버를 관리할 수 있습니다.  
Cockpit은 시스템 관리를 훨씬 직관적이고 쉽게 만들어줍니다.  
특히 가상머신을 관리하기에 아주 유용합니다.
