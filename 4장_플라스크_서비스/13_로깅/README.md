# 13. 로깅

이번장에서는 플라스크의 로깅(Logging)에 대해서 알아보자.

### 서버오류

서버환경에서 파이보를 운영하다보면 프로그램 오류 또는 예기치 못한 상황등의 오류가 발생할 수 있을 것이다. 

임시로 main_views.py의 index함수 첫번째 줄에 다음과 같이 강제로 오류가 발생하도록 코딩해 보자.

`pybo\views\main_views.py`

```python
...
@bp.route('/')
def index():
    # ---------------------------------------- [edit] ---------------------------------------- #
    3/0  # 강제로 오류발생
    # ---------------------------------------------------------------------------------------- #
    return redirect(url_for('question._list'))
```

ZeroDivisionError 오류 발생

운영서버에서는 500 에러 나와서 확인을 못한다.

이를 위해 로그 파일을 구성한다.

### 플라스크 로깅 설정

운영환경에서 로그를 파일로 저장하기 위해 다음처럼 production.py 파일을 수정하자.

`config\production.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from logging.config import dictConfig
# ---------------------------------------------------------------------------------------- #
...

# ---------------------------------------- [edit] ---------------------------------------- #
dictConfig({
    'version': 1,
    'formatters': {
        'default': {
            'format': '[%(asctime)s] %(levelname)s in %(module)s: %(message)s',
        }
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs/myproject.log'),
            'maxBytes': 1024 * 1024 * 5,  # 5 MB
            'backupCount': 5,
            'formatter': 'default',
        },
    },
    'root': {
        'level': 'INFO',
        'handlers': ['file']
    }
})
# ---------------------------------------------------------------------------------------- #
```

플라스크는 파이썬의 기본 logging 모듈을 사용하기 때문에 위처럼 파이썬 logging의 dictConfig를 사용하여 로그를 설정할 수 있다.

이 설정에 사용된 항목을 하나씩 살펴보기로 하자.

#### version

version은 고정값 "1"을 사용해야 한다. 만약 "1" 대신 다른 값을 입력하면 ValueError 가 발생한다. 

어찌보면 의미없는 설정으로 보이는데 logging모듈이 업그레이드 되어도 현재 설정을 보장하기 위한 안전장치로 보인다.

#### formatters

포맷터에는 로그를 출력할 형식을 정의한다. 

위에서는 "default"라는 포맷터를 등록하였다. 

default 포맷터에 사용된 항목은 다음과 같다.

- asctime - 현재시간
- levelname - 로그의 레벨 (debug, info, warning, error, critical)
- module - 로그를 호출한 모듈명
- message - 출력내용

#### handlers

핸들러(handlers)는 로그를 출력하는 방법을 정의한다. 

위에서는 "file"이라는 핸들러를 등록하였다. file 핸들러에 사용된 항목은 다음과 같다.

- level - 출력 로그레벨로 INFO를 사용
- class - 파일 핸들러로 RotatingFileHandler를 사용. RotatingFileHandler는 파일 사이즈가 설정한 사이즈보다 커지면 파일 뒤에 인덱스를 붙여서 백업한다. 이 핸들러의 장점은 로그가 무한히 증가되더라도 일정갯수의 파일로 Rolling되기 때문에 로그파일이 너무 커져서 디스크가 꽉 차게 되는 위험을 방지할 수 있다.
- filename - 로그파일명
- maxBytes - 로그파일 크기는 5MB로 설정
- backupCount - Rolling되는 파일의 갯수를 의미한다. 총 5개의 로그파일로 유지되도록 설정
- formatter - 포맷터는 default를 사용

#### root

모든 로거(Logger)의 상위가 되는 로거를 의미한다. 

로그레벨은 INFO 로그를 출력할 핸들러로는 file 핸들러를 추가해 주었다.

로그레벨은 다음과 같이 5개로 구성된다.

- DEBUG - 디버깅을 위한 목적으로 사용
- INFO - 일반적인 정보를 출력할때 사용
- WARNING - 마이너한 문제가 발생했을 때 사용
- ERROR: 오류등의 큰 문제가 발생했을 때 사용
- CRITICAL: 심각한 문제가 발생했을 때 사용
- 각 로그레벨은 다음과 같은 순서가 있다.

DEBUG < INFO < WARNING < ERROR < CRITICAL

만약 핸들러나 로거에서 로그레벨을 INFO로 설정하면 DEBUG로 출력한 로그는 출력되지 않고 INFO 이상으로 출력한 로그들만 출력된다. 

만약 ERROR로 설정했다면 ERROR 또는 CRITICAL로 출력한 로그들만 출력될 것이다.

이렇게 변경하고 git을 이용하여 production.py를 서버에 적용하자.

그리고 다음처럼 서버에 logs 디렉터리를 반드시 생성해야 한다.


### 로그확인

이제 다시 서버에서 파이보 메인페이지에 접속해 보자. 

main_vews.py의 index함수에서 3을 0으로 나누기 때문에 여전히 "Internal Server Error" 오류가 발생하게 될 것이다.

로그파일에 해당오류가 정상적으로 생성되었는지 다음처럼 확인해 보자.

오류의 내용이 정확히 출력되는 것을 확인할 수 있을 것이다.

### 파이보 로그

이번에는 오류등에 의해서 출력되는 로그가 아닌 프로그램에서 직접 로그를 출력할 수 있는 방법에 대해서 알아보자.

main_views.py의 index함수에 임시로 작성한 3/0 코드는 삭제하고 다음과 같이 수정해 보자.

`pybo\views\main_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, current_app
# ---------------------------------------------------------------------------------------- #
...

def index(request):
    # ---------------------------------------- [edit] ---------------------------------------- #
    current_app.logger.info("INFO 레벨로 출력")
    # ---------------------------------------------------------------------------------------- #
    ...
```

직접 로그를 출력하기 위해서는 위처럼 current_app.logger 를 사용해야 한다. 

current_app 은 create_app 함수에 의해서 생성된 플라스크 앱(app)을 의미하며 request와 마찬가지로 컨텍스트(Context) 구간 내에서 사용할 수 있도록 플라스크가 자동으로 생성해 주는 객체이다.

서버에 수정된 내용을 적용하고 파이보 메인페이지에 접속하면 다음처럼 myproject.log 파일에 로그가 출력되는 것을 확인할 수 있을 것이다.