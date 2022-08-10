# dockerinstall
## CentOS7 환경에서 관리자 권한(root)으로 docker를 설치해보았다
## docker 라는 것에 대해 알게 되었다 
- docker 란 ? Docker란 Go언어로 작성된 리눅스 컨테이너 기반으로하는 오픈소스 가상화 플랫폼이다.



``` 
사내 사용자 매뉴얼 사이트 제작 예정
전산과 개발로 나누어 제작 기획 중
상무님께서 docs.niceamc.co.kr 도메인으로 docker에 gitlab 구축 임무 주심 
대리님과 함께 수행
docker가 설치되어있는 centos7환경에 원격으로 붙어서 진행하였음
gitlab에서 사용하는 포트는 3가지로, 443-https , 80:http, 22:ssh가 있다고함 
여기에 맞춰서 사내 정책으로 대리님께서 열어주신 포트인 8000번에서 10000번 사이 포트와 80번에서 90사이 포트 중 사용할 수 있는 포트번호들을 각각 지정해준 부분이다
10.115.224.210 서버에서 진행하였다
이름은 gitlab_docs로, 도메인은 docs.niceamc.co.kr로 설정해준 부분입니다
(아래 volume부분은 생략이 가능할 수도 있을 것 같다고 테스트 추후에 진행해보기로 하였습니다)
크게는 82번 포트를 사용
만약 다른 사이트를 하나더 구축하려면 이름과 포트번호를 사용할 수 있는 것으로 아래 코드 부분을 변경하여 진행하면 될 것 같습니다
``` 
- 구축 명령 
``` 
docker run -e VIRTUAL_HOST=docs.niceamc.co.kr --expose 82 --detach --hostname 10.115.224.210 --publish 8442:443 --publish 82:80 --publish 822:22 --name gitlab_docs --restart always --volume /srv/gitlab/config:/etc/gitlab_docs --volume /srv/gitlab/logs:/var/log/gitlab_docs --volume /srv/gitlab/data:/var/opt/gitlab_docs gitlab/gitlab-ce:latest
```
- 구축 후 초기 비밀번호 생성
```
> docker exec -it [container_id] /bin/bash

# gitlab-rails console -e production

> user = User.where(id: 1).first

> user.password='변경할비밀번호'
> user.password_confirmation='변경할비밀번호'

user.save -> 8자리 이상 설정이 되어야 true 반환 -> 12345678 로 설정 완료
```
#
- docker 진행 시 자주 사용하는 명령
```
docker ps -a  -> docker 현 상태 (올라와있는지 등등) 확인 할 수 있는 명령
docker service start  -> docker 시작
docker service restart -> 재시작
```




