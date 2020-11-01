# 3. 블루프린트

hello_pybo 함수는 URL 주소 /에 매핑되는 함수로 @app.route('/') 라는 어노테이션이 그 블루프린트를 수행한다.

하지만 이러한 라우트함수(`@app.route`로 매핑되는 함수)들은 기능이 필요할때마다 계쏙 추가되어야 하므로 create_app 함수내에 이러한 함수들을 계속 추가하는 것은 불편함

따라서 플라스크의 블루프린트(Blueprint)를 사용하면 더 관리하기 쉬움

### 블루프린트 생성

먼저 hello_pybo 함수에 블루프린트를 적용하자

views라는 디렉터리를 pybo 디렉터리 하위에 생성하자.

그리고 main_views.py 파일을 다음과 같이 작성하자.

```python
from flask import Blueprint

bp = Blueprint('main', __name__, url_prefix='/')


@bp.route('/')
def hello_pybo():
    return 'Hello, Pybo!'
```

`pybo/__init__.py`에서 `@app.route`가 아닌 `@bp.route`로 변경되었다.

`@bp.route`의 bpㅔ는 다음과 같은 Blueprint 클래스의 객체이다.

```python
bp = Blueprint('main', __name__, url_prefix='/')
```

Blueprint는 이름, 모듈명, URL 프리픽스 값을 입력으로 객체를 생성한다. 여기서 사용된 'main' 이라는 이름은 나중에 함수명으로 URL을 찾아내는 url_for 함수에서 사용된다.

URL 프리픽스(url_prefix)는 main_views.py 파일에 있는 함수들의 URL 앞에 항상 붙게 되는 프리픽스 URL을 의미한다. 만약 위에서 url_prefix='/' 대신 url_prefix='/main' 이라고 선언했다면 hello_pybo 함수를 호출하기 위해서는 http://localhost:5000/ 대신 http://localhost:5000/main/ 이라고 호출해야 한다.

### 블루프린트 적용

`pybo/__init__.py`를 다음과 같이 수정하자

```python
from flask import Flask


def create_app():
    app = Flask(__name__)

    from .views import main_views
    app.register_blueprint(main_views.bp)

    return app
```

블루프린트를 사용하기 위해 `app.register_blueprint(main_views.bp)`와 같이 생성한 블루프린트 객체 bp를 등록해주면 된다.

`main_views.py`를 다음과 같이 수정하자

```python
from flask import Blueprint

bp = Blueprint('main', __name__, url_prefix='/')

@bp.route('/hello')
def hello_pybo():
    return 'Hello, Pybo!'


@bp.route('/')
def index():
    return 'Pybo index'
```

기존 hello_pybo 함수의 라우트를 `/`에서 `/hello`로 바꾸어 주고 index라는 함수를 추가하여 `/` URL에 매핑해 줌
이렇게 수정하고 실행하여 브라우저에서 `http://localhost:5000`와 `http://localhost:5000/hello`에 접근해보자
