---
layout: post
title:  "WSL Docker install"
date:   2022-07-10 10:00:00 +0900
categories: Environment WSL Docker
---
Reference
> <https://hbase.tistory.com/299>
> <https://blog.dalso.org/linux/wsl2/11306>
  
### Systemctl for WSL2 (Optional)
Docker나 그 외 service를 설치하다보면 systemctl을 기반으로 한 명령어를 많이 볼 수 있음. 하지만 WSL에서는 기본적으로 systemctl을 지원하지 않음.  
때문에 문제가 발생했을 때 stackoverflow 등에 나와 있는 방법으로 확인하기가 어려운 경우가 많아 아래 page를 보고 systemctl을 활성화 시키는 것을 추천.  
> <https://gist.github.com/djfdyuruiry/6720faa3f9fc59bfdf6284ee1f41f950>
  
설치 과정에서 아래와 같은 error가 발생한다.
  
```console
Timed out waiting for systemd to enter running state.
This may indicate a systemd configuration error.
Attempting to continue.
Failed units will now be displayed (systemctl list-units --failed):
  UNIT                       LOAD   ACTIVE SUB    DESCRIPTION
● ssh.service                loaded failed failed OpenBSD Secure Shell server
● systemd-remount-fs.service loaded failed failed Remount Root and Kernel File Systems
● multipathd.socket          loaded failed failed multipathd control socket
  
LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
  
3 loaded units listed.
```
ssh의 경우 사용할 것이기 때문에 설치하고 나머지 service는 불필요하기 때문에 비활성화하고 진행한다.
```console
$ sudo systemctl mask multipathd.socket & sudo systemctl mask systemd-remount-fs.service & sudo apt install openssh-server
```  
  
### Docker installation
WSL에 Ubuntu 20.04 LTS를 설치한 상태로 진행 함.  
  
#### 1) Install package  
```console
$ sudo apt install ca-certificates curl gnupg lsb-release
```
  
#### 2) GPG Key 인증  
curl로 GPG key 등록  
  
```console
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
> OK
```
  
Finger print 확인  
  
```console
$ sudo apt-key fingerprint 0EBFCD88
> pub   rsa4096 2017-02-22 [SCEA]
>       9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
> uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
> sub   rsa4096 2017-02-22 [S]
```
  
#### 3) Docker repository 등록  
```console
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
  
#### 4) Docker install  
```console
$ sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io
  
$ docker --version
> Docker version 20.10.17, build 100c701
  
$ sudo service docker start
> * Starting Docker: docker   [ OK ]
```
  
### Portainer installation
Portainer는 Docker의 image, container 등의 관리를 도와주는 Web GUI.  
- Docker를 설치한 후부터는 su 권한을 가지고 진행함.  
  
#### 1) Container <> Host간 volumn 설정을 위한 path 생성
```console
# mkdir -p ${SHARED_VOLUMN_PATH}
```
$\{SHARED_VOLUMN_PATH\} = /data/portainer  
  
#### 2) Container 실행
```console
# docker run --name portainer -p 9000:9000 -d --restart always -v /data/portainer:/data -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```
--name: container의 이름 설정  
-p: ${HOST_PORT}:${DOCKER_PORT}  
-d: Deamon으로 background로 수행  
--restart always: 재부팅시 자동 시작  
-v: Container <> Host 간 volumn match  
  
제대로 설치가 되었는지 확인 -> docker-proxy가 확인되면 정상  
```console
# netstat -lntp
> Active Internet connections (only servers)
> Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
> tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      11318/docker-proxy
> tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      221/systemd-resolve
> tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2303/sshd: /usr/sbi
> tcp6       0      0 :::9000                 :::*                    LISTEN      11325/docker-proxy
> tcp6       0      0 :::22                   :::*                    LISTEN      2303/sshd: /usr/sbi
```
#### 3) Portainer 설정
이후 browser로 접근하면 아래와 같이 초기 page가 나타난다.
- 127.0.0.1:9000
  
관리자 계정으로 사용할 계정명과 비밀번호를 입력하고 create user를 눌려준다.  
  
![Image Alt 텍스트](/img/220710_portainer_00.png)
  
사용할 container에 맞춰서 선택. (여기서는 Docker)  
![Image Alt 텍스트](/img/220710_portainer_01.png)
  
정상 설치 시 볼 수 있는 화면  
![Image Alt 텍스트](/img/220710_portainer_02.png)  
  
***
  