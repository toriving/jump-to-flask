# 6. 계정생성

이번장에서는 사용자를 추가 할 수 있는 계정생성 (회원가입) 기능을 추가한다.

### 계정모델

계정모델에는 최소한 다음과 같은 속성이 필요하다

|속성|설명|
|---|---|
|username|사용자명(ID)|
|password|비밀번호|

여기에 추가로 이메일 속성을 추가하여 다음과 같은 모델을 작성하자

`pybo/models.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(150), unique=True, nullable=False)
    password = db.Column(db.String(200), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
# ---------------------------------------------------------------------------------------- #    
```

id 속성은 자동증가되는 User 모델의 프라이머리키이다. 

username, password, email 컬럼에는 모두 null 값을 허용하지 않는 nullable=False 속성을 지정하였다.

username과 email 컬럼에는 unique=True 속성을 지정했는데 이것은 동일한 값을 저장할 수 없다는 의미이다.
 
즉, 사용자ID와 이메일주소는 중복된 값을 저장할 수 없다는 말이다.

모델을 신규로 생성했으니 flask db migrate 명령을 다음과 같이 수행하자.

```shell script
$ flask db migrate
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
  Generating /mnt/c/workspace/jump-to-flask/project/migrations/versions/7cc4beb579d7_.py ...  done
```

```shell script
$ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade 6187f85e2ccb -> 7cc4beb579d7, empty message
```

### 계정 폼

이번에는 계정생성을 위한 폼을 다음과 같이 작성해 보자.

`myproject\pybo\forms.py`

```python
from flask_wtf import FlaskForm
# ---------------------------------------- [edit] ---------------------------------------- #
from wtforms import StringField, TextAreaField, PasswordField
from wtforms.fields.html5 import EmailField
from wtforms.validators import DataRequired, Length, EqualTo, Email
# ---------------------------------------------------------------------------------------- #    

...

# ---------------------------------------- [edit] ---------------------------------------- #
class UserCreateForm(FlaskForm):
    username = StringField('사용자이름', validators=[DataRequired(), Length(min=3, max=25)])
    password1 = PasswordField('비밀번호', validators=[
        DataRequired(), EqualTo('password2', '비밀번호가 일치하지 않습니다')])
    password2 = PasswordField('비밀번호확인', validators=[DataRequired()])
    email = EmailField('이메일', [DataRequired(), Email()])
# ---------------------------------------------------------------------------------------- #  

```

계정생성을 위한 폼의 클래스명은 UserCreateForm이다. 

username은 필수항목이고 길이가 3-25 사이여야 한다는 검증조건이 설정되었다.

Length는 폼 유효성 검증시 문자열의 길이가 최소길이(min)와 최대길이(max) 사이에 해당하는지를 검증한다.

password1과 password2는 "비밀번호"와 "비밀번호확인"에 대한 필드이다. 

로그인 할때는 비밀번호가 한번만 필요하지만 계정생성시에는 입력한 비밀번호가 정확한지 확인하기 위해 2개의 필드가 필요하다. 

password1과 password2 속성은 PasswordField로 생성되었다. 

StringField와 비슷하지만 템플릿에서 자동변환으로 사용시 <input type="password"> 태그로 변환되는 차이점이 있다. 

password1 속성과 password2 속성은 모두 필수값이어야 하고 두개의 값이 일치해야 하는 EqualTo 검증이 추가되었다. 

password1 속성에 지정된 EqualTo('password2') 는 password1과 password2의 값이 일치해야 함을 의미한다.

그리고 email 속성은 EmailField로 생성되었다. 

EmailField 역시 StringField와 동일하지만 템플릿 자동변환으로 사용시 <input type="email"> 태그로 변환된다. 

email 속성에는 필수값 검증조건에 더하여 Email() 검증조건이 추가되었다. Email() 검증조건은 해당 속성의 값이 이메일형식과 일치하는지를 검증한다.

