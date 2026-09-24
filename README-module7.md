# 모듈 7. UFW 방화벽 설정

## 1. 실습 목적

Ubuntu Server의 UFW(Uncomplicated Firewall)를 이용하여 서버에 필요한 네트워크 포트만 허용하고, 방화벽 정책이 HTTP 서비스 접속에 어떤 영향을 주는지 확인한다.

이번 실습에서는 모듈 6에서 구축한 Nginx 웹서버를 기준으로 다음을 검증했다.

```text
Nginx 웹서비스
     │
     │ TCP 80
     ▼
    UFW
     │
     ├─ 80/tcp 허용 → HTTP 접속 가능
     │
     └─ 80/tcp 차단 → 외부 HTTP 접속 차단
```

교재의 모듈 7 목표는 필요한 포트만 허용하여 서버 접근 정책을 관리하는 것이다. 실습 항목은 SSH/VM 콘솔 접속 수단 확보, 기본 정책 설정, SSH 22번 및 HTTP 80번 허용, UFW 활성화, 정책 확인, 80번 포트 차단과 재허용에 따른 접속 차이 확인으로 구성된다.

---

## 2. 실습 환경

| 항목 | 내용 |
|---|---|
| 서버 | `linux-web-01` |
| 서버 IP | `192.168.56.10` |
| 운영체제 | Ubuntu Server |
| 웹서버 | Nginx |
| SSH 포트 | TCP 22 |
| HTTP 포트 | TCP 80 |
| 방화벽 | UFW |
| 클라이언트 | Windows 노트북 |

---

## 3. 핵심 개념

### 3.1 UFW란?

UFW(Uncomplicated Firewall)는 Ubuntu에서 방화벽 정책을 쉽게 설정하고 관리하기 위한 도구이다.

이번 실습에서 UFW의 역할은 네트워크 접근 정책을 관리하는 것이다.

```text
외부 네트워크
     │
     ▼
 Ubuntu Server
     │
     ▼
    UFW
     │
     ├─ 허용된 연결 → 서비스로 전달
     │
     └─ 차단된 연결 → 접근 차단
```

UFW는 웹서비스를 제공하는 프로그램이 아니라 네트워크 접근 정책을 관리하는 역할을 한다.

### 3.2 Nginx와 UFW의 관계

Nginx와 UFW는 서로 다른 역할을 담당한다.

```text
Nginx
→ 웹서비스를 제공
→ HTTP 요청을 처리
→ TCP 80번 포트를 사용

UFW
→ 네트워크 접근을 통제
→ 특정 포트의 연결을 허용하거나 차단
```

따라서 다음 두 조건은 별개이다.

```text
Nginx가 실행 중이다.
        ≠
o외부에서 Nginx에 접근할 수 있다.
```

예를 들어:

```text
Nginx → active (running)
UFW   → 80/tcp DENY
```

이면 Nginx 자체는 실행 중이어도 외부 HTTP 접속은 차단될 수 있다.

반대로:

```text
UFW → 80/tcp ALLOW
Nginx → inactive
```

이면 방화벽은 허용하지만 HTTP 서비스를 제공할 Nginx가 실행되지 않아 웹서비스가 정상 동작하지 않는다.

### 3.3 Incoming / Outgoing

이번 실습에서는 다음 기본 정책을 사용했다.

```text
Incoming → DENY
Outgoing → ALLOW
```

- **Incoming**: 외부에서 서버로 들어오는 연결
- **Outgoing**: 서버에서 외부로 나가는 연결

따라서 별도로 허용하지 않은 외부 연결은 기본적으로 차단하고, 서버에서 외부로 나가는 연결은 기본적으로 허용한다.

### 3.4 명시적 ALLOW와 기본 DENY

UFW에서 특정 포트에 허용 규칙이 없다고 해서 항상 자동으로 허용되는 것은 아니다.

이번 실습에서:

```text
Default incoming policy = deny
```

였기 때문에:

