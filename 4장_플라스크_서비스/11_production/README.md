# 11. production

지금까지는 우리는 FLASK_ENV 환경변수를 "development"로 설정하고 플라스크 어플리케이션을 실행하였지만 운영서버에서는 FLASK_ENV 환경변수를 "production"으로 설정해야 한다.

왜냐하면 FLASK_ENV를 "development"로 설정하면 디버그 모드가 자동으로 "on"이 되어 오류의 내용이 화면에 상세하게 보여지기 때문이다.

플라스크 어플리케이션을 "development" 설정으로 운영한다면 오류 발생시 서버의 많은 정보가 노출될 것이다. 

이것은 어쩌면 서버 해킹등의 매우 나쁜 결과를 초래할 수도 있을 것이다.

따라서 운영환경에서는 반드시 FLASK_ENV 항목을 "production"으로 설정해야 한다.

다음과 같이 서버의 Gunicorn 환경설정 파일인 myproject.env 파일을 다음과 같이 수정하자.

`myproject.env`

```shell script
FLASK_APP=pybo
FLASK_ENV=production
APP_CONFIG_FILE=/home/ubuntu/projects/myproject/config/production.py
```

그리고 변경된 사항을 적용하기 위해 Gunicorn을 다음처럼 재 시작해 주도록 하자.

`sudo systemctl restart myproject.service`