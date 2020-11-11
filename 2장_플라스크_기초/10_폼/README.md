# 10. 폼

플라스크의 폼 모듈을 사용하는 방법에 대해서 알아보자

### Flask-WTF 설치

플라스크에서 폼(Form) 을 사용하기 위해 Flask-WTF 를 먼ㅁ저 설치하자

```
$ pip install Flask-WTF
```

### SECRET_KEY

Flask-WTF를 사용하기 위해서는 플라스크 환경변수 SECRET_KEY가 필요

SECRET_KEY는 CSRF(cross-site request forgery)에서 사용

CSRF는 보안에 관련된 항목으로 form으로 전송된 데이터가 실제 웹페이지에서 작성된 데이터인지를 판단해 주는 가늠자 역할을 함

config.py 파일에 SECRET_KEY 환경변수를 추가

`project\config.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
SECRET_KEY = "dev"
# ---------------------------------------------------------------------------------------- #
```

개발환경이므로 "dev" 로 설정, 일반적으로 어려운 키를 사용해야함

### 질문등록

질문등록을 할 수 있는 기능 추가

템플릿 하단에 버튼을 생성하자

`templates\question\question_list.html`


```html
    <...>
    </table>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <a href="{{ url_for('question.create') }}" class="btn btn-primary">질문 등록하기</a>
    <!-- ---------------------------------------------------------------------------------------- -->
</div>
{% endblock %}
```

"질문 등록하기" 링크 `{{ url_for('question.create') }}` 를 추가함

#### 뷰 함수 추가

`{{ url_for('question.create') }}` 를 추가하였으므로 `question_views.py` 파일에 라우트 함수를 추가해야함

`pybo\views\question_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from ..forms import QuestionForm
# ---------------------------------------------------------------------------------------- #
...
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/create/')
def create():
    form = QuestionForm()
    return render_template('question/question_form.html', form=form)
# ---------------------------------------------------------------------------------------- #
```

create함수는 QuestionForm을 사용한다. 

QuestionForm은 질문을 등록하기 위해 사용할 플라스크의 폼(Form)이다. 

render_template 함수는 템플릿 렌더시 QuestionForm의 객체 form을 전달하였다. 

form 객체는 템플릿에서 라벨이나 입력폼등을 만들때 필요하다.

#### 폼 작성

QuestionForm 작성

`forms.py` 를 만들자

`myproject\pybo\forms.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField
from wtforms.validators import DataRequired


class QuestionForm(FlaskForm):
    subject = StringField('제목', validators=[DataRequired()])
    content = TextAreaField('내용', validators=[DataRequired()])
# ---------------------------------------------------------------------------------------- #
```

질문을 등록할 때 사용할 QuestionForm을 작성함 

QuestionForm은 Flask-WTF의 FlaskForm 클래스를 상속받아 작성해야 함

QuestionForm의 속성은 "제목"과 "내용"임

글자수의 제한이 있는 "제목"의 경우 StringField를 사용하고 글자수의 제한이 없는 "내용"은 TextAreaField를 사용한다.

필드에 대한 자세한 내용은 다음을 참조하자

https://wtforms.readthedocs.io/en/2.3.x/fields/#basic-fields

`StringField('제목', validators=[DataRequired()])` 에서 첫번째 입력인수인 "제목"은 폼 라벨(Label). 

템플릿에서 이 라벨을 이용하여 "제목"이라는 라벨을 출력할 수 있음.

두번째 입력인수인 `validators는` 검증을 위해 사용되는 도구로 필수 항목인지를 체크하는 `DataRequired`, 이메일인지를 체크하는 Email, 길이를 체크하는 `Length`등이 있음

예를들어 필수값이면서 이메일이어야 하면 `validators=[DataRequired(), Email()]` 과 같이 사용할 수 있음

validators 에 대한 자세한 내용은 다음을 참조하자

https://wtforms.readthedocs.io/en/2.3.x/validators/#built-in-validators