```text
80/tcp ALLOW 규칙 없음
        ↓
기본 incoming 정책 적용
        ↓
80/tcp 외부 접근 차단
```

이 된다.

즉 다음 세 가지 상태를 구분해야 한다.

```text
80/tcp ALLOW
→ 명시적으로 허용

80/tcp DENY
→ 명시적으로 차단

80/tcp 규칙 없음 + Default deny incoming
→ 기본 정책에 의해 차단
```

### 3.5 IPv4 / IPv6 규칙

UFW 상태를 확인했을 때 다음처럼 IPv4와 IPv6 규칙이 별도로 표시되었다.

```text
[1] OpenSSH
[2] 80/tcp
[3] OpenSSH (v6)
[4] 80/tcp (v6)
```

이는 같은 서비스에 대해 IPv4와 IPv6 규칙이 각각 존재하기 때문이다.

---

## 4. 실습 과정

### STEP 1. 현재 UFW 상태 확인

```bash
sudo ufw status
```

현재 방화벽 활성화 여부를 확인한다.

### STEP 2. 기본 Incoming 정책 설정

```bash
sudo ufw default deny incoming
```

외부에서 서버로 들어오는 연결을 기본적으로 차단한다.

실습 결과:

```text
Default incoming policy changed to 'deny'
```

### STEP 3. 기본 Outgoing 정책 설정

```bash
sudo ufw default allow outgoing
```

서버에서 외부로 나가는 연결을 기본적으로 허용한다.

실습 과정에서 처음 다음과 같이 잘못 입력했다.

```bash
sudo ufw default allow outcoming
```

결과:

```text
ERROR: Invalid syntax
```

원인은 `outcoming`이라는 오타였다.

정확한 명령으로 다시 실행했다.

```bash
sudo ufw default allow outgoing
```

정상 결과:

```text
Default outgoing policy changed to 'allow'
```

### STEP 4. SSH 22번 포트 허용

현재 서버에 SSH로 접속하고 있으므로 SSH 접속이 차단되지 않도록 SSH 규칙을 허용한다.

```bash
sudo ufw allow OpenSSH
```

실습 결과:

```text
Rule added
Rule added (v6)
```

즉 IPv4와 IPv6 모두 SSH 관련 규칙이 추가되었다.

### STEP 5. HTTP 80번 포트 허용

모듈 6에서 사용한 Nginx 웹서버의 HTTP 포트를 허용한다.

```bash
sudo ufw allow 80/tcp
```

실습 결과:

```text
Rule added
Rule added (v6)
```

같은 명령을 두 번째 실행했을 때:

```text
Skipping adding existing rule
Skipping adding existing rule (v6)
```

가 출력되었다.

이는 오류가 아니라 이미 동일한 규칙이 존재해서 중복 등록하지 않았다는 의미이다.

### STEP 6. UFW 활성화

실습에서는 다음 명령을 실행했다.

```bash
sudo ufw enable
```

결과:

```text
Firewall is active and enabled on system startup
```

따라서 UFW가 활성화되었고 시스템 시작 시에도 활성화되도록 설정되었다.

#### 실습 과정에서 확인한 주의점

실제 진행에서는 UFW를 먼저 활성화한 후 SSH 규칙을 추가했다.

권장 순서는 다음과 같다.

```text
기본 정책 설정
      ↓
SSH 22번 허용
      ↓
HTTP 80번 허용
      ↓
UFW 활성화
```

원격 서버에서는 방화벽을 활성화하기 전에 관리 접속용 SSH가 허용되어 있는지 확인해야 한다.

### STEP 7. 방화벽 규칙 확인

```bash
sudo ufw status numbered
```

실습 후 다음과 같은 구성을 확인했다.

```text
[1] OpenSSH         ALLOW IN    Anywhere
[2] 80/tcp          ALLOW IN    Anywhere
[3] OpenSSH (v6)    ALLOW IN    Anywhere (v6)
[4] 80/tcp (v6)     ALLOW IN    Anywhere (v6)
```

