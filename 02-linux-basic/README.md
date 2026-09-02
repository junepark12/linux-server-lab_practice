# 모듈 2. Linux 기본 명령과 파일 관리

## 1. 실습 목적

Linux 서버에서 파일, 디렉터리, 사용자, 관리자 권한, 프로세스를 터미널 명령어로 확인한다.

## 2. 핵심 개념

Linux 서버는 대부분 CLI 환경에서 운영한다.  
서버 관리자는 현재 위치, 파일 목록, 권한, 사용자, 실행 중인 프로세스를 명령어로 확인할 수 있어야 한다.

## 3. 실습 명령어

```bash
pwd
ls
ls -al
mkdir linux-practice
cd linux-practice
touch test.txt
echo "hello linux" > test.txt
cat test.txt
cp test.txt copy.txt
mv copy.txt renamed.txt
rm renamed.txt
whoami
sudo whoami
ps aux
top

## 4. 명령어 정리

| 명령어 | 기능 |
|---|---|
| `pwd` | 현재 내가 위치한 디렉터리 경로를 출력한다. |
| `ls` | 현재 디렉터리의 파일과 폴더 목록을 보여준다. |
| `ls -al` | 숨김 파일을 포함해 권한, 소유자, 크기, 수정 시간을 자세히 보여준다. |
| `mkdir linux-practice` | `linux-practice`라는 새 디렉터리를 만든다. |
| `cd linux-practice` | `linux-practice` 디렉터리로 이동한다. |
| `touch text1.txt` | `text1.txt`라는 빈 파일을 생성한다. |
| `echo "hello linux" > text1.txt` | 문자열을 `text1.txt` 파일에 저장한다. 기존 내용은 덮어쓴다. |
| `cat text1.txt` | `text1.txt` 파일의 내용을 터미널에 출력한다. |
| `cp text1.txt copy.txt` | `text1.txt`를 `copy.txt`라는 이름으로 복사한다. |
| `mv copy.txt renamed.txt` | `copy.txt`의 이름을 `renamed.txt`로 변경한다. |
| `rm renamed.txt` | `renamed.txt` 파일을 삭제한다. |
| `tail -n 5 text1.txt` | `text1.txt`의 마지막 5줄을 출력한다. |
| `whoami` | 현재 로그인한 사용자 이름을 출력한다. |
| `sudo whoami` | 관리자 권한으로 명령을 실행했을 때의 사용자 이름을 확인한다. |
| `ps aux` | 현재 실행 중인 프로세스 목록을 자세히 보여준다. |
| `ps aux \| head` | 실행 중인 프로세스 목록 중 앞부분만 출력한다. |
| `top` | CPU, 메모리, 프로세스 상태를 실시간으로 보여준다. |

## 5. `ls -al` 결과 해석

예시:

```bash
-rw-rw-r-- 1 kahuna kahuna 0 9월 2 15:20 text1.txt
