# 모듈 5. 서비스 상태와 로그 확인

## 1. 실습 목적

Linux 서버에서 서비스가 정상적으로 동작하는지 확인하고, 서비스의 로그와 네트워크 포트 상태를 함께 확인한다.

이번 실습에서는 `systemctl`, `journalctl`, `ss`를 사용하여 SSH 서비스를 기준으로 상태 확인 → 로그 확인 → 포트 확인 과정을 실습했다.

또한 서비스 장애가 발생했을 때 명령어를 이용해 원인을 좁혀가는 기본적인 장애 분석 흐름을 이해하는 것을 목표로 한다.

---

## 2. 실습 환경

- OS: Ubuntu Server
- 서버: `linux-web-01`
- 주요 확인 서비스: SSH
- SSH 기본 포트: TCP 22
- 이전 모듈에서 완료한 내용:
  - Ubuntu Server 환경 구성
  - Linux 기본 명령어
  - 고정 IP 설정
  - SSH Key 및 sudo 권한 설정

---

## 3. 핵심 개념

### 3.1 systemctl

`systemctl`은 Linux의 systemd 기반 서비스 상태를 관리하고 확인하는 명령어다.

이번 실습에서는 다음 명령어를 사용했다.

```bash
systemctl status ssh