이를 통해 SSH 22번과 HTTP 80번이 허용된 상태임을 확인했다.

### STEP 8. 기본 정책까지 상세하게 확인

```bash
sudo ufw status verbose
```

실습 결과:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
```

핵심은:

```text
Incoming → deny
Outgoing → allow
```

이다.

---

## 5. 80번 포트 차단 실습

### STEP 9. HTTP 80번 차단

```bash
sudo ufw deny 80/tcp
```

그 결과 80번 포트에 대한 명시적인 차단 규칙이 등록되었다.

```text
80/tcp → DENY
```

이후:

```bash
sudo ufw status numbered
```

로 확인했으며 다음과 같이 확인했다.

```text
[1] OpenSSH         ALLOW IN
[2] 80/tcp          DENY IN
[3] OpenSSH (v6)    ALLOW IN
[4] 80/tcp (v6)     DENY IN
```

### STEP 10. 외부 HTTP 접속 차단 확인

Windows 노트북에서:

```text
http://192.168.56.10
```

으로 접속하여 HTTP 접속이 차단되는 것을 확인했다.

중요한 점은 이때 Nginx가 반드시 꺼진 것은 아니라는 것이다.

```text
Nginx → 실행 중
UFW   → 80번 차단
```

이면 외부 요청이 방화벽 단계에서 차단될 수 있다.

---

## 6. 80번 포트 차단 규칙 철회 및 복구

### STEP 11. 80번 DENY 규칙 삭제

```bash
sudo ufw delete deny 80/tcp
```

80번 포트의 명시적인 DENY 규칙을 삭제한다.

### STEP 12. HTTP 80번 다시 허용

```bash
sudo ufw allow 80/tcp
```

HTTP 허용 규칙을 다시 등록한다.

### STEP 13. 최종 규칙 확인

```bash
sudo ufw status numbered
```

최종 상태에서 핵심적으로 다음 규칙이 존재하는지 확인한다.

```text
OpenSSH    ALLOW IN
80/tcp     ALLOW IN
```

### STEP 14. HTTP 접속 복구 확인

Windows 브라우저에서:

```text
http://192.168.56.10
```

으로 다시 접속하여 Nginx 페이지가 정상적으로 표시되는지 확인한다.

실습 흐름:

```text
80/tcp ALLOW
   ↓
HTTP 접속 성공

80/tcp DENY
   ↓
HTTP 접속 차단

DENY 규칙 삭제
   ↓
80/tcp ALLOW 재등록
   ↓
HTTP 접속 복구
```

이를 통해 UFW 정책이 실제 HTTP 서비스 접근에 영향을 준다는 것을 확인했다.

---

## 7. 모듈 7에서 확인한 장애 분석 관점

웹서비스가 접속되지 않을 때는 Nginx와 UFW를 동일한 문제로 보면 안 된다.

### Nginx 서비스 상태

```bash
systemctl status nginx
```

확인 내용:

```text
Nginx가 실행 중인가?
```

### 80번 포트 사용 상태

```bash
sudo ss -tulpen | grep ':80'
```

확인 내용:

```text
80번 포트를 Nginx가 LISTEN하고 있는가?
```

### UFW 정책

```bash
sudo ufw status numbered
```

확인 내용:

```text
외부에서 80번 포트로 접근하는 것을 허용하고 있는가?
```

### 실제 HTTP 응답

```bash
curl http://192.168.56.10
```

확인 내용:

```text
실제로 웹서비스가 HTTP 응답을 반환하는가?
```

즉:

```text
systemctl
→ 서비스 상태

ss
→ 포트 상태

ufw
→ 네트워크 접근 정책