- 새로 docs.niceamc.co.kr 사내 매뉴얼 깃랩 구축을 위해 docker에 하나 더 깃랩 이미지를 올린 명령입니다
```
docker run -it --detach --name py.niceamc.co.kr --hostname py.niceamc.co.kr --expose 81 --network nginx-proxy -e VIRTUAL_HOST=py.niceamc.co.kr -e VIRTUAL_HOST=py.niceamc.co.kr gitlab/gitlab-ce
-> 이 명령으로 올라간 py.niceamc.co.kr 은 사내에서 접속 시 파이썬 교육자료들이 있으며 파이썬 교육을 들을 수 있는 페이지입니다.
-> 페이지 도메인을 py.niceamc.co.kr 로 지정하여 올렸습니다. 


docker run -it --detach --name docs.niceamc.co.kr --hostname docs.niceamc.co.kr --expose 82 --network nginx-proxy -e VIRTUAL_HOST=docs.niceamc.co.kr -e VIRTUAL_HOST=docs.niceamc.co.kr gitlab/gitlab-ce
-> 이 명령으로 올라간 docs.niceamc.co.kr 은 앞으로 제작하게 될 사내 개발, 전산 매뉴얼 페이지입니다.
-> 페이지 도메인을 docs.niceamc.co.kr 로 지정하여 올렸습니다.

-> 이렇게 두개를 docker에서 gitlab/gitlab-ce 이미지를 docker hub 에서 pull 해와 두가지를 올린 과정입니다. 
-> 먼저 py.niceamc.co.kr 도커 이미지가 먼저 올라와있었는데 docs.niceamc.co.kr 이미지를 올리는 과정에서 기존의 이미지가 없어지는 사고가 발생하였습니다.

원인 : 이미지는 여러개 올릴 수 있는데 과정에서 포트 충돌이 원인이었습니다. 
      포트 80 http를 사용하여 py.niceamc.co.kr 도메인에 접속하고 있었는데 80으로 또다시 docs.niceamc.co.kr을 올림으로써 포트 충돌이 일어난 것이 원인이 되었습니다.
      1개의 포트는 1개의 서비스를 담당해줍니다.
      
      이러한 부분을 해결하기 위해 nginx-proxy를 사용하기로 해주었습니다.
      nginxproxy/nginx-proxy 이미지를 불러와 이를 80번 포트로 서비스해주었습니다. 
      이 nginx-proxy는 포트 분배를 해준다고 생각하면 쉽게 개념이 다가옵니다. 
      즉 위 수행한 명령에서 볼 수 있듯이 --network nginx-proxy 명령을 추가하면서 이 명령으로 인해 py.niceamc.co.kr은 81번 포트로, docs.niceamc.co.kr 은 82번 포트로 nginx-
      proxy가 지정을 해주어서 분배가 되어 두 사이트가 충돌없이 서비스 될 수 있습니다.

```
#Ubuntu 관련 설치 명령 기록 
```
* Ubuntu 18.04 환경에서 docker 설치 수행 명령

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ununtu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt-get update
sudo apt-get install docker-ce

다음 명령어들로 docker 설치 완료하였습니다.
```
```
* Ubuntu 18.04 환경에 원격으로 붙기 위한 vnc 설치 과정(일반사용자 계정niceadmin으로 루트위치에서 수행)

- vnc란? VNC는 Virtual Network Computing 데스크톱 시스템을 원격으로 공유하기 위한 프로토콜 집합으로 TigerVNC, TightVNC, Vino, vnc4server 등을 포함하여 Linux 기반 데스크탑에 원격으로 액세스하는 데 사용할 수 있는 소프트웨어가 많이 있다.
우리는 그 중 TigerVNC를 설치하여 사용할 것이다. TigerVNC는 Linux 기반 데스크탑 시스템을 원격으로 제어하거나 액세스하는 데 사용되는 무료 오픈 소스 고성능 VNC 서버로 원격 시스템의 그래픽 응용 프로그램과 상호 작용할 수 있는 클라이언트/서버 응용 프로그램이다.

sudo apt update  #업데이트
sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils #우분투 리포지토리로 사용할 수 있는 데스크톱 환경인 Xfce 설치
sudo apt install tigervnc-standalone-server tigervnc-common   #다양한 vnc 중에서 tigerVNC 서버 설치
vncserver  # 암호 설정 부분이 나옵니다(꼼수2122!로 설정) -> n으로 대답해주어야함(이유 : 보기 전용 암호로 설정할지 여부를 묻는 메시지가 표시되는데 보기 전용 암호를 설정하도록 선택하면 사용자가 마우스 및 키보드를 사용하여 VNC 인스턴스와 상호 작용할 수 없다.여기서 나는 섣부르게 y를 설정하였다가 vnspasswd 를 쳐서 다시 비번을 치고 n으로 설치하였다.(시행착오))
vncserver -kill :1   #위 암호를 설치하면 호스트 이름 뒤에 :1이 있는데 이는 vnc서버가 실행 중인 디스플레이 포트 번호를 나타낸다.이 부분에 대한 내용은 아래에 따로 설명을 추가하겠습니다. 
nano ~/.vnc/xstartup   # Xfce를 사용하도록 TigerVNC를 구성하기 위한 파일을 만들기 ~/.vnc/xstartup

파일안에서 다음과 같은 내용 추가 
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4 



다음 명령어들로 vnc 설치 완료하였습니다. (이후 원격으로 붙을 수 있었습니다)
```
```
# You will require a password to access your desktops.
# 
# Password:
# Verify:
# Would you like to enter a view-only password (y/n)? n
# /usr/bin/xauth:  file /home/linuxize/.Xauthority does not exist
# 
# New 'server2.linuxize.com:1 (linuxize)' desktop at :1 on machine server2.linuxize.com
# 
# Starting applications specified in /etc/X11/Xvnc-session
# Log file is /home/linuxize/.vnc/server2.linuxize.com:1.log
# 
# Use xtigervncviewer -SecurityTypes VncAuth -passwd /home/linuxize/.vnc/passwd :1 to connect to the VNC server.
 

vncserver 명령을 처음 실행하면 암호 파일이 생성되어 ~/.vnc에 저장됩니다. 이 디렉토리는 존재하지 않는 경우 작성됩니다. 
위의 출력에서 호스트 이름 뒤에 :1이 있습니다. vnc 서버가 실행 중인 디스플레이 포트 번호를 나타냅니다. 이 경우 서버는 TCP 포트 5901(5900+1)에서 실행되고 있습니다. vncserver를 사용하여 두 번째 인스턴스를 생성하면 다음 사용 가능한 포트에서 실행됩니다. 즉, :2는 서버가 포트 5902(5900+2)에서 실행 중임을 의미합니다. 
명심해야 할 점은 VNC 서버와 작업할 때 :X는 5900+X를 참조하는 디스플레이 포트입니다
다음 단계를 계속하기 전에 -kill 옵션이 포함된 vncserver 명령과 서버 번호를 인수로 사용하여 VNC 인스턴스를 중지합니다. 이 예에서는 서버가 포트 5901(:1)에서 실행 중이므로 다음을 사용하여 중지합니다.
 
- 위의 vncserver -kill :1 명령에 대한 설명 
- [vnc 설치 참고] : (https://jjeongil.tistory.com/1332)
```


- [참고한 자료1. 설치 코드, 삭제] (https://jaynamm.tistory.com/entry/Install-Docker-Engine-on-CentOS7-centos7-%EB%8F%84%EC%BB%A4-%EC%84%A4%EC%B9%98)
- [참고한 자료2. 설치코드 구글링 참고자료] (https://1mini2.tistory.com/21)
- [docker 개념] (https://myjamong.tistory.com/297)
- [깃랩 초기 비번 생성 참고자료] : (https://oingdaddy.tistory.com/369)