그리고 Email() 검증을 사용하기 위해서는 다음처럼 email-validator를 설치해 주어야 한다.

```shell script
$ pip install email_validator
```

### 계정생성 뷰

이제 계정모델과 계정폼이 준비되었으니 계정생성을 위한 뷰를 만들어보자. 

계정생성은 메인(main_views.py), 질문(question_views.py), 답변(answer_views.py) 어디에도 해당되지 않으므로 계정생성과 로그인/로그아웃을 담당할 auth_views.py 파일을 신규로 생성하는 것이 좋을 것이다.

다음처럼 auth_views.py 파일을 작성하자.

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, render_template, flash, request
from werkzeug.security import generate_password_hash
from werkzeug.utils import redirect

from pybo import db
from pybo.forms import UserCreateForm
from pybo.models import User

bp = Blueprint('auth', __name__, url_prefix='/auth')


@bp.route('/signup/', methods=('GET', 'POST'))
def signup():
    form = UserCreateForm()
    if request.method == 'POST' and form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if not user:
            user = User(username=form.username.data,
                        password=generate_password_hash(form.password1.data),
                        email=form.email.data)
            db.session.add(user)
            db.session.commit()
            return redirect(url_for('main.index'))
        else:
            flash('이미 존재하는 사용자입니다.')
    return render_template('auth/signup.html', form=form)
# ---------------------------------------------------------------------------------------- #
```

auth 라는 블루프린트를 추가해 주었다. 

이제 /auth/ 라는 URL 프리픽스로 시작하는 URL이 호출되면 auth_views.py의 함수들이 호출될 것이다.

계정생성을 위한 /signup 라우트를 추가하고 signup 함수를 생성하였다. 

signup 함수는 질문을 등록하는 것과 동일한 방법이 사용되었다. 

POST 요청이 발생할 경우에는 계정을 등록하고 GET요청이 발생할 경우에는 계정을 등록할수 있는 템플릿을 렌더링하도록 구현되었다.

계정을 등록할때에는 username으로 조회하여 이미 존재하는 사용자인지를 확인한다. 

이미 존재하는 사용자인 경우에는 flash('이미 존재하는 사용자입니다.') 코드로 오류를 발생시킨다. 

flash는 필드자체에 대한 오류가 아닌 로직에 의한 오류를 발생시킬 경우에 사용된다. 

flash로 발생시킨 오류를 템플릿에 표시하는 방법에 대해서는 잠시후에 얘기하도록 하자.

그리고 이미 존재하는 사용자가 아닌 경우에는 사용자를 신규 등록한다. 

이때 비밀번호는 입력받은 값을 그대로 저장하지 않고 generate_password_hash 함수를 사용하여 복호화가 불가능한 형태로 암호화하여 저장한다.

※ 복호화가 불가능하기 때문에 로그인시에는 입력받은 비밀번호를 암호화하여 저장된 비밀번호 데이터와 비교해야 한다.

### 블루프린트 등록

그리고 auth 블루프린트를 사용하기 위해 다음처럼 파이보 앱에 auth 블루프린트를 등록하자.

`myproject\pybo\__init__.py`

```python
...

def create_app():
    ...

    # 블루프린트
    # ---------------------------------------- [edit] ---------------------------------------- #
    from .views import main_views, question_views, answer_views, auth_views
    # ---------------------------------------------------------------------------------------- #
    app.register_blueprint(main_views.bp)
    app.register_blueprint(question_views.bp)
    app.register_blueprint(answer_views.bp)
    # ---------------------------------------- [edit] ---------------------------------------- #
    app.register_blueprint(auth_views.bp)
    # ---------------------------------------------------------------------------------------- #

    ...

    return app