#### 템플릿 작성

질문을 등록하는 템플릿을 다음과 같이 작성하자

`question_form.html` 을 생성하도록 하자

`pybo\templates\question\question_form.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">

        {{ form.subject.label }}
        {{ form.subject() }}

        {{ form.content.label }}
        {{ form.content() }}

        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

QuestionForm에서 작성한 두개의 필드 subject와 content가 템플릿에서 사용되었음 

`{{ form.subject.label }}` 는 라벨을 표시하고 `{{ form.subject() }`}는 입력폼을 표시한다.

#### GET과 POST

브라우저에서 확인하면 "질문 등록하기" 버튼이 추가되었음

하지만 등록을 하면 `Method Not Allowed` 에러가 발생함

오류가 발생한 이유는 폼이 POST방식으로 전송되었기 때문.

`<form method="post" class="post-form my-3">`

폼 태그의 method 속성이 "post" 이므로 POST로 전송됨

라우트 함수인 create 함수는 디폴처 설정인 GET만 처리할 수 있음

POST를 사용하려면 create 함수의 라우트 속성을 다음과 같이 변경해야함

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/create/', methods=('GET', 'POST'))
# ---------------------------------------------------------------------------------------- #
def create():
    form = QuestionForm()
    return render_template('question/question_form.html', form=form)
```

하지만 다시한번 시도해보면 오류는 발생하지않지만 아무변화가 없음

create 함수에는 데이터를 저장하는 코드가 존재하지 않기 때문

전송된 폼을 저장하는 방법에 대해 알아보며 create 함수를 수정하자

`pybo\views\question_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from datetime import datetime

from flask import Blueprint, render_template, request, url_for
from werkzeug.utils import redirect

from .. import db
from ..models import Question
from ..forms import QuestionForm
# ---------------------------------------------------------------------------------------- #
...

@bp.route('/create/', methods=('GET', 'POST'))
def create():
    form = QuestionForm()
    # ---------------------------------------- [edit] ---------------------------------------- #
    if request.method == 'POST' and form.validate_on_submit():
        question = Question(subject=form.subject.data, content=form.content.data, create_date=datetime.now())
        db.session.add(question)
        db.session.commit()
        return redirect(url_for('main.index'))
    # ---------------------------------------------------------------------------------------- #        
    return render_template('question/question_form.html', form=form)
```

폼 전송이 POST로 요청된 경우 질문데이터를 한건 등록하도록 수정하였음

request.method는 현재 요청된 전송방식을 의미함

form.validate_on_submit() 은 POST로 전송된 폼 데이터의 정합성을 체크함

폼 데이터의 정합성은 폼 작성시 생성했던 DataRequired() 같은 체크항목을 말함

폼으로 전송된 "제목"은 form.subject.data 처럼 data속성을 사용하여 얻을 수 있음

그리고 POST로 전송된 데이터의 저장이 완료되면 메인화면(main.index)으로 리다이렉트하도록 하였음

목록조회 화면에서 "질문 등록하기" 버튼을 클릭한 경우에는 http://localhost:5000/question/create/ 페이지가 GET 방식으로 요청(request.method == 'GET')되어 질문등록 화면이 호출되고 질문등록 화면에서 "저장하기" 버튼을 클릭하면 http://localhost:5000/question/create/ 페이지가 POST 방식으로 호출(request.method == 'POST')되어 데이터가 저장되됨


#### 폼 위젯

{{ form.subject() }} 와 같은 코드는 HTML코드가 자동으로 생성되기 때문에 부트스트랩을 적용할 수가 없음

템플릿을 수정해보자

`pybo\templates\question\question_form.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">

        {{ form.subject.label }}
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {{ form.subject(class="form-control") }}
        <!-- ---------------------------------------------------------------------------------------- -->

        {{ form.content.label }}
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {{ form.content(class="form-control") }}
        <!-- ---------------------------------------------------------------------------------------- -->

        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```

form.content / form.subject 에 `class="form-control"` 처럼 부트스트랩 클래스를 적용할 수 있음

#### 수동 폼 작성

`{{ form.subject() }}` 와 같은 자동생성 코드를 사용하면 빠르게 만들 수 있지만 HTML 코드가 자동생성되어 디자인이 망함

태그 추가나 속성 추가도 어려움

자동 HTML 코드가 아닌 직접 HTML코드를 작성해보자

질문등록 템플릿을 수정한다.

`pybo\templates\question\question_form.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        <div class="form-group">
            <label for="subject">제목</label>
            <input type="text" class="form-control" name="subject" id="subject">
        </div>
        <div class="form-group">
            <label for="content">내용</label>
            <textarea class="form-control" name="content" id="content" rows="5"></textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
        <!-- ---------------------------------------------------------------------------------------- -->
    </form>
</div>
{% endblock %}
```

조금 달라지지만 수동이 나을 것 같음.

### 질문등록 오류

현재 질문등록을 해도 질문등록 화면이 계속 나옴

왜냐하면 create 함수의 `form.validate_on_submit()`가 실패하기 때문임

어떤 이유로 그러는지 알기 위해 오류를 표시하도록 수정한다.

`pybo\templates\question\question_form.html]`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        <!-- 오류표시 Start -->
        {% for field, errors in form.errors.items() %}
        <div class="alert alert-danger" role="alert">
            <strong>{{ form[field].label }}</strong>: {{ ', '.join(errors) }}
        </div>
        {% endfor %}
        <!-- 오류표시 End -->
        <!-- ---------------------------------------------------------------------------------------- -->
        <div class="form-group">
            <label for="subject">제목</label>
            <input type="text" class="form-control" name="subject" id="subject">
        </div>
        <div class="form-group">
            <label for="content">내용</label>
            <textarea class="form-control" name="content" id="content" rows="5"></textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```

create함수에서 form.validate_on_submit() 코드가 실패하면 폼에는 오류내용이 자동으로 등록됨

등록된 오류는 form.errors 속성을 사용하여 위와 같이 표시할 수 있음

form.errors.items 의 field는 subject나 content와 같은 입력폼의 필드를 의미함

그 후 아무것도 입력하지 않고 저장하기를 누르면 오류가 발생한다.

세가지 오류 중 마지막 오류인 CSRF Token 오류는 이해가 안된다.

CSRF는 보안에 관련 항목으로 form으로 전송되ㅑㄴ 데이터가 실제 웹페이지에서 작성된 데이터인지 판단하는 역할을 한다.

비정상적으로 폼데이터를 강제로 생성시도하면 CSRF 데이터를 생성할수 없기에 오류가 발생할 수 있다.

이 오류를 해결하기 위해 폼 태그 바로 밑에 `{{ form.csrf_token }}` 코드를 삽입한다.

`pybo\templates\question\question_form.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {{ form.csrf_token }}
        <!-- ---------------------------------------------------------------------------------------- -->
        ...
    </form>
</div>
{% endblock %}
```

### 사라진 필드

질문등록시 "질문"에는 내용을 입력하고 "내용"에는 아무런 값도 입력하지 않을 경우 "내용" 입력하라고 오류가 발생

하지만 이미 등록했던 제목값이 사라지는데 이를 방지하기 위해 템플릿 수정

`pybo\templates\question\question_form.html`

```html
<...>
<div class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form method="post" class="post-form my-3">
        <...>
        <div class="form-group">
            <label for="subject">제목</label>
            <!-- ---------------------------------------- [edit] ---------------------------------------- -->
            <input type="text" class="form-control" name="subject" id="subject"
                   value="{{ form.subject.data or '' }}">
            <!-- ---------------------------------------------------------------------------------------- -->
        </div>
        <div class="form-group">
            <label for="content">내용</label>
            <!-- ---------------------------------------- [edit] ---------------------------------------- -->
            <textarea class="form-control" name="content"
                      id="content" rows="10">{{ form.content.data or '' }}</textarea>
            <!-- ---------------------------------------------------------------------------------------- -->
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
```

필드 값에 `{{ form.subject.data or '' }}` 을 추가해서 해결

GET으로 요청되리 경우 기존 값이 없으므로 '' 을 해줌 ''이 없으면 None


### 오류메시지

오류 메시지를 변경해보자

`myproject\pybo\forms.py`

```python
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField
from wtforms.validators import DataRequired


class QuestionForm(FlaskForm):
    # ---------------------------------------- [edit] ---------------------------------------- #
    subject = StringField('제목', validators=[DataRequired('제목은 필수입력 항목입니다.')])
    content = TextAreaField('내용', validators=[DataRequired('내용은 필수입력 항목입니다.')])
    # ---------------------------------------------------------------------------------------- #
```

### 답변등록

답변등록에서도 플라스크 폼을 사용해보자

답변등록시 사용할 AnswerForm을 forms.py 파일에 만들자

`myproject\pybo\forms.py`

```python
...

# ---------------------------------------- [edit] ---------------------------------------- #
class AnswerForm(FlaskForm):
    content = TextAreaField('내용', validators=[DataRequired('내용은 필수입력 항목입니다.')])
# ---------------------------------------------------------------------------------------- #
```

answer_views.py의 create 함수를 변경하자

`myproject\pybo\views\answer_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, request, render_template
# ---------------------------------------------------------------------------------------- #
from werkzeug.utils import redirect

from .. import db
# ---------------------------------------- [edit] ---------------------------------------- #
from ..forms import AnswerForm
# ---------------------------------------------------------------------------------------- #
from ..models import Question, Answer

bp = Blueprint('answer', __name__, url_prefix='/answer')


@bp.route('/create/<int:question_id>', methods=('POST',))
def create(question_id):
    # ---------------------------------------- [edit] ---------------------------------------- #
    form = AnswerForm()
    question = Question.query.get_or_404(question_id)
    if form.validate_on_submit():
        content = request.form['content']
        answer = Answer(content=content, create_date=datetime.now())
        question.answer_set.append(answer)
        db.session.commit()
        return redirect(url_for('question.detail', question_id=question_id))
    return render_template('question/question_detail.html', question=question, form=form)
    # ---------------------------------------------------------------------------------------- #
```

상세조회 템플릿도 CSRF와 오류를 표시하기 위한 영역을 추가하자

`pybo\templates\question\question_detail.html`

```html
<...>
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <...>
    <form action="{{ url_for('answer.create', question_id=question.id) }}" method="post" class="my-3">
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {{ form.csrf_token }}
        <!-- 오류표시 Start -->
        {% for field, errors in form.errors.items() %}
        <div class="alert alert-danger" role="alert">
            <strong>{{ form[field].label }}</strong>: {{ ', '.join(errors) }}
        </div>
        {% endfor %}
        <!-- 오류표시 End -->
        <!-- ---------------------------------------------------------------------------------------- -->
        <div class="form-group">
            <textarea name="content" id="content" class="form-control" rows="10"></textarea>
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary">
    </form>
</div>
{% endblock %}
<...>
```

상세조회 템플릿에 폼이 추가되었으므로 question_views.py의 detail 함수도 폼을 사용하도록 수정

`myproject\pybo\views\question_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from ..forms import QuestionForm, AnswerForm
# ---------------------------------------------------------------------------------------- #
...

@bp.route('/detail/<int:question_id>/')
def detail(question_id):
    # ---------------------------------------- [edit] ---------------------------------------- #
    form = AnswerForm()
    question = Question.query.get_or_404(question_id)
    return render_template('question/question_detail.html', question=question, form=form)
    # ---------------------------------------------------------------------------------------- #

...
```

테스트해보자 !