# Module 4 — SSH 원격 접속 및 SSH Key 인증 실습

## 1. 실습 목적

이번 모듈에서는 Ubuntu Server에 SSH 서버를 구성하고, Windows PowerShell에서 Ubuntu로 원격 접속하는 과정을 실습했습니다.

최종 목표는 Ubuntu 계정 비밀번호가 아니라 **SSH 공개키 인증 방식**으로 Windows에서 Ubuntu Server에 접속하는 것입니다.

이번 실습을 통해 다음 내용을 학습했습니다.

- SSH의 기본 개념
- SSH Client와 SSH Server의 역할
- 공개키와 개인키의 차이
- `authorized_keys`의 역할
- SSH 인증 과정
- Linux 파일 권한과 소유권
- Windows PowerShell에서 SSH Key 전송
- 공개키 인증 접속 테스트
- SSH 접속 오류 해결 과정

---

## 2. 실습 환경

| 항목 | 내용 |
|---|---|
| 클라이언트 | Windows PowerShell |
| 서버 | Ubuntu Server 22.04.5 LTS |
| 서버 호스트명 | `linux-web-01` |
| Ubuntu 계정 | `juneseo` |
| Ubuntu IP 주소 | `192.168.56.10` |
| SSH 포트 | `22` |
| 키 종류 | Ed25519 |
| 개인키 파일 | `id_ed25519` |
| 공개키 파일 | `id_ed25519.pub` |

---

## 3. SSH 개념 정리

### 3.1 SSH란?

SSH는 **Secure Shell**의 약자로, 네트워크를 통해 다른 컴퓨터에 안전하게 원격 접속하기 위한 프로토콜입니다.

SSH를 사용하면 Windows에서 Ubuntu Server의 터미널에 접속하여 명령어를 실행할 수 있습니다.

SSH는 일반적으로 다음과 같은 용도로 사용됩니다.

- 원격 서버 접속
- 서버 관리
- 파일 전송
- 명령어 원격 실행
- 공개키 기반 인증

기본 SSH 접속 형식은 다음과 같습니다.

```text
ssh 사용자명@서버IP
```

실습에서 사용한 접속 형식은 다음과 같습니다.

```powershell
ssh juneseo@192.168.56.10
```

---

### 3.2 SSH Client와 SSH Server

#### SSH Client

접속을 시도하는 컴퓨터입니다.

이번 실습에서는 Windows PowerShell이 SSH Client 역할을 수행했습니다.

#### SSH Server

SSH 접속 요청을 받아 인증을 처리하고 원격 터미널을 제공하는 컴퓨터입니다.

이번 실습에서는 Ubuntu Server가 SSH Server 역할을 수행했습니다.

구조는 다음과 같습니다.

```text
Windows PowerShell
       |
       | SSH 접속 요청
       v
Ubuntu Server
192.168.56.10:22
       |
       v
juneseo 계정으로 인증
```

---

## 4. SSH Key 개념

### 4.1 공개키와 개인키

SSH Key 인증은 한 쌍의 키를 사용합니다.

- 공개키: 다른 컴퓨터에 등록하는 키
- 개인키: 본인 컴퓨터에 보관하는 비밀 키

이번 실습에서 생성한 키는 다음과 같습니다.

| 파일 | 역할 | 보관 방법 |
|---|---|---|
| `id_ed25519` | 개인키 | 외부에 공개하면 안 됨 |
| `id_ed25519.pub` | 공개키 | 서버에 등록 가능 |

### 4.2 개인키

개인키는 Windows에 보관하며, SSH 인증을 수행할 때 사용합니다.

개인키는 절대로 다른 사람에게 전달하거나 GitHub에 업로드하면 안 됩니다.

실습에서 사용한 개인키 경로는 다음과 같습니다.

```text
C:\Users\dudrn\.ssh\id_ed25519
```

### 4.3 공개키

공개키는 접속하려는 Ubuntu Server에 등록합니다.

Ubuntu에서는 일반적으로 다음 파일에 공개키를 저장합니다.

```text
/home/juneseo/.ssh/authorized_keys
```

