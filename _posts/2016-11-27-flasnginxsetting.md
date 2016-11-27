---
layout: post
title: Flask + Nginx 설정
date: 2016-11-27 15:00
categories: python
---


Flask + Nginx 설치 방법
===

환경
---
데비안 계열 중 우분투 서버<br>
여기서 접근하는 모든 디렉토리는 권한이 있어야함.

설치
----
```
apt-get install nginx # Nginx
apt-get install uwsgi # uWSGI
apt-get install uwsgi-plugin-python # python과 uWSGI를 연결하는 플러그인
```

Nginx 세팅
---------
디폴트 파일로 설정하는 방법
`vi /etc/nginx/sites-available/default`

```
server {
       listen       80; # 연결할 포트
       server_name  0.0.0.0;
       location / { try_files $uri @app; }
       location @app {
           include uwsgi_params;
           uwsgi_pass unix:socket파일이 위치할 경로;
           # ex) /home/server/uwsgi.sock
       }
}
```

uWSGI 세팅
----------
`vi /etc/uwsgi/apps-available/uwsgi.ini`

```
[uwsgi]
chdir = 프로젝트 경로
uid = 실행할계정
gid = 실행할계정
chmod-socket = 666
socket = socket파일이 위치할 경로
module = 모듈이름(실행할 파일 이름)
callable = 연결될 Flask 모듈이름
virtualenv = python 가상 환경 경로(virtualenv)
```

참고로 프로젝트 경로는 root를 포함하지 않는 것이 좋다 (권한 문제 때문에)

허용하는 파일로 등록하기위해 apps-enabled 디렉토리에 link함
`ln -s /etc/uwsgi/apps-available/uwsgi.ini /etc/uwsgi/apps-enabled/`

실행 방법
--------
세팅 후 재시작을 한다.
```
sudo service nginx restart
sudo service uwsgi restart
```

설정한 uwsgi에 맞춰서 실행
`uwsgi /etc/uwsgi/apps-available/uwsgi.ini &`
