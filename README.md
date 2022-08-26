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
여기 이후 실패 .. 사용자 계정 및 루트 계정과 systemd관련 파일 active가 fail로 되는 문제 등.. 다양한 시행착오 발생 
중단 후 아래 방법으로 
```
```
이렇게 작성 수행중에 무언가가 제대로 수행이 되지 않아서 중단!
다른방법으로 수행하였습니다.
성공방법 -> [참고한 문서 ununtu18.04에서 vnc server 설치] : (https://z-wony.tistory.com/19)
Putty로 우분투 cli환경에 접속하여 root 계정으로 수행 -> 일반 사용자 계정으로는 systemd파일을 만들 수 없었다.
[성공수행 명령]
sudo apt-get install tigervnc-standalone-server tigervnc-xorg-extension  #사용한 TigerVNC서버 설치 
vncpasswd  #비밀번호 설정(꼼수2122!) 와 n 입력 무조건
sudo nano ~/.vnc/xstartup  #xstartup 작성

다음 내용으로 작성
#!/bin/sh
# Start Gnome 3 Desktop 
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
vncconfig -iconic &
dbus-launch --exit-with-session gnome-session &

저장 후 
vncserver -localhost no #서버실행
vncserver -list #명령으로 잘 수행되고 있는지 확인

UltraVNC Viewer를 내 PC윈도우에 설치 후 
위의 vncserver -list 로 확인한 명령의 DISPLAY 뒤의 숫자를 590X 의 X자리에 포트로 붙여서 접속
우분투 서버 ip인 172.28.4.244:5903 으로 접속 완료

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




- 우분투 OS위에 docker 설치 후 외부 인터넷망에서 접속할 수 있도록 jupyter notebook 서버 구축 테스트(8월 16일 수행)
```
[사용한 명령어]
- (pip install --upgrade pip)
- (pip install jupyter)
- service docker start #설치 완료한 docker 시작
- docker run -d -it --name remote_jupyter -p 8888:8888 --mount type=bind,source=/home/user/Documents,target=/root python:3.7.3 #컨테이너 생성
- docker start remote_jupyter  
- docker exec -it remote_jupyter /bin/bash # Container가 생성되면 바로 윗줄 명령과 이 명령으로 exec으로 접속해야 한다. attach로 접속하면 python 환경으로 들어가진다.(컨테이너 접속한 부분)
- pip install --upgrade pip
- pip install jupyter # 컨테이너 접속 후 우분투 설치
- jupyter notebook --generate-config -y #config 생성하는 부분 -> 서버를 띄우기 위해 인증정보를 생성해야 한다. config 파일은 생성 시 안내되는 폴더에 저장된다.(e.g. /root/.jupyter/jupyter_notebook_config.py) -> cd 명령으로 이 내용을 편집하기 위해서 이동하는 과정이 있다.
- ipython # 우선 ipython 실행 -> ipython으로 인증정보를 생성을 위해서
- # ipython 환경 실행해서 아래 절차 예시로 수행(생성되는 아래 토큰 부분은 따로 복사하여 저장해두어야함 -> 아래는 예시 토큰이다)
- In [1] :from notebook.auth import passwd
- In [2]: passwd()
- >>>> Enter password:
- >>>> Verify password:
- ### 이 부분 따로 복사해두어야 함
- Out[2]:  'argon2:$argon2id$v=19$m=10240,t=10,p=8$PLyCJPwzphBgN9jEthOcKw$HKjo9Clr7BCIKd7OhchspA' 
- In [3]: quit()
- # 과정 마무리 하고 나와서 config.py 파일을 편집한다(아래)
- apt-get update #업데이트 한번 수행해준다. 
- apt-get install nano #파일 편집을 위하여 다음 nano가 설치가 안되어있다면 설치진행
- nano /root/.jupyter/jupyter_notebook_config.py # 다음 경로에 있는 경로로 cd를 사용하여 이동 후 config 파일 열기
- # 아래 6줄을 상단에 내용을 입력해준다. 입력후 ctrl+S => y => ctrl+x로 나온다
- c=get_config()
- c.NotebookApp.ip='localhost'
- c.NotebookApp.open_browser=False
- c.NotebookApp.password='argon2:$argon2 ...... chspA'
- c.NotebookApp.password_required=True
- c.NotebookApp.port=8888
# 빠져나와서 
- jupyter notebook --ip=우분투ip(비밀)
- jupyter notebook --ip 0.0.0.0 --allow-root #다음 명령으로 띄어준다. 그러면 동작이 되는데 이렇게 한 후 나의 인터넷망에서 url검색하는 부분에 http://우분투ip:8888/tree 로 접속하면 우분투 환경에 있는 jupyter 서버에 접속할 수 있다
- 실행할 때마다 upyter notebook --ip 0.0.0.0 --allow-root 이를 해준 후 인터넷 망에서 위 url을 입력하여 접속한다. 
- 이렇게 하여서 Docker에 원격 주피터 서버 Container로 띄워보는 테스트를 마무리하였다.



- [설치 참고1] : (https://soundprovider.tistory.com/entry/DockerJupyter-%EC%9B%90%EA%B2%A9-%EC%A3%BC%ED%94%BC%ED%84%B0-%EC%84%9C%EB%B2%84-Container%EB%A1%9C-%EB%9D%84%EC%9A%B0%EA%B8%B0)
```

- 2022 8월 18일 기록
- 현재 맡아 하고 있는 업무부분을 기록합니다
```
STT 기술조사 및 RPA 활용방안에 대한 진행현황을 공유드립니다.
감사합니다.

담당 : 박선우 주임

1. STT 테스트	AICC pororo 구축
[7월말 시작 ~ 8월 12일]	

환경구성 중
 - 우분투 환경에서 텍스트 관련 테스트까지 완료상태
 - 음성 테스트를 위한 wav2letter 라이브러리 설치 시도 중
  (STT테스트는 미수행)

테스트 준비
 - STT 테스트 샘플 생성
(아나운서 대본예시를 보고 자리에서 전화기로 간단하게 녹음하여 wav파일을 제작)


2. 테스트	업무자동화 테스트 과제 선정
그룹 RPA 관련 접속 오류(시간 소요)
 - 학습 후 적용가능한 과제를 선정하려 했으나, 임시 보류

대체 > 파이썬을 활용한 업무자동화 준비 중
 - 일일점검 업무와 같은 단순 반복, 간단한 업무를 선정하여 자동화 및 시각화 케이스 취합
  → 다음 단계는 대기 중(docker 이미지 구축 테스트 진행 후)
  → 자동화 과제 프로그래밍 진행 예정
  ```
  
  - 도커 위에 구축한 pycaret용 쥬피터 노트북 한글 지원 
  ```
  - 그래프의 한글이 지원이 안되는 현상 해결 
  https://bagng.tistory.com/159
  
 - 우분투에서 작업을 진행하는 것이 아닌 우분투에서 도커 이미지서버로 접속했어야했다.
 
 
#Docker Container 만들기
#command line에서 다음과 같이 실행합니다.
docker run -it -p 8888:8888 --name "container name" -e LANG=ko_KR.UTF-8 pycaret/full
 
#Container를 만들었으면 한글 패키지를 설치합니다.
docker start "container name"
docker exec -it "container name" /bin/bash

[호스트 우분투가 아니라 위의 docekr exec -it --user root 컨테이너id /bin/bash
 위 명령으로 컨테이너로 접속하여 이곳에서 위의 사이트 참고하여 한글 지원 진행] 
apt-get update
apt-get install locales
locale-gen ko_KR.UTF-8
locale -a
 
#한글 폰트와 vi 입력기를 설치합니다.
apt-get install -y fonts-nanum fonts-nanum*
apt-get install vim

#서울 Timezone도 설치합니다.
apt install tzdata
tzselect

#4: Asia
#23: Korea (South)
#Asia/Seoul

ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

그래프 한글 문제 해결 완료
```
  
  
  
 
- 파이썬 업무 자동화 
```
- 쥬피터 노트북에서 수행
- 팀장님 일일 점검 파이썬 업무 자동화 테스트 진행하였습니다.

!python -m pip install paramiko  # ssh접속을 위한 필요 라이브러리 paramiko 설치 (쥬피터 노트북에서 수행)


import paramiko
ssh=paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('10.115.226.50', port='22', username='otadmin', password='admin00!!')  # 일일 작업 서버 정보 받아서 입력 
stdin, stdout, stderr = ssh.exec_command('df -h')  # 이 서버에서 일일 작업하는 명령어인 df -h 수행
print(''.join(stdout.readlines()))
ssh.close()

# 위 명령 수행을 위해 방화벽 및 문서중앙화를 열어주셨습니다 (정책으로 인해)
# 위 명령 결과 
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 7.8G     0  7.8G   0% /dev
tmpfs                    7.8G     0  7.8G   0% /dev/shm
tmpfs                    7.8G  201M  7.6G   3% /run
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root   47G   12G   36G  26% /
/dev/sda1               1014M  151M  864M  15% /boot
tmpfs                    1.6G     0  1.6G   0% /run/user/1005
tmpfs                    1.6G     0  1.6G   0% /run/user/1000
tmpfs                    1.6G     0  1.6G   0% /run/user/1004




import paramiko
ssh=paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('172.28.220.38', port='22', username='otadmin', password='admin00!!')
stdin, stdout, stderr = ssh.exec_command('bdf')
print(''.join(stdout.readlines()))
ssh.close()    # 다음 명령도 수행

# 위 명령 결과 
Filesystem          kbytes    used   avail %used Mounted on
/dev/vg00/lvol4    31457280 15132160 16197584   48% /
/dev/vg00/lvol1    2097152  156240 1925848    8% /stand
/dev/vg00/lvol6    8388608 5810442 2417323   71% /var
/dev/vg00/lvol7    16777216   17128 16629160    0% /var/adm/crash
/dev/vg10/lvol3    30736384 11827966 17728309   40% /userLog
/dev/vg10/lvol2    30736384 7665811 21629111   26% /oracle
/dev/vg00/lvol5    10485760 7934688 2531152   76% /opt
/dev/vg10/lvol4    30605312 21502387 8560465   72% /nsr
/dev/vg10/lvol1    51216384 48204973 2842619   94% /ics
 

# 다음과 같이 파이썬 일일점검 업무 자동화를 위한 테스트를 진행하였습니다.
# 사내 정책으로 인해 위와 같이 코딩하였을 때 [WinError 10060] 연결된 구성원으로부터 응답이 없어 연결하지 못했거나, 호스트로부터 응답이 없어 연결이 끊어졌습니다 
# 위 같은 오류가 나타났습니다. 팀장님께 말씀드려 문서중앙화, 방화벽 정책을 테스트를 위해 열어주셔서 테스트 결과가 잘 출력되었습니다.

- [참고한 자료2. paramiko를 이용한 ssh접속 후 명령어 실행] (https://zero-gravity.tistory.com/324)


``` 
  
  
- [참고한 자료1. 설치 코드, 삭제] (https://jaynamm.tistory.com/entry/Install-Docker-Engine-on-CentOS7-centos7-%EB%8F%84%EC%BB%A4-%EC%84%A4%EC%B9%98)
- [참고한 자료2. 설치코드 구글링 참고자료] (https://1mini2.tistory.com/21)
- [docker 개념] (https://myjamong.tistory.com/297)
- [깃랩 초기 비번 생성 참고자료] : (https://oingdaddy.tistory.com/369)
- [docker study] : (https://unpasoadelante.tistory.com/193?category=901931)