curl
→ 실제 HTTP 응답
```

로 각각 역할이 다르다.

---

## 8. 실습에서 발생한 문제와 해결

### 문제 1. `outcoming` 오타

입력:

```bash
sudo ufw default allow outcoming
```

결과:

```text
ERROR: Invalid syntax
```

해결:

```bash
sudo ufw default allow outgoing
```

### 문제 2. UFW 활성화 순서

실습에서는 UFW를 먼저 활성화한 후 SSH 규칙을 추가했다.

권장 순서는:

```text
기본 정책 설정
→ SSH 허용
→ HTTP 허용
→ UFW 활성화
```

원격 서버에서는 방화벽을 활성화하기 전에 관리 접속용 SSH가 허용되어 있는지 확인한다.

### 문제 3. 80번 포트를 이미 허용했는데 다시 `allow` 실행

결과:

```text
Skipping adding existing rule
```

이는 오류가 아니다. 이미 동일한 규칙이 등록되어 있어 중복 추가를 건너뛴 것이다.

---

## 9. UFW와 Nginx의 실제 관계

이번 실습의 핵심 개념을 구조로 표현하면 다음과 같다.

```text
Windows
  │
  │ HTTP 요청
  │ 192.168.56.10:80
  ▼
Ubuntu Server
  │
  ▼
Linux 네트워크 계층
  │
  ▼
UFW가 설정한 방화벽 정책
  │
  ├─ DENY → 연결 차단
  │
  └─ ALLOW
       │
       ▼
    TCP 80
       │
       ▼
     Nginx
       │
       ▼
    HTML 응답
```

정리하면:

```text
Nginx
→ 실제 웹서비스를 제공하는 프로그램

UFW
→ 네트워크 연결을 허용/차단하는 방화벽 관리 도구
```

따라서:

```text
Nginx 실행 중 + UFW 80 DENY
→ 외부 HTTP 접속 차단 가능

Nginx 중지 + UFW 80 ALLOW
→ HTTP 서비스 정상 응답 불가
```

두 요소는 서로 다른 역할을 하며 함께 웹서비스의 접근 가능 여부를 결정한다.

---

## 10. GitHub 저장 구조

교재 기준의 저장 구조:

```text
07-ufw-firewall/
├─ README.md
└─ ufw-rules.md
```

커밋 메시지 예시:

```bash
git commit -m "Add UFW firewall configuration"
git commit -m "Document HTTP allow rule test"
```

---

## 11. 최종 검증 결과

| 항목 | 결과 |
|---|---|
| UFW 활성화 | 완료 |
| 부팅 시 UFW 자동 활성화 | 완료 |
| 기본 incoming 정책 | `deny` |
| 기본 outgoing 정책 | `allow` |
| SSH 22번 허용 | 완료 |
| HTTP 80번 허용 | 완료 |
| IPv4 규칙 확인 | 완료 |
| IPv6 규칙 확인 | 완료 |
| 80번 포트 차단 실험 | 완료 |
| 외부 HTTP 차단 확인 | 완료 |
| 80번 DENY 규칙 삭제 | 완료 |
| 80번 ALLOW 규칙 재등록 | 완료 |
| HTTP 접속 복구 확인 | 완료 |

---

## 12. 면접용 요약

> UFW를 이용해 Ubuntu Server의 기본 incoming 정책을 deny, outgoing 정책을 allow로 설정하고 SSH 22번과 HTTP 80번만 허용했습니다. 이후 80번 포트를 의도적으로 차단하여 Nginx가 실행 중이어도 방화벽 정책에 따라 외부 HTTP 접속이 차단되는 것을 확인했습니다. 차단 규칙을 삭제하고 80번 포트를 다시 허용하여 HTTP 접속이 복구되는 과정까지 검증했습니다.

---

## 13. 완료 체크리스트

- [x] UFW 상태 확인
- [x] 기본 incoming deny 설정
- [x] 기본 outgoing allow 설정
- [x] SSH 22번 허용
- [x] HTTP 80번 허용
- [x] UFW 활성화
- [x] `ufw status numbered` 확인
- [x] `ufw status verbose` 확인
- [x] 80번 포트 차단 실습
- [x] HTTP 접속 차단 확인
- [x] 80번 DENY 규칙 삭제
- [x] 80번 ALLOW 규칙 재등록
- [x] HTTP 접속 복구 확인

**모듈 7 실습 완료**
