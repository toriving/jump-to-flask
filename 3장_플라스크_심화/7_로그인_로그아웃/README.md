# 7. 로그인 로그아웃

로그인 로그라웃 기능을 추가하자

### 로그인

로그인을 구현해보자

#### 로그인 폼

로그인 폼을 작성하자

`myproject\pybo\forms.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
class UserLoginForm(FlaskForm):
    username = StringField('사용자이름', validators=[DataRequired(), Length(min=3, max=25)])
    password = PasswordField('비밀번호', validators=[DataRequired()])
# ---------------------------------------------------------------------------------------- #
```

#### 로그인 라우트 함수

로그인을 수행하는 라우트 함수를 작성하자

`myproject\pybo\views\auth_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, render_template, flash, request, session
from werkzeug.security import generate_password_hash, check_password_hash
# ---------------------------------------------------------------------------------------- #
from werkzeug.utils import redirect

from pybo import db
# ---------------------------------------- [edit] ---------------------------------------- #
from pybo.forms import UserCreateForm, UserLoginForm
# ---------------------------------------------------------------------------------------- #
...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/login/', methods=('GET', 'POST'))
def login():
    form = UserLoginForm()
    if request.method == 'POST' and form.validate_on_submit():
        error = None
        user = User.query.filter_by(username=form.username.data).first()
        if not user:
            error = "존재하지 않는 사용자입니다."
        elif not check_password_hash(user.password, form.password.data):
            error = "비밀번호가 올바르지 않습니다."
        if error is None:
            session.clear()
            session['user_id'] = user.id
            return redirect(url_for('main.index'))
        flash(error)
    return render_template('auth/login.html', form=form)
# ---------------------------------------------------------------------------------------- #
```

`/login/` 이라는 라우트 URL에 매핑되는 login 함수를 생성하였다.

login함수도 signup함수와 마찬가지로 POST 요청인 경우에는 로그인을 수행하고 GET 요청인 경우에는 로그인을 할 수 있는 템플릿을 렌더링하도록 하였다.
 
로그인 수행시 폼 입력으로 받은 username에 해당되는 사용자가 있는지를 조사하여 없을 경우 "존재하지 않는 사용자입니다." 라는 오류를 발생시킨다. 

그리고 check_password_hash 함수를 사용하여 폼 입력으로 받은 비밀번호와 데이터베이스에 저장된 비밀번호가 일치하는지를 조사한다. 

데이터베이스에 저장된 비밀번호는 암호화된 비밀번호이므로 비밀번호 검증시에는 반드시 check_password_hash 함수를 사용해야 한다.

폼 검증과 사용자 및 비밀번호 체크가 이상없다면 플라스크 세션(session)에 user_id라는 키에 조회된 사용자의 id값을 저장한다. 

세션은 request와 마찬가지로 플라스크가 자동으로 생성하여 제공하는 변수이다. 

단, request는 요청과 응답이라는 한 사이클에만 의미있는 값이라면 세션은 플라스크 메모리에 저장되기 때문에 플라스크 서버가 구동중인 동안에는 영구적으로 사용할 수 있는 값이다.


#### 로그인 템플릿

그리고 로그인을 수행할 템플릿을 다음과 같이 작성하자.

`myproject\pybo\templates\auth\login.html`

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
            <label for="password">비밀번호</label>
            <input type="password" class="form-control" name="password" id="password"
                   value="{{ form.password.data or '' }}">
        </div>
        <button type="submit" class="btn btn-primary">로그인</button>
    </form>
</div>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

### 네비게이션 바

로그인 URL을 변경하자

`\myproject\pybo\templates\navbar.html`

```html
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <...>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="{{ url_for('auth.signup') }}">계정생성</a>
            </li>
            <li class="nav-item ">
                <!-- ---------------------------------------- [edit] ---------------------------------------- -->
                <a class="nav-link" href="{{ url_for('auth.login') }}">로그인</a>
                <!-- ---------------------------------------------------------------------------------------- -->
            </li>
        </ul>
    </div>
</nav>
```

### 로그인 수행

정상적인 사용자이름과 비밀번호를 입력하면 로그인이 수행되고 메인화면으로 이동되는 것을 확인할 수 있다. 

하지만 로그인이 성공되었지만 네비게이션바에는 여전히 "로그인" 링크가 보일 것이다. 

로그인을 하고 난 후에는 "로그아웃" 링크가 보이고 로그아웃 상태에서는 "로그인" 링크가 보이도록 수정해야 할 것이다.

