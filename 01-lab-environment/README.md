# 모듈 1. 실습 환경 준비

## 1. 실습 목적

VirtualBox를 사용해 Ubuntu 기반 Linux 서버 운영 실습 환경을 구성한다.  
실제 서버 장비 없이 Windows 노트북 안에서 웹서버와 백업 서버 역할을 하는 가상머신 2대를 만든다.

## 2. 핵심 개념

### VirtualBox
Windows 노트북 안에서 가상 컴퓨터를 만들고 실행할 수 있게 해주는 가상화 프로그램이다.

### 가상머신
실제 컴퓨터처럼 CPU, RAM, Disk, Network를 할당받아 동작하는 가상의 컴퓨터이다.

### ISO
운영체제를 설치하기 위한 설치 이미지 파일이다. Ubuntu ISO는 Ubuntu를 설치하기 위한 가상 설치 USB 역할을 한다.

### NAT
가상머신이 인터넷에 접속할 수 있게 해주는 네트워크 방식이다. 패키지 설치나 업데이트에 필요하다.

## 3. 실습 환경

| 항목 | 내용 |
|---|---|
| Host OS | Windows |
| Virtualization | Oracle VirtualBox |
| Guest OS | Ubuntu |
| VM 1 | linux-web-01 |
| VM 2 | linux-backup-01 |
| CPU | 1~2 core |
| RAM | 2048 MB |
| Disk | 20 GB |
| Network Adapter 1 | NAT |

## 4. 서버 역할

| 서버명 | 역할 | 설명 |
|---|---|---|
| linux-web-01 | 웹서버 | Nginx 웹서버 운영 실습용 |
| linux-backup-01 | 백업 서버 | rsync 백업 자동화 실습용 |

## 5. 진행한 작업

1. VirtualBox 설치
2. Ubuntu ISO 준비
3. linux-web-01 가상머신 생성
4. linux-backup-01 가상머신 생성
5. 각 VM에 CPU, RAM, Disk 할당
6. NAT 네트워크 설정
7. Ubuntu 설치 완료

## 6. 확인할 명령어

Ubuntu 터미널에서 아래 명령어로 기본 상태를 확인한다.

```bash
hostname
whoami
ip addr
