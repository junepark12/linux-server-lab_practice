# 모듈 3. 고정 IP와 네트워크 연결 확인

## 실습 목적
linux-web-01과 linux-backup-01에 고정 IP를 설정하고 두 서버 간 통신을 확인한다.

## IP 설계
| 서버명 | 역할 | Host-only IP |
|---|---|---|
| linux-web-01 | 웹서버 | 192.168.56.10/24 |
| linux-backup-01 | 백업 서버 | 192.168.56.20/24 |

## 사용한 네트워크 방식
| 어댑터 | 방식 | 목적 |
|---|---|---|
| 어댑터 1 | NAT | 인터넷 연결 |
| 어댑터 2 | Host-only Adapter | VM 간 내부 통신 |

## 사용 명령어
```bash
ip addr
sudo nano /etc/netplan/01-network-manager-all.yaml
sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
sudo netplan try
sudo netplan apply
ping 192.168.56.20
ping 192.168.56.10
ping 8.8.8.8
ip route