공개키는 서버에 등록되어 있어야 하며, Windows의 개인키와 한 쌍으로 동작합니다.

---

## 5. SSH 인증 과정

SSH Key 인증 과정은 다음과 같습니다.

1. Windows에서 SSH 접속 요청을 보냅니다.
2. Ubuntu SSH Server가 접속 사용자를 확인합니다.
3. Ubuntu의 `authorized_keys`에 등록된 공개키를 확인합니다.
4. Windows SSH Client가 개인키를 이용해 인증을 시도합니다.
5. 서버가 공개키와 개인키의 대응 관계를 검증합니다.
6. 인증이 성공하면 Ubuntu 터미널에 접속됩니다.

중요한 점은 다음과 같습니다.

> 서버에는 공개키를 등록하고, 클라이언트는 개인키를 사용합니다.

---

## 6. Ubuntu SSH 서버 설정

### 6.1 SSH 서비스 상태 확인

Ubuntu에서 SSH 서비스가 실행 중인지 확인합니다.

```bash
systemctl status ssh
```

정상적인 상태는 다음과 같습니다.

```text
Active: active (running)
```

SSH 서버가 실행 중이면 Ubuntu는 기본적으로 TCP 22번 포트에서 SSH 접속을 기다립니다.

---

## 7. Ubuntu 사용자 및 SSH 디렉터리 설정

### 7.1 사용자 확인

실습에서 사용한 Ubuntu 계정은 `juneseo`입니다.

```bash
whoami
```

사용자 정보 확인:

```bash
id juneseo
```

### 7.2 `.ssh` 디렉터리 생성

`juneseo` 사용자의 SSH 설정 디렉터리를 생성했습니다.

```bash
sudo mkdir -p /home/juneseo/.ssh
```

### 7.3 `.ssh` 디렉터리 권한 설정

```bash
sudo chmod 700 /home/juneseo/.ssh
```

권한 `700`의 의미는 다음과 같습니다.

| 대상 | 권한 |
|---|---|
| 소유자 | 읽기, 쓰기, 실행 |
| 그룹 | 권한 없음 |
| 기타 사용자 | 권한 없음 |

### 7.4 `.ssh` 디렉터리 소유권 설정

```bash
sudo chown juneseo:juneseo /home/juneseo/.ssh
```

---

## 8. authorized_keys 파일 설정

### 8.1 authorized_keys란?

`authorized_keys`는 해당 계정으로 SSH Key 인증을 허용할 공개키를 저장하는 파일입니다.

이번 실습에서는 다음 경로를 사용했습니다.

```text
/home/juneseo/.ssh/authorized_keys
```

### 8.2 파일 생성

```bash
sudo touch /home/juneseo/.ssh/authorized_keys
```

### 8.3 파일 소유권 설정

처음 파일을 생성했을 때 `root` 소유로 만들어졌기 때문에 소유권을 변경했습니다.

```bash
sudo chown juneseo:juneseo /home/juneseo/.ssh/authorized_keys
```

### 8.4 파일 권한 설정

```bash
sudo chmod 600 /home/juneseo/.ssh/authorized_keys
```

권한 `600`의 의미는 다음과 같습니다.

| 대상 | 권한 |
|---|---|
| 소유자 | 읽기, 쓰기 |
| 그룹 | 권한 없음 |
| 기타 사용자 | 권한 없음 |

최종적으로 다음과 같은 상태가 되도록 설정했습니다.

```text
drwx------  juneseo juneseo  .ssh
-rw-------  juneseo juneseo  authorized_keys
```

---

## 9. Windows에서 SSH Key 생성

Windows PowerShell에서 Ed25519 방식의 SSH Key를 생성했습니다.

```powershell
ssh-keygen -t ed25519 -C "linux-lab"
```

생성 과정에서 저장 경로와 passphrase를 입력합니다.

기본 저장 경로는 다음과 같습니다.

```text
C:\Users\dudrn\.ssh\id_ed25519
```

공개키 파일은 다음 경로에 생성됩니다.

```text
C:\Users\dudrn\.ssh\id_ed25519.pub
```

생성된 파일 확인:

