## 배운 개념

이번 실습에서는 VirtualBox의 NAT와 Host-only Adapter를 구분해 사용했다. NAT는 VM이 인터넷에 접속하기 위한 네트워크이고, Host-only Adapter는 Windows Host와 VM, VM 간 내부 통신을 위한 네트워크이다.

linux-web-01에는 `192.168.56.10/24`, linux-backup-01에는 `192.168.56.20/24`를 설정했다. 두 서버는 같은 `192.168.56.0/24` 대역에 있으므로 서로 통신할 수 있다.

Ubuntu의 네트워크 설정은 netplan을 통해 관리했고, YAML 들여쓰기 오류를 수정하면서 설정 파일의 구조가 중요하다는 것을 확인했다.

## 명령어별 목적

| 명령어 | 사용 목적 |
|---|---|
| `ip addr` | 네트워크 인터페이스와 IP 주소 확인 |
| `ls /etc/netplan` | netplan 설정 파일 이름 확인 |
| `sudo cp ... ...bak` | 설정 변경 전 원본 파일 백업 |
| `sudo nano /etc/netplan/01-network-manager-all.yaml` | netplan 설정 파일 수정 |
| `sudo chmod 600 /etc/netplan/01-network-manager-all.yaml` | 설정 파일 권한을 안전하게 변경 |
| `sudo netplan try` | 네트워크 설정을 임시 적용하여 오류 확인 |
| `sudo netplan apply` | 검증된 네트워크 설정 최종 적용 |
| `ping 192.168.56.20` | web 서버에서 backup 서버로 내부 통신 확인 |
| `ping 192.168.56.10` | backup 서버에서 web 서버로 내부 통신 확인 |
| `ping 8.8.8.8` | NAT를 통한 인터넷 연결 확인 |
| `ping google.com` | DNS 이름 해석까지 정상인지 확인 |
| `ip route` | 패킷이 나가는 네트워크 경로 확인 |

## 실습 결과

| 서버 | NAT IP | Host-only IP | 역할 |
|---|---|---|---|
| linux-web-01 | 10.0.2.x | 192.168.56.10/24 | 웹서버 |
| linux-backup-01 | 10.0.2.15/24 | 192.168.56.20/24 | 백업 서버 |

## 문제 해결 기록

netplan 설정 중 YAML 들여쓰기 오류가 발생했다.  
`renderer`, `ethernets`, `addresses` 항목의 들여쓰기 위치를 수정했고, IP 주소 앞에 `-`를 붙여 리스트 형식으로 작성하여 문제를 해결했다.

## 면접용 요약

VirtualBox에서 NAT와 Host-only Adapter를 분리해 Ubuntu 서버 2대를 구성했습니다. NAT는 인터넷 연결용으로 DHCP를 사용했고, Host-only Adapter에는 고정 IP를 설정해 서버 간 통신을 안정적으로 구성했습니다. netplan 설정 후 `ip addr`, `ping`, `ip route`로 IP, 통신, 라우팅 경로를 검증했습니다.
