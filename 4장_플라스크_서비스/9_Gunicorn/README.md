# 9. Gunicorn

이번장에서는 파이보 시스템이 사용할 WSGI 서버인 Gunicorn을 설치하고 사용해 보기로 하자.

※ Gunicorn은 구니콘이라고 읽는다.

WSGI 서버의 양대산맥으로 Gunicorn과 uwsgi가 있다. 

과거에는 "성능은 uwsgi가 앞서고 편의성은 Gunicorn이 좋다" 라는 의견들이 많았는데 요새는 Gunicorn의 성능이 매우 좋아졌기 때문에 Gunicorn을 사용하는 사람들이 점점 더 많아지고 있다.

### Gunicorn 설치

pip를 이용해서 설치할 수 있다.

`pip install gunicorn`

### Gunicorn 테스트

project 디렉터리에서 `gunicorn --bind 0:5000 "pybo:create_app()"` 을 해보자

접속도 잘 된다.

하지만 Gunicorn은 어플리케이션 서버로는 적당하지만 정적파일을 처리하기에는 효율적이지 않다.

파이보에는 bootstrap.min.css, bootstrap.min.js, style.css등 많은 정적파일이 있다.

정적파일을 효과적으로 처리하기 위해서는 웹서버가 필요하다.

웹 서버는 다음장에서 알아볼 것이다.

일단 Gunicorn이 정상동작 되는것을 확인했으므로 Ctrl-C 를 입력하여 Gunicorn서버를 중지하도록 하자.

### Gunicorn 소켓

WSGI 서버인 Gunicorn은 위에서 테스트한 것과 같이 포트를 이용하여 서버를 띄울수도 있지만 Unix계열 시스템에서는 포트로 서비스하는 방식 보다는 유닉스 소켓(Unix Socket)을 이용하는 방법을 더 선호한다.

이번에는 Gunicorn을 소켓으로 서비스하는 방법에 대해서 알아보자.

Gunicorn을 다음처럼 실행해 보자.

`gunicorn --bind unix:/tmp/myproject.sock "pybo:create_app()"`

※ 이렇게 소켓방식으로 Gunicorn 서버를 실행할 경우에는 단독으로 Gunicorn을 테스트할 수 없다. 

Nginx와 같은 웹서버에서 이 소켓파일로 WSGI 서버에 접속하도록 설정해야만 테스트가 가능하다.

### Gunicorn 서비스

이번에는 Gunicorn을 서비스로 등록해 보자. 

서비스로 등록하는 이유는 Gunicorn의 시작, 중지를 쉽게 하기 위함이고 또 AWS 서버가 재 시작될 때 자동으로 Gunicorn을 실행해 주기 위해서이다.

Gunicorn을 서비스로 등록하기 위해서는 환경변수 파일과 서비스 파일을 작성해야 한다.

#### 환경변수 파일

Gunicorn이 서비스로 실행될 경우 필요한 환경변수 값들을 읽을 수 있도록 Gunicorn이 사용하는 환경변수 파일을 생성해 주어야 한다.

다음의 환경변수 파일을 생성하도록 하자.

`myproject.env`

```
FLASK_APP=pybo
FLASK_ENV=development
APP_CONFIG_FILE=/home/ubuntu/projects/myproject/config/production.py
```

#### 서비스 파일

서비스 파일은 /etc/systemd/system/ 디렉터리에 myproject.service이라는 이름으로 생성한다.

※ 이 디렉터리는 시스템 디렉터리이기 때문 sudo vi myproject.service 처럼 관리자(sudo) 권한으로 파일을 생성해 주어야 한다. 

(만약 vi 대신에 nano 편집기를 사용한다면 sudo nano myproject.service 로 파일을 생성하자.)

`myproject.service`

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/projects/myproject
EnvironmentFile=/home/ubuntu/venvs/myproject.env
ExecStart=/home/ubuntu/venvs/myproject/bin/gunicorn \
        --workers 2 \
        --bind unix:/tmp/myproject.sock \
        "pybo:create_app()"
[Install]
WantedBy=multi-user.target
```

위 파일에서 사용된 EnvironmentFile 항목이 위에서 작성한 환경변수 파일을 불러오는 부분이다. 

--worker 2의 의미는 Gunicorn 프로세스 갯수를 2개로 사용하라는 의미이다.



#### 서비스 실행과 등록

파일이 생성되었으면 다음처럼 서비스를 실행해 보자. 서비스 실행은 역시 관리자(sudo)로 실행해야 한다.

`sudo systemctl start myproject.service`

서비스를 종료하기 위해서는 다음 명령을 수행한다.

`sudo systemctl stop myproject.service`

서비스를 재시작 하기 위해서는 다음 명령을 수행한다.

`sudo systemctl restart myproject.service`