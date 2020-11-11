# 2. 프로젝트의 구조

규모있는 프로젝트를 만들기 위해서는 프로젝트의 구조를 잘 만들어야 한다.

### 프로젝트의 구조
앞으로 만들어갈 파이보 프로젝트의 구조
```text
c:/projects/myproject
├── pybo/
│      ├─ __init__.py
│      ├─ models.py
│      ├─ forms.py
│      ├─ views/
│      │   └─ main_views.py
│      ├─ static/
│      │   └─ style.css
│      └─ templates/
│            └─ index.html
└── config.py
```

### pybo 패키지
pybo.py 앱은 pybo 패키지로 변경할 수 있다. pybo 디렉터리의 `__init__.py`가 pybo.py의 역할을 수행할 수 있다.
그리고 pybo 패키지 하위에는 다음과 같은 파일들이 필요하다.

#### Models.py
파이보 프로젝트는 ORM을 지원하는 파이썬 데이터베이스 툴킷인 SQLAchemy를 사용함.

SQLAchemy는 모델기반으로 데이터베이스를 처리하기 때문에 models.py가 필요.

#### forms.py
브라우저에서 서버로 전송된 폼을 처리하기 위해서 WTFForms라는 라이브러리 사용.

WTForms 역시 모델기반으로 폼을 처리하기 때문에 폼 클래스를 정의하는 forms.py가 필요.

#### views
pybo.py 파일에 등록되어 있던 hello_pybo와 같은 함수를 뷰(views) 디렉터리를 생성하여 그 하위에 기능별로 저장.

#### static
파이보 프로젝트의 스타일시트(`*.css`), 자바스크립트(`*.js`) 그리고 이미지파일(`*.jpg`, `*.png`) 등을 저장하는 디렉터리.

#### templates
템플릿 디렉터리는 파이보의 질문목록, 질문상세등의 HTML 파일을 저장하는 디렉터리

### Config.py
config.py 파일은 파이보 프로젝트의 환경변수등을 저장하는 파일

