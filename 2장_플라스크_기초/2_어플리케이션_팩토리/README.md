# 2. 어플리케이션 팩토리

플라스크 어플리케이션(app)은 다음처럼 Flask라는 클래스에 의해서 만들어진 인스턴스이다.

```python
app = Flask(__name__)
```

그리고 플라스크는 이 app을 사용하여 많은 설정을 함.

app을 전역변수로 사용할 경우 프로젝트 규모가 커질수록 순환참조 같은 문제가 발생할 소지가 많아짐.

이런 문제를 예방하기 위한 어플리케이션 팩토리를 사용하라고 플라스크는 권고함

### __init__.py
우선 pybo.py를 `pybo/__init__.py`로 변경하자.

그리고 플라스크를 로컬서버에서 실행해보자

```shell script
$ source source.me

$ flask run
 * Serving Flask app "pybo" (lazy loading)
 * Environment: development
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 195-632-818
```

FLASK_APP=pybo로 설정했었고 여기서 사용된 pybo는 모듈이므로 pybo.py 대신 pybo/__init__.py 파일이 대신하게 되어 정상 작동된다.

### 어플리케이션 팩토리

`__init__.py` 파일을 다음처럼 수정하자

```python
from flask import Flask

def create_app():
    app = Flask(__name__)

    @app.route('/')
    def hello_pybo():
        return 'Hello, Pybo!'

    return app 
```

create_app 이라는 함수를 추가하여 Flask 클래스의 인스턴스(app)를 생성하여 리턴하도록 변경함.

create_app 함수가 플라스크의 어플리케이션 팩토리이다.