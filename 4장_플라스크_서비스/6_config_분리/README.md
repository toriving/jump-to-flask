# 6. config 분리

이번장에서는 개발환경과 운영환경에서 서로다른 config.py 파일을 읽어서 처리할 수 있는 방법에 대해서 알아보자.

### 환경파일 분리

먼저 config.py 파일을 다음과 같은 패키지 구조로 변경하자.

```
config/
  __init__.py
  default.py
  production.py
  development.py
```

#### config.py

그리고 원래 있던 `config.py` 파일은 더이상 필요하지 않으므로 삭제한다.

#### __init__.py

그리고 `__init__.py` 파일은 config 디렉터리가 패키지임을 알려주는 파일로 내용없이 파일만 생성하면 된다.

#### default.py

`myproject\config\default.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
import os

BASE_DIR = os.path.dirname(os.path.dirname(__file__))
# ---------------------------------------------------------------------------------------- #
```

BASE_DIR 은 기존 config.py 파일에 있던 환경변수로 프로젝트의 루트 디렉터리를 의미한다.

기존 config.py의 위치는 `C:\projects\myproject`이고 default.py의 위치는 `C:\projects\myproject\config`로 디렉터리가 1 Depth 더 늘어났기 때문에 `os.path.dirname` 을 한번 더 사용하여 위와같이 BASE_DIR을 설정해 주었다. 

`C:\projects\myproject\config\default.py`에서 총 2번의 `os.path.dirname`이 사용되었으므로 BASE_DIR은 결국 `C:\projects\myproject`가 될 것이다.

#### development.py

`myproject\config\development.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from config.default import *

SQLALCHEMY_DATABASE_URI = 'sqlite:///{}'.format(os.path.join(BASE_DIR, 'pybo.db'))
SQLALCHEMY_TRACK_MODIFICATIONS = False
SECRET_KEY = "dev"
# ---------------------------------------------------------------------------------------- #
```

개발환경에서 사용하는 환경변수들을 정의하였다. 

`from config.default import *` 는 `config/default.py` 파일에 정의되어 있는 모든 내용을 사용한다는 의미이다. 

즉, development.py에서 default.py의 BASE_DIR 환경변수 값을 그대로 사용할 수 있게 된다.

#### production.py

`myproject\config\production.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from config.default import *

SQLALCHEMY_DATABASE_URI = 'sqlite:///{}'.format(os.path.join(BASE_DIR, 'pybo.db'))
SQLALCHEMY_TRACK_MODIFICATIONS = False
SECRET_KEY =b'Z\xbd\xb1$\x8d\x94n9\x01\xa3\xc9B\x0f2\xdfw'
# ---------------------------------------------------------------------------------------- #
```

운영에서 사용할 데이터베이스는 아직 준비되지 않았으므로 일단 개발과 동일한 데이터베이스 설정을 유지하였다. 

그리고 운영환경에서 사용할 SECRET_KEY는 다음처럼 발급하여 저장하였다.

서버 콘솔창에서 다음과 같이 타이핑하자

`python -c "import os; print(os.urandom(16))"`

나온 것을 SECRET_KEY로 사용한다.

### 환경파일 적용

`pybo\__init__.py`

```python
...

def create_app():
    app = Flask(__name__)
    # ---------------------------------------- [edit] ---------------------------------------- #
    app.config.from_envvar('APP_CONFIG_FILE')
    # ---------------------------------------------------------------------------------------- #

    # ORM
    ...
```

※ 파일 상단의 `import config` 문장은 삭제하도록 하자.

`app.config.from_object(config)` 을 `app.config.from_envvar('APP_CONFIG_FILE'`) 로 변경해 주었다. 

`app.config.from_envvar('APP_CONFIG_FILE'`) 코드는 환경변수 `APP_CONFIG_FILE`에 정의된 파일을 환경파일로 사용하겠다는 의미이다.

#### 개발환경

따라서 개발환경에서는 다음과 같이 APP_CONFIG_FILE 환경변수를 설정하고 개발서버를 실행해야 한다.

`export APP_CONFIG_FILE=config\development.py`

헤주면 된다.

하지만 매번 하기에 귀찮으니 다른 방법을 사용하자

sh-script를 만들자

```shell script
export APP_CONFIG_FILE=config\development.py
...
flask run
```

대충 이런식..


#### 서버환경

서버환경도 개발환경에 적용했던것과 마찬가지 방법을 사용하면 된다.

### 서버 설정 자동화

서버도 마찬가지로 위와 같이 할 수 있다.

하지만 이렇게 하면 실행이 안되서 롤백했다.