사용자가 로그인이 되었는지는 session의 값을 조사하면 알 수 있다. 

이렇게 단순히 session에 user_id라는 값이 존재하는지만 체크할수도 있지만 좀 더 일반적으로 사용할 수 있는 다음의 방법을 사용하기로 하자.

다음처럼 auth_views.py 파일에 load_logged_in_user 함수를 구현해 보자.

`myproject\pybo\views\auth_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, render_template, flash, request, session, g
# ---------------------------------------------------------------------------------------- #
from werkzeug.security import generate_password_hash, check_password_hash
from werkzeug.utils import redirect
...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.before_app_request
def load_logged_in_user():
    user_id = session.get('user_id')
    if user_id is None:
        g.user = None
    else:
        g.user = User.query.get(user_id)
# ---------------------------------------------------------------------------------------- #
```

`@bp.before_app_request` 어노테이션은 플라스크에서 제공하는 기능으로 이 어노테이션이 적용된 함수는 라우트 함수 실행전에 항상 먼저 실행된다. 

따라서 load_logged_in_user 함수는 모든 라우트 함수 실행전에 반드시 먼저 실행될 것이다.

load_logged_in_user함수에서 사용된 g변수는 플라스크가 제공하는 컨텍스트 변수로 request변수와 마찬가지로 요청과 응답이라는 한 사이클에서만 유효한 컨텍스트 변수이다. 

load_logged_in_user함수는 session 변수에 user_id 값이 있는 경우 User데이터를 조회하여 g.user 변수에 조회된 User객체를 저장한다.

이렇게 하면 이제 사용자가 로그인이 되었는지 체크하기 위해서 session을 찾을 필요없이 g.user에 값이 있는지만 조사하면 될 것이다. 

또한 g.user에는 User객체가 저장되어 있기 때문에 사용자이름이나 이메일등의 정보도 추가로 얻을수 있는 이점이 있다.

이제 네비게이션바를 다음과 같이 수정하자.

`myproject\pybo\templates\navbar.html`

```html
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{{ url_for('main.index') }}">Pybo</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {% if g.user %}
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">{{ g.user.username }} (로그아웃)</a>
            </li>
        </ul>
        {% else %}
        <!-- ---------------------------------------------------------------------------------------- -->
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="{{ url_for('auth.login') }}">로그인</a>
            </li>
            <li class="nav-item ">
                <a class="nav-link" href="{{ url_for('auth.signup') }}">계정생성</a>
            </li>
        </ul>
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {% endif %}
        <!-- ---------------------------------------------------------------------------------------- -->
    </div>
</nav>
```

`{% if g.user %}` 코드는 사용자의 로그인 유무를 판별한다. 

g.user는 앞서 구현한 load_logged_in_user 함수에 의해 값이 생성된다. 

로그인이 되었다면 g.user에 값이 존재하므로 사용자이름과 "로그아웃" 링크를 보여주고 로그인이 되어 있지 않다면 "로그인"과 "계정생성" 링크를 보여줄 것이다.


### 로그아웃

로그아웃은 간단하게 구현할 수 있다. 

다음처럼 로그아웃 함수를 auth_views.py 파일에 추가하자.

`myproject\pybo\views\auth_views.py`

```python
...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/logout/')
def logout():
    session.clear()
    return redirect(url_for('main.index'))
# ---------------------------------------------------------------------------------------- #
```

`/logout/` 라우트 URL에 매핑되는 logout함수를 생성하였다. 

`session.clear()` 코드는 세션의 모든 값을 삭제한다. 

따라서 session에 저장된 user_id도 삭제될 것이다. 

그러면 앞서 작성한 load_logged_in_user함수에서 session의 user_id 값을 읽을 수 없으므로 g.user도 None이라는 값이 설정될 것이다.

그리고 네비게이션바의 로그아웃 링크도 다음처럼 수정하자.

`myproject\pybo\templates\navbar.html`

```html
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <...>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        {% if g.user %}
        <ul class="navbar-nav">
            <li class="nav-item ">
                <!-- ---------------------------------------- [edit] ---------------------------------------- -->
                <a class="nav-link" href="{{ url_for('auth.logout') }}">{{ g.user.username }} (로그아웃)</a>
                <!-- ---------------------------------------------------------------------------------------- -->
            </li>
        </ul>
        {% else %}
        <...>
    </div>
</nav>
```