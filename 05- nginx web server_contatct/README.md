# 06. Nginx 웹서버 운영

## 1. 실습 목적
`linux-web-01` 서버에 Nginx 웹서버를 설치하고 정상적으로 운영되는지 검증한다. 단순히 설치에서 끝내지 않고, 서비스 상태·포트 리스닝·HTTP 응답·접속 로그 4가지 관점에서 웹서버가 실제로 요청을 처리하고 있는지 확인하는 것이 핵심 목표다.

## 2. 실습 환경
- Ubuntu Server (linux-web-01, 192.168.56.10)
- VirtualBox / VMware, Host-only 네트워크
- Nginx (apt 기본 저장소 버전)
- 접속 확인용 노트북 브라우저

## 3. 구성도 또는 구조
```
[노트북 브라우저] --HTTP(80)--> [linux-web-01 : Nginx]
                                   ├─ systemctl (서비스 상태)
                                   ├─ ss -tulpen (포트 리스닝)
                                   ├─ curl (응답 확인)
                                   └─ /var/log/nginx/*.log (접속 기록)
```
서버 내부에서 `curl http://localhost`로 1차 확인 후, 노트북 브라우저에서 `http://192.168.56.10`으로 외부 접근을 2차 확인하는 2단계 검증 구조로 진행했다.

## 4. 실행한 명령
```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
systemctl status nginx
sudo ss -tulpen | grep ':80'
curl http://localhost
curl http://192.168.56.10
```
- `apt update` : 패키지 목록을 최신화하여 설치 오류를 방지
- `apt install -y nginx` : Nginx 설치
- `systemctl enable --now nginx` : 서비스 등록과 즉시 실행을 동시에 처리 (재부팅 후 자동 시작 포함)
- `ss -tulpen` : 80번 포트가 LISTEN 상태인지, 어떤 프로세스가 점유하는지 확인
- `curl` : 실제 HTTP 응답(200 OK, HTML 내용)을 텍스트로 즉시 확인

## 5. 검증 결과
| 항목 | 확인 방법 | 결과 |
| --- | --- | --- |
| 서비스 상태 | `systemctl status nginx` | active (running) |
| 포트 리스닝 | `ss -tulpen | grep ':80'` | 0.0.0.0:80 LISTEN |
| 내부 응답 | `curl http://localhost` | 200, 기본 Nginx 페이지 반환 |
| 외부 응답 | 브라우저 `http://192.168.56.10` | Welcome to nginx! 페이지 표시 |
| 접속 로그 | `/var/log/nginx/access.log` | 요청 IP, 시간, 상태 코드 기록 확인 |

## 6. 발생한 문제와 해결
- **문제** : 설치 직후 `curl http://localhost`는 성공했지만 브라우저(외부 IP)에서는 응답이 없었다.
- **원인 추정** : 서비스는 떠 있는데 외부 접근이 막힌 상황 → 방화벽 또는 네트워크 어댑터 문제로 추정
- **확인 명령** : `sudo ss -tulpen`으로 80번이 `0.0.0.0`이 아닌 `127.0.0.1`에만 바인딩되어 있는지, `ip addr`로 Host-only IP가 제대로 잡혀 있는지 확인
- **실제 원인/해결** : Host-only 네트워크 어댑터의 IP 설정 확인 후 재확인하여 정상 응답 확보 (환경에 따라 UFW 미설치 상태였다면 이 단계에서는 방화벽 이슈는 아니었음, 모듈 7에서 방화벽 정책을 별도로 다룸)

## 7. 배운 점
- "설치됨"과 "서비스됨"은 다르다. 패키지 설치가 끝났다고 웹서버가 정상 운영 중이라고 단정할 수 없고, 서비스 상태 → 포트 → 응답 → 로그까지 순서대로 확인해야 신뢰할 수 있는 결과다.
- 내부(`localhost`)에서 되는데 외부에서 안 되는 경우, 문제 범위를 서버 내부(서비스/포트)와 외부(네트워크/방화벽)로 나누어 좁혀가는 접근이 효율적이다.
- 로그 파일(`access.log`)은 단순 기록이 아니라, 실제로 요청이 서버까지 도달했는지 보여주는 가장 확실한 증거다.

## 8. 면접용 요약
> Nginx를 설치한 뒤 systemctl로 서비스 상태를, ss로 포트 리스닝 상태를, curl로 실제 HTTP 응답을 확인했고, access.log를 통해 접속 기록까지 남는 것을 검증하여 웹서버가 실제로 요청을 처리하고 있음을 확인했습니다.
