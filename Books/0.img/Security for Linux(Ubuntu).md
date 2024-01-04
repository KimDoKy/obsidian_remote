---
tags:
  - devops
  - security
  - linux
---
## 계정 및 권한관리
### 1.1 패스워드 복잡도가 설정되었는가?
### 1.2 패스워드 재사용에 대한 보안설정이 적용되었는가?
### 1.3 패스워드 잠금 정책에 대한 보안설정이 적용되었는가?
```bash
# 패스워드 복작도 설정
# 패스워드의 최소 암호 길이
# 패스워드 재사용
# 최근 암호 기억
# /etc/pam.d/common-password
password        [success=1 default=ignore]      pam_unix.so obscure sha512 remember=2 minlen=8 dcredit=-1 ucredit=-1 lcredit=-1 ocredit=-1

# 최대/최소 사용기간
# /etc/login.defs
PASS_MAX_DAYS 90
PASS_MIN_DAYS 7

# 패스워드 잠금 정책
# /etc/pam.d/common-auth

auth    required                        pam_tally2.so deny=5 unlock_time=1800 even_deny_root
```

### 1.4 주요 시스템 파일에 대한 접근 권한이 제한되고 있는가?
```bash
# A. 사용자 계정, 그룹 리스트 정보 파일에 대한 접근 권한이 제한되어 있는가?
# /etc/passwd /etc/group의 권한 644
# 기본값이라 따로 설정할 필요 없지만, 확인 필요
ls -al /etc/passwd /etc/group

# B. 사용자 계정 암호 파일에 대한 접근 권한이 제한되어 있는가?
# 실행하여 소유자 [root], 퍼미션 [400] 확인
ls -al /etc/shadow
sudo chmod 400 /etc/shadow
# sudo chgrp root /etc/shadow

# 계정의 패스워드 해시값이 존재하는 지 확인
cat /etc/passwd

# C. /etc/hosts 파일에 대한 접근 권한이 제한되어 있는가?
# 소유자 [root], 퍼미션 [644] 확인
# 기본값이라 따로 설정할 필요 없지만, 확인 필요
ls -al /etc/hosts

# D. /etc/services 파일에 대한 접근 권한이 제한되어 있는가?
# 소유자 [root], 퍼미션 [644] 확인
# 기본값이라 따로 설정할 필요 없지만, 확인 필요
ls -al /etc/services

# E. 일반 사용자가 중요 명령어를 실행하지 못하도록 설정되었는가?
# 소유자 [root] 또는 [sys], [bin], 퍼미션 [700] 확인 
ls -l /usr/bin/last
sudo chmod 700 /usr/bin/last

# F. 사용자 환경 설정 파일에 대한 접근 권한이 제한되어 있는가?
# 소유자 [root], 퍼미션 [755] 
ls -al /etc/profile
sudo chmod 755 /etc/profile
# 소유자 [소유자 계정], 퍼미션 [소유자 이외 쓰기 권한 없음] 
## cicd의 경우 ubuntu, jenkins 2개의 계정 사용중. 모두 해당 보안 적용됨
# 기본값이라 따로 설정할 필요 없지만, 확인 필요
awk -F ":" '{print $6"/.profile"}' /etc/passwd | xargs ls –al

# J. 패스워드 규칙 설정 파일에 대한 접근 권한이 제한되어 있는가?
# 소유자 [root/bin], 타 사용자 [쓰기권한 없음] 확인
# 기본값이라 따로 설정할 필요 없지만, 확인 필요
ls -l /etc/pam.d/common-auth

# L. 주요 백업 파일 접근 권한이 제한되어 있는가?
# 백업 목적 등으로 유사이름으로 존재 시 (passwd.old, passwd.050523 등) 소유자 [root], 퍼미션 [600] 확인
# 당연히 존재하지 않음
ls -al /etc/passwd* /etc/xinetd.conf* /etc/services* /etc/hosts* /var/adm/wtmp* /var/adm/btmp* /var/adm/sulog*
```

### 1.5 접근 권한 부여를 위한 기반 보안 설정이 적용되어 있는가?
```bash
# A. root 계정의 UMASK 설정은 적절한가?
# root 계정으로 로그인 후 [umask] 실행하여 [022] 또는 [027] 확인
# 기본값이라 따로 설정할 필요 없지만, 확인 필요
sudo su
umask
0022

# B. 일반 계정의 umask 설정이 적절한가?
# 각 알반 계정으로 로그인 한 후 [#umask]를 통해 root 계정의 [umask 설정]이 0022 또는 0027 인지 확인
umask
sudo vi /etc/profile
## /etc/profile
...
if [ "${PS1-}" ]; then
  umask 022           <--- 여기 삽입
  if [ "${BASH-}" ] && [ "$BASH" != "/bin/sh" ]; then
...
## 재접속 후 umask 확인
umask
0022
```

### 1.6 일정 기간 미입력 시 Session Timeout 을 적용하고 있는가?
```bash
# A. Session Timeout을 적용하고 있는가?
# /etc/profile]파일에서 [TMOUT] 항목 확인 
sudo vi /etc/profile
# 마지막에 export TMOUT=1800 추가
```

### User 생성 및 SSH 접속 셋팅
```bash
# user 생성
sudo adduser [username]

# 생성한 유저에 sudo 권한 부여
sudo usermod -aG sudo [username]

# sudo 권한 적용 확인
sudo -u [username] whoami

# user 로그인
su - [username]

# ssh 접속 셋팅
mkdir .ssh
vi authorized_keys
## 여기에 공개키를 등록한다.

# ssh port 번호 바꾸기
## root 권한 계정으로만 작업이 가능함 ex. ubuntu
sudo vi /etc/ssh/sshd_config
# Port 22 이 부분의 주석을 해제하고 포트번호를 변경한다
# ssh 재시작
sudo systemctl restart ssh

# AWS SG에서 해당 포트를 열어주어야 함
```