```powershell
Get-ChildItem "$HOME\.ssh"
```

---

## 10. Windows 공개키를 Ubuntu에 등록

Windows와 Ubuntu VM 사이에서 클립보드 복사·붙여넣기가 정상적으로 동작하지 않았습니다.

따라서 공개키 내용을 직접 복사하지 않고, Windows PowerShell에서 SSH 연결을 통해 Ubuntu로 전송했습니다.

### 10.1 공개키 전송 명령어

Windows PowerShell에서 실행합니다.

```powershell
Get-Content "$HOME\.ssh\id_ed25519.pub" | ssh juneseo@192.168.56.10 "mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys; chmod 600 ~/.ssh/authorized_keys"
```

### 10.2 명령어 동작

#### Windows에서 공개키 읽기

```powershell
Get-Content "$HOME\.ssh\id_ed25519.pub"
```

Windows의 공개키 파일을 읽습니다.

#### Ubuntu에 SSH 접속

```powershell
ssh juneseo@192.168.56.10
```

Ubuntu의 `juneseo` 계정으로 접속합니다.

이 단계에서는 공개키 인증이 아직 설정되지 않았기 때문에 Ubuntu 계정 비밀번호를 입력할 수 있습니다.

#### Ubuntu에서 공개키 저장

```bash
mkdir -p ~/.ssh
cat >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Windows에서 전달된 공개키를 Ubuntu의 `authorized_keys`에 추가하고 파일 권한을 설정합니다.

---

## 11. SSH Key 인증 테스트

공개키 등록 후 Windows PowerShell에서 SSH Key 인증만 사용하도록 접속을 시도했습니다.

```powershell
ssh -o PreferredAuthentications=publickey -o PasswordAuthentication=no -i "$HOME\.ssh\id_ed25519" juneseo@192.168.56.10
```

### 옵션 설명

| 옵션 | 의미 |
|---|---|
| `-o PreferredAuthentications=publickey` | 공개키 인증을 우선 사용 |
| `-o PasswordAuthentication=no` | 비밀번호 인증 사용 안 함 |
| `-i` | 사용할 개인키 지정 |
| `juneseo@192.168.56.10` | 접속 계정과 서버 주소 |

### 성공 기준

다음과 같은 Ubuntu 프롬프트가 나타나면 접속 성공입니다.

```text
juneseo@linux-web-01:~$
```

비밀번호를 묻지 않고 접속되면 SSH Key 인증이 정상적으로 동작한 것입니다.

---

## 12. 실습 중 발생한 오류와 해결

### 12.1 ssh-keygen 옵션 오류

처음 Ubuntu에서 다음과 같은 형태로 명령어를 입력했습니다.

```bash
ssh-keygen -t ed25519 -c "linux-lab"
```

`-c` 옵션은 공개키 생성 시 사용하는 일반적인 주석 옵션이 아니므로 오류가 발생했습니다.

정상적인 명령어는 다음과 같습니다.

```bash
ssh-keygen -t ed25519 -C "linux-lab"
```

`-C`는 키에 주석을 추가하는 옵션입니다.

---

### 12.2 잘못된 사용자 홈 디렉터리 사용

처음에는 다음과 같이 잘못된 경로를 사용했습니다.

```text
/home/operator/.ssh
```

실제 작업 대상은 `juneseo` 사용자였으므로 다음 경로를 사용해야 합니다.

```text
/home/juneseo/.ssh
```

Linux에서는 사용자별 홈 디렉터리가 다르므로, 반드시 접속 대상 계정의 홈 디렉터리를 사용해야 합니다.

---

### 12.3 권한 부족 오류

다음과 같은 오류가 발생했습니다.

```text
ls: cannot open directory '/home/juneseo': Permission denied
```

다른 사용자의 홈 디렉터리는 일반 사용자 권한으로 접근할 수 없을 수 있습니다.

관리자 권한으로 확인했습니다.

```bash
sudo ls -la /home/juneseo
```

---

### 12.4 authorized_keys 소유권 문제

처음 `authorized_keys` 파일이 다음과 같이 root 소유였습니다.

```text
-rw-r--r-- 1 root root authorized_keys
```

이를 다음 명령어로 수정했습니다.

```bash
sudo chown juneseo:juneseo /home/juneseo/.ssh/authorized_keys
sudo chmod 600 /home/juneseo/.ssh/authorized_keys
```

최종 상태는 다음과 같습니다.

```text
-rw------- 1 juneseo juneseo authorized_keys
```

---

### 12.5 Windows와 VM 사이의 클립보드 문제

Windows에서 공개키를 복사한 뒤 Ubuntu VM에 붙여넣으려 했지만, VM과 Windows 사이의 복사·붙여넣기가 정상적으로 동작하지 않았습니다.

이를 해결하기 위해 공개키를 SSH 연결을 통해 직접 전송했습니다.

```powershell
Get-Content "$HOME\.ssh\id_ed25519.pub" | ssh juneseo@192.168.56.10 "mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys; chmod 600 ~/.ssh/authorized_keys"
```

---

### 12.6 공개키가 명령어로 실행된 오류

공개키를 잘못된 방식으로 전달하여 Ubuntu에서 다음과 같은 오류가 발생했습니다.

```text
-bash: line 1: ssh-ed25519: command not found
```

이는 공개키 문자열이 파일에 저장되지 않고, Bash 명령어로 해석되었기 때문에 발생한 오류입니다.

이후 공개키를 `cat >> ~/.ssh/authorized_keys`로 전달하는 방식으로 수정했습니다.

---

## 13. 보안 주의사항

### 13.1 개인키는 공개하지 않기

다음 파일은 절대로 GitHub에 업로드하면 안 됩니다.

```text
id_ed25519
```

개인키가 유출되면 해당 키를 이용한 인증이 악용될 수 있습니다.

### 13.2 공개키만 서버에 등록하기

서버에 등록하는 파일은 다음 공개키 파일입니다.

```text
id_ed25519.pub
```

### 13.3 GitHub 업로드 전 확인

GitHub에 업로드하기 전에 다음 파일이 포함되어 있지 않은지 확인합니다.

- `id_ed25519`
- `id_ed25519.pub`
- SSH 설정 파일
- 비밀번호
- 서버 개인 인증 정보
- API Key
- 토큰

공개키는 일반적으로 공개해도 되지만, 실습 저장소에서는 필요하지 않다면 공개키 내용 자체도 올리지 않는 것이 안전합니다.

---

## 14. 최종 결과

이번 모듈 4에서 다음 작업을 완료했습니다.

- [x] Ubuntu SSH 서버 설치 및 실행
- [x] SSH 서비스 상태 확인
- [x] `juneseo` 사용자 확인
- [x] `.ssh` 디렉터리 생성
- [x] `.ssh` 디렉터리 권한 `700` 설정
- [x] `.ssh` 디렉터리 소유권 설정
- [x] `authorized_keys` 파일 생성
- [x] `authorized_keys` 소유권 설정
- [x] `authorized_keys` 권한 `600` 설정
- [x] Windows에서 Ed25519 SSH Key 생성
- [x] Windows 공개키 파일 확인
- [x] 공개키를 Ubuntu에 전송
- [x] SSH Key 인증 접속 성공
- [x] 비밀번호 없이 SSH Key로 원격 접속 확인

---

## 15. 핵심 요약

이번 실습의 핵심은 다음과 같습니다.

> Windows에는 개인키를 보관하고, Ubuntu의 `authorized_keys`에는 공개키를 등록한다.

SSH Key 인증이 정상적으로 동작하려면 다음 조건이 필요합니다.

1. Ubuntu SSH 서비스가 실행 중이어야 합니다.
2. 공개키가 올바른 사용자 계정의 `authorized_keys`에 등록되어야 합니다.
3. `.ssh` 디렉터리와 `authorized_keys`의 권한이 적절해야 합니다.
4. Windows에서 올바른 개인키를 사용해야 합니다.
5. 접속 대상 IP 주소와 사용자명이 정확해야 합니다.

이번 모듈을 통해 Windows에서 Ubuntu Server로 SSH Key 기반 원격 접속을 구성하고 검증했습니다.
