# dockerinstall
## CentOS7 환경에서 관리자 권한(root)으로 docker를 설치해보았다
## 성공적..
## docker 라는 것에 대해 알게 되었다 
- docker 란 ? Docker란 Go언어로 작성된 리눅스 컨테이너 기반으로하는 오픈소스 가상화 플랫폼이다.


- 사내 사용자 매뉴얼 사이트 제작 예정
- 전산과 개발로 나누어 제작 기획 중
- 상무님께서 docs.niceamc.co.kr 도메인으로 docker에 gitlab 구축 임무 주심 
- 대리님과 함께 수행
- docker가 설치되어있는 centos7환경에 원격으로 붙어서 진행하였음
- gitlab에서 사용하는 포트는 3가지로, 443-https , 80:http, 22:ssh가 있다고함 
- 여기에 맞춰서 사내 정책으로 대리님께서 열어주신 포트인 8000번에서 10000번 사이 포트와 80번에서 90사이 포트 중 사용할 수 있는 포트번호들을 각각 지정해준 부분이다
- 10.115.224.210 서버에서 진행하였다
- 이름은 gitlab_docs로, 도메인은 docs.niceamc.co.kr로 설정해준 부분입니다
- (아래 volume부분은 생략이 가능할 수도 있을 것 같다고 테스트 추후에 진행해보기로 하였습니다)
- 크게는 82번 포트를 사용
- 만약 다른 사이트를 하나더 구축하려면 이름과 포트번호를 사용할 수 있는 것으로 아래 코드 부분을 변경하여 진행하면 될 것 같습니다

``` 
docker run -e VIRTUAL_HOST=docs.niceamc.co.kr --expose 82 --detach --hostname 10.115.224.210 --publish 8442:443 --publish 82:80 --publish 822:22 --name gitlab_docs --restart always --volume /srv/gitlab/config:/etc/gitlab_docs --volume /srv/gitlab/logs:/var/log/gitlab_docs --volume /srv/gitlab/data:/var/opt/gitlab_docs gitlab/gitlab-ce:latest
```

- docker 진행 시 자주 사용하는 명령
```
docker ps -a  -> docker 현 상태 (올라와있는지 등등) 확인 할 수 있는 명령
docker service start  -> docker 시작
docker service restart -> 재시작
```



- [참고한 자료1. 설치 코드, 삭제] (https://jaynamm.tistory.com/entry/Install-Docker-Engine-on-CentOS7-centos7-%EB%8F%84%EC%BB%A4-%EC%84%A4%EC%B9%98)
- [참고한 자료2. 설치코드 구글링 참고자료] (https://1mini2.tistory.com/21)
- [docker 개념] (https://myjamong.tistory.com/297)