```


### 계정생성 템플릿

이제 계정생성 템플릿 파일을 다음과 같이 작성하자. 

(계정생성 템플릿 파일은 템플릿 디렉터리 하위에 auth라는 디렉터리를 추가한 후 그곳에 저장하도록 하자.)

`pybo\templates\auth\signup.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends "base.html" %}
{% block content %}
<div class="container my-3">
    <form method="post" class="post-form">
        {{ form.csrf_token }}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="username">사용자 이름</label>
            <input type="text" class="form-control" name="username" id="username"
                   value="{{ form.username.data or '' }}">
        </div>
        <div class="form-group">
            <label for="password1">비밀번호</label>
            <input type="password" class="form-control" name="password1" id="password1"
                   value="{{ form.password1.data or '' }}">
        </div>
        <div class="form-group">
            <label for="password2">비밀번호 확인</label>
            <input type="password" class="form-control" name="password2" id="password2"
                   value="{{ form.password2.data or '' }}">
        </div>
        <div class="form-group">
            <label for="email">이메일</label>
            <input type="text" class="form-control" name="email" id="email"
                   value="{{ form.email.data or '' }}">
        </div>
        <button type="submit" class="btn btn-primary">생성하기</button>
    </form>
</div>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

계정생성을 위해 필요한 "사용자 이름", "비밀번호", "비밀번호 확인", "이메일" 총 4개의 필드가 추가되었다. 

"생성하기" 버튼을 클릭하면 POST 방식으로 폼 데이터가 /auth/signup/ URL로 전송될 것이다.

그리고 계정생성시 발생하는 오류를 표시하기 위해 {% include "form_errors.html" %}를 사용하였다. 

필드오류와 로직오류에 해당하는 flash 오류를 처리하기 위한 form_errors.html 템플릿파일을 포함하였다.

form_errors.html 템플릿 파일은 다음과 같다.

`pybo\templates\form_errors.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<!-- 필드오류 -->
{% for field, errors in form.errors.items() %}
<div class="alert alert-danger" role="alert">
    <strong>{{ form[field].label }}</strong>: {{ ', '.join(errors) }}
</div>
{% endfor %}
<!-- flash 오류 -->
{% for message in get_flashed_messages() %}
<div class="alert alert-danger" role="alert">
    {{ message }}
</div>
{% endfor %}
<!-- ---------------------------------------------------------------------------------------- -->
```

필드에서 발생한 오류를 표시하는 부분과 flash에 의해 발생한 오류를 표시하는 두 부분이 존재한다. 

필드오류는 폼 validators의 검증에 실패할 경우에 표시되고 flash 오류는 flash('이미 존재하는 사용자입니다.') 와 같은 로직에 의해 표시된다.

### 계정생성 링크추가

이제 계정생성 화면으로 진입할수 있는 링크를 네비게이션바에 다음과 같이 추가하자.

`pybo\templates\navbar.html`

```html
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <...>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <!-- ---------------------------------------- [edit] ---------------------------------------- -->
                <a class="nav-link" href="{{ url_for('auth.signup') }}">계정생성</a>
                <!-- ---------------------------------------------------------------------------------------- -->
            </li>
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
```

### 테스트
 
이제 계정생성을 테스트해 보자.

네비게이션바의 "계정생성" 링크를 클릭하자. 그러면 다음과 같은 계정생성을 할 수 있는 화면이 나타난다.

입력값 4개를 모두 채워 계정생성을 해 보자. 만약 비밀번호1과 비밀번호2를 다르게 입력하면 다음과 같은 오류가 발생할 것이다.

필수값 검증과 이메일규칙 검증등이 모두 적용되는 것을 확인할 수 있을 것이다.

입력값을 올바르게 하여 계정을 생성하였다면 메인페이지로 리다이렉트될 것이다.

다음과 같이 플라스크 쉘을 사용하여 생성된 계정을 확인해 보자.

```shell script
$ flask shell

Python 3.8.2 (default, Apr 27 2020, 15:53:34)
[GCC 9.3.0] on linux
App: pybo [development]
Instance: /mnt/c/workspace/jump-to-flask/project/instance
>>> from pybo.models import User
>>> User.query.all()
[<User 1>]
>>> User.query.first().username
'박동주'
>>>
```
