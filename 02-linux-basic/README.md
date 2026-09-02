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
