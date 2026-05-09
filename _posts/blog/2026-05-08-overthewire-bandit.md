---
title: "[OverTheWire] Bandit Level 0 ~ 25 전체 풀이"
date: 2026-05-08
categories: [blog]
---


## 시작하기 전에 — WSL 설치 (Windows 사용자)

WSL(Windows Subsystem for Linux)은 Windows에서 리눅스 명령어 환경을 그대로 사용할 수 있는 기능이다.  

**PowerShell을 관리자 권한으로 실행 후:**

```powershell
wsl --install
```

이 후 Ubuntu가 자동 실행되고 사용자 이름과 비밀번호를 설정하면 완료다.

---

## 공통 접속 방법

모든 레벨의 접속 형식은 동일하다:

```bash
ssh bandit{레벨번호}@bandit.labs.overthewire.org -p 2220
```

- 호스트: `bandit.labs.overthewire.org`
- 포트: `2220`
- 초기 비밀번호: 이전 레벨에서 획득한 패스워드

---

## Level 0

### 레벨 목표

SSH를 사용하여 게임 서버에 로그인하는 것이 목표이다.  
호스트는 `bandit.labs.overthewire.org`, 포트는 `2220`, 사용자 이름과 비밀번호 모두 `bandit0`이다.

### 풀이

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# password: bandit0
```
![bandit0](/assets/images/blog/overthewire_bandit/bandit0.png)

ssh는 원격 서버에 접속하는 명령어이다. 

접속 후 OverTheWire의 환영 배너가 표시되면 성공이다.

---

## Level 0 → Level 1

### 레벨 목표

다음 레벨의 비밀번호는 홈 디렉토리에 있는 `readme` 파일에 저장되어 있고 SSH로 bandit1 계정에 로그인하면 된다.

### 풀이

```bash
ls
cat readme
```
![bandit1](/assets/images/blog/overthewire_bandit/bandit1.png)

`readme` 파일을 `cat`으로 읽으면 bandit1의 비밀번호가 나온다.

---

## Level 1 → Level 2

### 레벨 목표

다음 레벨의 비밀번호는 홈 디렉토리에 있는 `-` 라는 이름의 파일에 저장되어 있다.

### 풀이

```bash
ls
cat ./-
```
![bandit2](/assets/images/blog/overthewire_bandit/bandit2.png)

파일 이름이 `-`이므로 `cat -`를 그대로 입력하면 표준입력으로 인식된다.  
`./`를 앞에 붙이거나 절대/상대 경로로 지정해야 한다.

---

## Level 2 → Level 3

### 레벨 목표

다음 레벨의 비밀번호는 홈 디렉토리에 있는 `spaces in this filename` 파일에 저장되어 있다.

### 풀이

```bash
cat ./"spaces in this filename"
```
![bandit3](/assets/images/blog/overthewire_bandit/bandit3.png)
파일 이름에 공백이 포함되어 있으므로 따옴표로 감싸야 한다.

---

## Level 3 → Level 4

### 레벨 목표

다음 레벨의 비밀번호는 `inhere` 디렉토리 안의 숨겨진 파일에 저장되어 있다.

### 풀이

```bash
cd inhere
ls -a
# .hidden 파일 발견

