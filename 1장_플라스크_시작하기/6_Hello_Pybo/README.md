# 6. Hello Pybo

"Hello Pybo!"라는 문장을 브라우저에 출력해주는 플라스크 프로그램 작성

1. 어플리케이션(Application)
2. FLASK_APP
3. FLASK_ENV
4. 환경변수 추가

### 어플리케이션(Application)

pybo.py 파일 생성
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_pybo():
    return 'Hello, Pybo!'
```

`app = Flask(__name__)`은 플라스크 어플리케이션을 생성하는 문장

__name__은 현재 모듈명을 의미하므로 pybo라는 문자열이 대입됨

@app.route는 특정 URL로 접속하면 해당함수를 호출하는 플라스크의 데코레이터이다

- 참고 : [클로저와 데코레이터](https://wikidocs.net/83687)

현재 상태에서 아래와 같이 로컬 서버를 실행하면 오류가 발생한다

```shell script
$ flask run

 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
Usage: flask run [OPTIONS]

Error: Could not locate a Flask application. You did not provide the "FLASK_APP" environment variable, and a "wsgi.py" or "app.py" module was not found in the current directory.
```

FLASK_APP 환경을 설정을 하지 않아서 발생

### FLASK_APP

플라스크 서버를 실행할때는 반드시 `FLASK_APP` 이라는 환경변수를 통해 플라스크 앱을 지정해야 함

하지만 플라스크 앱의 모듈명이 app.py 라면 지정하지 않아도 디폴트로 인식이 됨

`FLASK_APP` 지정은 다음과 같음

```shell script
$ export FLASK_APP=pybo
```

Windows 에서는 export 대신 set을 사용하자

그리고 다시 실행

```shell script
$ flask run

 * Serving Flask app "pybo"
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

잘 작동하지만 개발을 위해 사용되는 서버인데 운영환경(production)으로 실행되었다는 경고문을 볼 수 있음

디폴트로 운영환경으로 되어 있기 때문에 경고 발생


### FLASK_ENV

개발환경으로 바꾸는 방법은 다음과 같음

```shell script
$ export FLASK_ENV=development
```

Windows는 마찬가지로 set을 이용

그리고 다시 실행

```shell script
$ flask run

 * Serving Flask app "pybo" (lazy loading)
 * Environment: development
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 319-981-972
```

정상 작동 된다.

브라우저에서 `http://127.0.0.1:5000/` 로 접속해보자.

Good

### 환경변수 추가

`pybo/source.me`를 만들어 사용