cat ...Hidding-From-You
```
![bandit4](/assets/images/blog/overthewire_bandit/bandit4.png)

숨겨진 파일은 이름이 `.`으로 시작하며 `ls`에는 보이지 않는다.  
`ls -a` 옵션으로 숨긴 파일까지 표시할 수 있다.

---

## Level 4 → Level 5

### 레벨 목표

다음 레벨의 비밀번호는 `inhere` 디렉토리에서 유일하게 사람이 읽을 수 있는(human-readable) 파일에 저장되어 있다.

### 풀이

```bash
cd inhere
file ./*

cat ./-file07
```
![bandit5](/assets/images/blog/overthewire_bandit/bandit5.png)

`-file00` ~ `-file09`까지 10개의 파일 중 ASCII 텍스트 파일을 찾아야 한다.  
`file` 명령어로 파일 형식을 확인할 수 있다.
출력 결과 중 `-file07`가 `ASCII text`라고 표시된 파일임을 확인했고 `cat`으로 읽었다.

---

## Level 5 → Level 6

### 레벨 목표

다음 레벨의 비밀번호는 `inhere` 디렉토리 어딘가에 저장되어 있으며 아래 조건을 모두 만족한다:

- 사람이 읽을 수 있음 (human-readable)
- 크기가 1033 bytes
- 실행 불가능 (not executable)

### 풀이

```bash
cd inhere
find . -size 1033c
cat ./maybehere07/.file2
```
![bandit6](/assets/images/blog/overthewire_bandit/bandit6.png)

현재 디렉토리(.) 아래에서 크기가 1033바이트(-size 1033c)인 파일을 찾았더니 한 번에 나왔다.

---

## Level 6 → Level 7

### 레벨 목표

비밀번호는 서버 어딘가에 저장되어 있으며 아래 조건을 모두 만족한다:

- 소유자: `bandit7`
- 그룹: `bandit6`
- 크기: 33 bytes

### 풀이

```bash
find / -user bandit7 -group bandit6 -size 33c
```
![bandit7-1](/assets/images/blog/overthewire_bandit/bandit7-1.png)

서버 전체에서 검색해야 하므로 `/`부터 탐색했다. 

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```
![bandit7-2](/assets/images/blog/overthewire_bandit/bandit7-2.png)

파일이 너무 많이 떠서 권한 거부 같은 에러 메시지를 `2>/dev/null`로 숨겼다.

---

## Level 7 → Level 8

### 레벨 목표

비밀번호는 `data.txt` 파일에서 "millionth" 단어 옆에 있다.

### 풀이

```bash
grep "millionth" data.txt
```
![bandit8](/assets/images/blog/overthewire_bandit/bandit8.png)

`grep`으로 `millionth`가 포함된 줄만 출력한다.

---

## Level 8 → Level 9

### 레벨 목표

비밀번호는 `data.txt`에서 딱 한 번만 등장하는 줄이다.

### 풀이

```bash
sort data.txt | uniq -u
```
![bandit9](/assets/images/blog/overthewire_bandit/bandit9.png)

`sort` 명령어는 텍스트 파일의 행 단위 내용을 특정 기준에 따라 정렬하여 출력한다. 
`uniq -u`는 인접한 중복 줄 중 한 번만 나온 줄을 출력한다.

---

## Level 9 → Level 10

### 레벨 목표

비밀번호는 `data.txt` 파일 안에서 사람이 읽을 수 있는 문자열 중 하나에 있으며, 그 문자열 앞에는 여러개의 `=` 문자가 붙어있다.

### 풀이

```bash
strings data.txt | grep "="
```
![bandit10](/assets/images/blog/overthewire_bandit/bandit10.png)

`string`을 사용해 바이너리 파일에서 사람이 읽을 수 있는 문자열만 추출한 뒤 `=`로 필터링했다.

---

## Level 10 → Level 11

### 레벨 목표

비밀번호는 `data.txt`에 Base64로 인코딩되어 저장되어 있다.

### 풀이

```bash
base64 -d data.txt
```
![bandit11](/assets/images/blog/overthewire_bandit/bandit11.png)

`base64`로 인코딩된 파일은 -d 옵션을 사용해 디코딩할 수 있다.

---

## Level 11 → Level 12

### 레벨 목표

비밀번호는 `data.txt`에 저장되어 있으며, 모든 대소문자가 13칸씩 이동(ROT13) 되어 있다.

### 풀이

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
![bandit12](/assets/images/blog/overthewire_bandit/bandit12.png)

`tr`는 문자를 치환하는 명령어로, ROT13 변환에 사용했다. A-Z를 N-ZA-M으로, a-z를 n-za-m으로 매핑하여 알파벳을 13글자씩 이동시킨다.

---

## Level 12 → Level 13

### 레벨 목표

비밀번호는 `data.txt`에 있으며, 이 파일은 **반복적으로 압축된 파일의 hexdump**이다.

작업 중 파일을 안전하게 다루기 위해 /tmp 아래에 작업용 디렉토리를 만드는 것을 추천한다.

- mkdir
- mktemp -d  -> tmp 디렉토리 아래에 랜덤한 이름의 안전한 디렉토리를 생성하는 명령어 

### 풀이

```bash
mkdir /tmp/bndlv13
cp data.txt /tmp/bndlv13
cd /tmp/bndlv13
```
![bandit13-1](/assets/images/blog/overthewire_bandit/bandit13-1.png)

작업 디렉토리를 만들고 파일을 복사한다.

복구를 시작해보자.
```bash
xxd -r data.txt > file1
file file1
gunzip file1  #확장자를 모르면 거부된다

mv file1 file1.gz
gunzip file1.gz
```
![bandit13-2](/assets/images/blog/overthewire_bandit/bandit13-2.png)

`xxd`는 바이너리 데이터를 hexadecimal(hex dump) 형태로 변환하거나 복원하는 명령어이다.
`-r` 옵션(reverse)을 사용해 hex dump 형태의 데이터를 원래 바이너리 파일로 복원했다.

![bandit13-3](/assets/images/blog/overthewire_bandit/bandit13-3.png)

`gunzip`은 gzip 형식이 아니면 압축 해제를 할 수 없다.
`file` 명령어로 파일 형식을 확인하고 `gunzip` 명령으로 gzip 압축 파일을 해제했다.
![bandit13-4](/assets/images/blog/overthewire_bandit/bandit13-4.png)

```bash
# gzip compressed data → mv data data.gz && gzip -d data.gz
# bzip2 compressed data → mv data data.bz2 && bzip2 -d data.bz2
# POSIX tar archive → tar -xvf data.tar
```
`file1`을 확인해 보면 `bzip`으로 압축되어 있는데 위 과정을 반복하면 된다. `tar`로 압축되어 있어도 동일한 방식으로 계속 풀면 된다.
`ASCII text`가 나올 때까지 반복한 뒤 `cat`으로 읽으면 된다.

---

## Level 13 → Level 14

### 레벨 목표

다음 레벨의 비밀번호는 `/etc/bandit_pass/bandit14`에 있으며, bandit14 사용자만 읽을 수 있다. 

비밀번호를 직접 얻는 것이 아니라, 다음 레벨에 접속할 수 있는 **SSH 개인키**가 있어 이를 이용해 로그인할 수 있다.

### 풀이

```bash
ls -al
# sshkey.private 발견

file sshkey.private
cat sshkey.private
```
![bandit14-1](/assets/images/blog/overthewire_bandit/bandit14-1.png)

`sshkey.private` 내용을 확인해보면 `RSA` 형식의 `SSH 개인 키(private key)`라는 것을 알 수 있다.

```bash
ssh -i [private key 파일] [username]@[hostanme] [포트]
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```
해당 키를 이용해 SSH 로그인을 시도했지만 접속되지 않았다.

그래서 키 파일을 다운로드한 뒤 인증에 사용해 보기로 했다.
```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
```
![bandit14-2](/assets/images/blog/overthewire_bandit/bandit14-2.png)

`scp`는 원격 서버의 파일을 로컬로 복사하는 명령어이다. `.`은 현재 디렉토리에 저장한다.

```bash
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220 
```
![bandit14-3](/assets/images/blog/overthewire_bandit/bandit14-3.png)

다음 레벨에 바로 접속할 수 있다.

---

## Level 14 → Level 15

### 레벨 목표

`localhost:30000` 포트에 현재 레벨의 비밀번호를 전송하면 다음 레벨 비밀번호를 받을 수 있다.

### 풀이

```bash
cat /etc/bandit_pass/bandit14
```
![bandit15-1](/assets/images/blog/overthewire_bandit/bandit15-1.png)

`/etc/bandit_pass/bandit14` 파일에서 현재 레벨의 비밀번호를 확인했다.

```bash
telnet localhost 30000
```
![bandit15-2](/assets/images/blog/overthewire_bandit/bandit15-2.png)

`telnet`으로 해당 포트에 접속한 뒤 비밀번호를 입력하면, 서버가 다음 레벨의 비밀번호를 응답으로 반환한다.

---

## Level 15 → Level 16

### 레벨 목표

`localhost:30001`에 **SSL 암호화**를 사용하여 현재 레벨의 비밀번호를 전송하면 다음 레벨 비밀번호를 받을 수 있다.

### 풀이

```bash
opeenssl s_client [옵션] [호스트:포트]
openssl s_client -connect localhost:30001
```
![bandit16-1](/assets/images/blog/overthewire_bandit/bandit16-1.png)

`openssl s_client`는 `SSL/TLS`로 암호화된 서버에 접속하는 클라이언트 도구이다.

![bandit16-2](/assets/images/blog/overthewire_bandit/bandit16-2.png)

연결 정보가 출력된 후 비밀번호를 입력하면 다음 레벨 비밀번호가 출력된다.

---

## Level 16 → Level 17

### 레벨 목표

`31000~32000` 포트 범위 내에서 현재 레벨 비밀번호를 제출하면 다음 레벨 자격증명(SSH 개인키)을 받을 수 있는 포트가 하나 존재한다.

### 풀이

```bash
nmap -sV localhost -p 31000-32000
```
![bandit17-2](/assets/images/blog/overthewire_bandit/bandit17-2.png)

`nmap`은 네트워크 포트 스캔 도구이다. `-sV`는 해당 포트에서 실행 중인 서비스 버전을 확인한다.
31790 포트로 연결해봤다.

```bash
openssl s_client -connect localhost:31790
```
KEYUPDATE라고 뜨면서 종료되었다. 의도치 않은 작동을 방지하기 위해서, 해당 기능등을 비활성화하는 옵션을 추가로 끝에 입력했다.

```bash
openssl s_client -connect localhost:31790 -ign_eof
```
![bandit17-3](/assets/images/blog/overthewire_bandit/bandit17-3.png)

`-ign_eof`는 입력이 끝나도 연결을 바로 종료하지 않도록 한다.

![bandit17-4](/assets/images/blog/overthewire_bandit/bandit17-4.png)
read R BLOCK 메시지가 출력된 뒤 비밀번호를 입력하면, 응답으로 RSA 개인키가 온다.

```bash
mktemp -d  # 랜덤한 디렉토리 생성
cd /tmp/tmp.fIycgVlGU9

vim key
```
![bandit17-5](/assets/images/blog/overthewire_bandit/bandit17-5.png)
![bandit17-7](/assets/images/blog/overthewire_bandit/bandit17-7.png)

해당 내용을 파일로 저장한 뒤 개인 키 값 전체를 넣어준다.

![bandit17-6](/assets/images/blog/overthewire_bandit/bandit17-6.png)

이후 생성한 키 파일을 로컬로 다운로드하고, 이전 레벨(13→14)에서와 같이 `ssh -i` 옵션을 사용해 다음 레벨 인증에 활용할 수 있다.

---

## Level 17 → Level 18

### 레벨 목표

홈 디렉토리에 `passwords.old`와 `passwords.new` 두 파일이 있다.  
비밀번호는 `passwords.new`에서 변경된 유일한 한 줄 이다.

### 풀이

```bash
diff passwords.old passwords.new
```
![bandit18](/assets/images/blog/overthewire_bandit/bandit18.png)

`>`로 표시된 줄(`passwords.new`에만 있는 줄)이 bandit18의 비밀번호이다.

---

## Level 18 → Level 19

### 레벨 목표

비밀번호는 홈 디렉토리의 `readme` 파일에 있다.  
하지만 `.bashrc`가 수정되어 로그인 즉시 로그아웃 된다.

### 풀이

SSH 접속 시 셸이 실행되기 전에 명령어를 직접 전달하는 방식을 사용해야한다.

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```
![bandit19-2](/assets/images/blog/overthewire_bandit/bandit19-2.png)

로그인 후 `.bashrc` 들어가기 전에 바로 `cat readme` 실행되기 때문에 logout 트랩 우회가 가능하다.

---

## Level 19 → Level 20

### 레벨 목표

홈 디렉토리에 **setuid 바이너리**를 사용해야 한다. 비밀번호는 `/etc/bandit_pass` 에 있다.

### 풀이

```bash
ls -la
# bandit20-do 파일 확인 (소유자: bandit20, setuid 설정됨)

./bandit20-do cat /etc/bandit_pass/bandit20
```
![bandit20-1](/assets/images/blog/overthewire_bandit/bandit20-1.png)
![bandit20-2](/assets/images/blog/overthewire_bandit/bandit20-2.png)

`ls -al` 를 이용해 `bandit20-do` 파일의 권한을 확인했다. `setuid`가 설정되어 있으며, 파일 소유자가 `bandit20` 인 것을 알 수 있다.

`-rwsr-x---`에서 `s`는 setuid를 의미하며, 이 파일은 실행 시 실행한 사용자가 아니라 파일 소유자인 `bandit20`의 권한으로 동작한다.

![bandit20-3](/assets/images/blog/overthewire_bandit/bandit20-3.png)

---

## Level 20 → Level 21

### 레벨 목표

홈 디렉토리의 setuid 바이너리는 지정한 포트에 연결해 입력받은 문자열을 bandit20의 비밀번호와 비교하고, 일치하면 bandit21의 비밀번호를 출력한다.

### 풀이

![bandit21-1](/assets/images/blog/overthewire_bandit/bandit21-1.png)

`suconnect` 라는 `ELF` 실행 파일을 확인했다. ELF(Executable and Linkable Format)는 Linux에서 사용되는 실행 파일 형식이다.
프로그램을 인자 없이 실행하자 연결할 포트 번호를 입력하라는 메시지가 출력되었다.

```bash
nd -lvp 12345
```
```bash
./suconnect 12345
```
![bandit21-2](/assets/images/blog/overthewire_bandit/bandit21-2.png)
![bandit21-3](/assets/images/blog/overthewire_bandit/bandit21-3.png)

`nc`로 포트를 열어 비밀번호를 대기시키고, 다른 창에서 `suconnect`가 연결하도록 했다.  

![bandit21-4](/assets/images/blog/overthewire_bandit/bandit21-4.png)
![bandit21-5](/assets/images/blog/overthewire_bandit/bandit21-5.png)

포트를 연 서버 측에서 bandit20의 비밀번호를 입력하면, 응답으로 bandit21의 비밀번호를 받을 수 있다.

---

## Level 21 → Level 22

### 레벨 목표

`cron`이라는 시간 기반 작업 스케줄러가 주기적으로 프로그램을 실행하고 있다. `/etc/cron.d/`를 확인해 어떤 명령어가 실행되는지 찾아야 한다.

### 풀이

![bandit22-1](/assets/images/blog/overthewire_bandit/bandit22-1.png)

`/etc/cron.d/` 디렉토리를 확인하면 `cronjob_bandit22` 파일이 있다.

```bash
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```
`cronjob_bandit22`에서 위와 같은 cron 설정이 들어 있다.

스크립트 내용:
- `@reboot bandit22 ...` → 시스템이 재부팅될 때 bandit22 사용자 권한으로 해당 스크립트 실행
- `* * * * * bandit22 ...` → 매 1분마다 bandit22 사용자 권한으로 해당 스크립트 실행
- `/usr/bin/cronjob_bandit22.sh` → 실제로 실행되는 스크립트 파일
- `&> /dev/null` → 실행 결과(표준 출력 + 에러 출력)를 모두 버림

즉, 이 cron 설정은 bandit22 권한으로 특정 스크립트를 재부팅 시와 매 1분마다 실행하며, 출력은 모두 숨긴다.

`/usr/bin/cronjob_bandit22.sh`의 내용을 확인해보자.
![bandit22-2](/assets/images/blog/overthewire_bandit/bandit22-2.png)
```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```
cron이 bandit22 비밀번호를 읽어 `/tmp`에 저장하고, 해당 파일을 읽기 가능하도록 권한을 변경하는 스크립트이다.

해당 파일의 내용을 읽으면 bandit22 비밀번호가 나온다.

---

## Level 22 → Level 23

### 레벨 목표

이전 레벨과 같이 `cron` 프로그램이 실행되고 있으며 `/etc/cron.d/`를 확인해 어떤 명령어가 실행되는지 찾으면 된다.

### 풀이

![bandit23-1](/assets/images/blog/overthewire_bandit/bandit23-1.png)
```bash
cat cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh
```

`cronjob_bandit23.sh` 스크립트 내용:
```bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```
이 스크립트는 현재 사용자 이름을 기반으로 MD5 해시 파일명을 생성한 뒤, 해당 사용자의 비밀번호를 `/tmp` 디렉토리에 저장한다.

```bash
echo "I am user bandit23" | md5sum
cat /tmp/[계산된 해시값]
```
![bandit23-3](/assets/images/blog/overthewire_bandit/bandit23-3.png)

`echo "I am user bandit23" | md5sum` 명령을 사용해 생성되는 MD5 해시 값을 계산한 뒤,
해당 값이 `/tmp/`의 파일 이름이 되므로 그 파일을 찾아 비밀번호를 확인한다.

---

## Level 23 → Level 24

### 레벨 목표

이전 레벨과 같이 `cron` 프로그램이 실행되고 있으며 `/etc/cron.d/`를 확인해 어떤 명령어가 실행되는지 찾으면 된다.
이번 레벨에서는 직접 셸 스크립트를 만들어야 하며, 작성한 스크립트는 60초 후 삭제된다.

### 풀이

![bandit24-1](/assets/images/blog/overthewire_bandit/bandit24-1.png)

```bash
cat /usr/bin/cronjob_bandit24.sh
```
![bandit24-2](/assets/images/blog/overthewire_bandit/bandit24-2.png)

스크립트 내용:
```bash
#!/bin/bash
myname=$(whoami)
cd /var/spool/"$myname"/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." ] && [ "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" "./$i")"
        if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
            timeout -s 9 60 "./$i"
        fi
        rm -rf "./$i"
    fi
done
```
이 스크립트는 `/var/spool/$USER/foo` 디렉토리 안의 파일을 검사한 뒤,
소유자가 bandit23인 파일만 실행하고, 실행이 끝나면 해당 파일을 삭제하는 cron 스크립트이다.

![bandit24-3](/assets/images/blog/overthewire_bandit/bandit24-3.png)
`others(-wx)` → foo 디렉토리에 파일은 생성할 수 있지만 읽기는 불가능하다.

![bandit24-4](/assets/images/blog/overthewire_bandit/bandit24-4.png)

나만의 스크립트를 작성했다:
```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/tmp.SjIKXW6utu/password.txt
```
![bandit24-5](/assets/images/blog/overthewire_bandit/bandit24-5.png)

![bandit24-6](/assets/images/blog/overthewire_bandit/bandit24-6.png)

1분 후 결과를 확인한다.

---

## Level 24 → Level 25

### 레벨 목표

`localhost:30002` 포트에서 데몬이 실행 중이다.  
bandit24의 비밀번호 + 4자리 핀코드를 전송하면 bandit25의 비밀번호를 받을 수 있다.  
핀코드는 `0000~9999` 중 하나로, **브루트포스** 를 이용하면 된다. 

### 핵심 개념: 브루트포스 (Brute Force)

가능한 모든 조합을 시도하는 공격 기법이다.  
핀코드가 4자리(10,000가지)로 제한되어 있어 스크립트로 빠르게 자동화할 수 있다.

### 풀이

![bandit25-1](/assets/images/blog/overthewire_bandit/bandit25-1.png)

```bash
for i in {0000..9999}; do
    echo "$PASSWORD $i"
done | nc localhost 30002

PASSWORD="여기에_bandit24_비밀번호_입력"
```
![bandit25-2](/assets/images/blog/overthewire_bandit/bandit25-2.png)

![bandit25-3](/assets/images/blog/overthewire_bandit/bandit25-3.png)

`Correct! The password of user bandit25 is ...` 라는 줄에서 비밀번호를 확인하면 된다.

---
