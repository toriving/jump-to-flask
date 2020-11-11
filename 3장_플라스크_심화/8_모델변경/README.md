# 8. 모델변경

Question과 Answer 모델에 "글쓴이"에 해당되는 user 속성을 추가하자

### SQLite 패치

`pybo/__init__.py` 수정

```python
from flask import Flask
from flask_migrate import Migrate
from flask_sqlalchemy import SQLAlchemy
# ---------------------------------------- [edit] ---------------------------------------- #
from sqlalchemy import MetaData
# ---------------------------------------------------------------------------------------- #

import config

# ---------------------------------------- [edit] ---------------------------------------- #
naming_convention = {
    "ix": 'ix_%(column_0_label)s',
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(column_0_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s"
}
db = SQLAlchemy(metadata=MetaData(naming_convention=naming_convention))
migrate = Migrate()
# ---------------------------------------------------------------------------------------- #

def create_app():
    app = Flask(__name__)
    app.config.from_object(config)

    # ORM
    db.init_app(app)
    # ---------------------------------------- [edit] ---------------------------------------- #
    if app.config['SQLALCHEMY_DATABASE_URI'].startswith("sqlite"):
        migrate.init_app(app, db, render_as_batch=True)
    else:
        migrate.init_app(app, db)
    # ---------------------------------------------------------------------------------------- #        
    from . import models
    ...
```

SQLite 데이터베이스에서 사용하는 인덱스등의 제약조건의 이름규칙을 MetaData 클래스를 사용하여 정의해 주어야 한다. 

이렇게 이름을 정의하지 않으면 SQLite 데이터베이스는 오류를 발생시킨다.

이렇게 변경하면 제약조건의 이름들이 변경되기 때문에 flask db migrate 명령과 flask db upgrade 명령으로 데이터베이스를 변경해 주어야 한다.

### Question 속성추가

이제 질문 모델에 작성자를 추가해 보도록 하자. 

Question모델에 user 속성을 다음과 같이 추가하자.

`myproject\pybo\models.py`

```python
...
class Question(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    subject = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    # ---------------------------------------- [edit] ---------------------------------------- #
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    user = db.relationship('User', backref=db.backref('question_set'))
    # ---------------------------------------------------------------------------------------- #
...
```

`$ flask db migrate && flask db upgrade`

upgrade에서 오류 발생

user_id 속성이 Null을 허용하지 않도록 `nullable=False`로 지정이 되었기 때문

데이터베이스에는 이미 많은 Question 모델 데이터가 저장되어 있고 계정에 대한 정보는 없기 때문에 오류가 발생하는 것이다.

이러한 문제를 해결하려면 다음과 같은 해법을 사용해야 한다.

1. nullable=False 대신 nullable=True 로 바꾸고 기존 데이터의 user_id를 모두 1로 설정
2. flask db migrate 명령과 flask db upgrade 명령 실행
3. 다시 nullable=False 로 변경
4. flask db migrate 명령과 flask db upgrade 명령 실행

이미 테이블에 데이터가 있을경우 Not Null 신규컬럼 추가시에는 좀 절차가 복잡하지만 어쩔수 없는 부분이다. 

우선 nullable=False 대신 nullable=True 로 변경하고 디폴트 값을 1로 설정해 보자. 

여기서 사용한 디폴트 값 1은 최초로 생성한 계정(User)모델의 id번호이다.

`myproject\pybo\models.py`

```python
...
class Question(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    subject = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    # ---------------------------------------- [edit] ---------------------------------------- #
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=True, server_default='1')
    # ---------------------------------------------------------------------------------------- #
    user = db.relationship('User', backref=db.backref('question_set'))
...
```

`$ flask db stamp heads` -> `$ flask db migrate && flask db upgrade`

이제 다시 `nullable=False`로 변경하자

`myproject\pybo\models.py`

```python
...
class Question(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    subject = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    # ---------------------------------------- [edit] ---------------------------------------- #
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    # ---------------------------------------------------------------------------------------- #
    user = db.relationship('User', backref=db.backref('question_set'))
...
```

`$ flask db migrate && flask db upgrade`

### Answer 속성추가

같은 방법으로 진행하자

`myproject\pybo\models.py`

```python
...
class Answer(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    question_id = db.Column(db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'))
    question = db.relationship('Question', backref=db.backref('answer_set'))
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    # ---------------------------------------- [edit] ---------------------------------------- #
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=True, server_default='1')
    user = db.relationship('User', backref=db.backref('answer_set'))
    # ---------------------------------------------------------------------------------------- #
...    
```
`$ flask db migrate && flask db upgrade`

`myproject\pybo\models.py`

```python
...
class Answer(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    question_id = db.Column(db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'))
    question = db.relationship('Question', backref=db.backref('answer_set'))
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    # ---------------------------------------- [edit] ---------------------------------------- #
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    user = db.relationship('User', backref=db.backref('answer_set'))
    # ---------------------------------------------------------------------------------------- #
...    
```

`$ flask db migrate && flask db upgrade`

### 속성 적용

이제 Question과 Answer 모델에 user 속성이 추가되었으므로 질문과 답변을 등록하는 프로그램에도 user 속성을 추가하도록 수정해 주어야 한다.

먼저 답변을 등록하는 부분을 다음처럼 수정하자.

`pybo\views\answer_views.py`

```python
from datetime import datetime
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, request, render_template, g
# ---------------------------------------------------------------------------------------- #
from werkzeug.utils import redirect
...

@bp.route('/create/<int:question_id>', methods=('POST',))
def create(question_id):
    form = AnswerForm()
    question = Question.query.get_or_404(question_id)
    if form.validate_on_submit():
        content = request.form['content']
        # ---------------------------------------- [edit] ---------------------------------------- #
        answer = Answer(content=content, create_date=datetime.now(), user=g.user)
        # ---------------------------------------------------------------------------------------- #
        question.answer_set.append(answer)
        db.session.commit()
        return redirect(url_for('question.detail', question_id=question_id))
    return render_template('question/question_detail.html', question=question, form=form)
```

Answer 모델의 객체를 생성할때 user 속성으로 g.user를 추가해 주었다. 

g.user는 현재 로그인한 계정을 의미한다. 

g.user는 auth_views.py의 @bp.before_app_request 라우트의 load_logged_in_user 함수에서 자동으로 설정되기 때문에 이렇게 사용할 수 있다.

질문을 등록하는 부분도 마찬가지 방법으로 다음처럼 수정하자.

`pybo\views\question_views.py`

```python
from datetime import datetime
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, render_template, request, url_for, g
# ---------------------------------------------------------------------------------------- #
from werkzeug.utils import redirect
...

@bp.route('/create/', methods=('GET', 'POST'))
def create():
    form = QuestionForm()
    if request.method == 'POST' and form.validate_on_submit():
        # ---------------------------------------- [edit] ---------------------------------------- #
        question = Question(subject=form.subject.data, content=form.content.data,
                            create_date=datetime.now(), user=g.user)
        # ---------------------------------------------------------------------------------------- #
        db.session.add(question)
        db.session.commit()
        return redirect(url_for('main.index'))
    return render_template('question/question_form.html', form=form)
```

질문의 글쓴이는 현재 로그인 한 계정이므로 Question 모델 객체 생성시 user속성에 g.user를 대입해 주었다.

이제 로그인 후에 질문과 답변을 등록해 보면 문제없이 잘 등록되는 것을 확인할 수 있을 것이다.

### 로그인이 필요한 함수

하지만 로그아웃 상태에서 질문 또는 답변을 등록하면 다음과 비슷한 오류가 발생하게 된다.

로그아웃 상태에서 이런 오류가 발생하는 이유는 g.user가 None이기 때문이다. 

g.user는 로그인 후에만 값이 로그인한 계정으로 저장되기 때문이다.

이 문제를 해결하기 위해서는 로그인 유무를 판단하여 로그인을 했으면 그대로 수행하고 로그아웃 상태라면 로그인 페이지로 리다이렉트하는 로직을 추가해야 할 것이다.

하지만 모든 함수 상단에 이러한 로직을 추가하는 것은 좋지 않은 방법이다. 

이럴 경우에는 파이썬 데코레이터를 사용하는 것이 좋다.

먼저 auth_views.py 파일에 다음과 같은 함수를 생성해 보자.

`pybo\views\auth_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
import functools
# ---------------------------------------------------------------------------------------- #
...

def login_required(view):
    @functools.wraps(view)
    def wrapped_view(**kwargs):
        if g.user is None:
            return redirect(url_for('auth.login'))
        return view(**kwargs)
    return wrapped_view

```

`\pybo\views\question_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from pybo.views.auth_views import login_required
# ---------------------------------------------------------------------------------------- #
...

@bp.route('/create/', methods=('GET', 'POST'))
# ---------------------------------------- [edit] ---------------------------------------- #
@login_required
# ---------------------------------------------------------------------------------------- #
def create():
    ...
```

질문을 등록하는 함수인 create 함수에 @login_required 어노테이션을 추가해 주었다.

마찬가지로 답변을 등록하는 함수도 다음과 같이 수정하자.

`pybo\views\answer_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from .auth_views import login_required
# ---------------------------------------------------------------------------------------- #
...

@bp.route('/create/<int:question_id>', methods=('POST',))
# ---------------------------------------- [edit] ---------------------------------------- #
@login_required
# ---------------------------------------------------------------------------------------- #
def create(question_id):
    ...
```

이렇게 수정한 후 로그아웃 상태에서 질문을 등록하거나 답변을 등록하려고 하면 로그인 화면으로 이동되는 것을 확인할 수 있을 것이다.

### disabled

하지만 한가지 더 생각해 봐야 할 것이 있다. 

질문등록은 로그아웃 상태에서는 글 조차 작성할 수 없었지만 답변등록은 로그아웃 상태에서도 글을 작성할 수 있게 되어 있다. 

다만 답변을 작성후에 "저장하기" 버튼을 클릭하면 그때서야 로그인 화면으로 이동되고 작성한 글은 사라지게 된다.

작성한 글이 사라지는 문제를 해결하려면 로그아웃 상태에서는 아예 답변 작성을 못하게 막는 것이 좋을 것이다.

답변작성 템플릿을 다음처럼 수정해 보자.

`pybo\templates\question\question_detail.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <...>
    <form action="{{ url_for('answer.create', question_id=question.id) }}" method="post" class="my-3">
        <...>
        <div class="form-group">
            <!-- ---------------------------------------- [edit] ---------------------------------------- -->
            <textarea {% if not g.user %}disabled{% endif %}
                    name="content" id="content" class="form-control" rows="10"></textarea>
            <!-- ---------------------------------------------------------------------------------------- -->
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary">
    </form>
</div>
{% endblock %}
```

답변을 작성하는 textarea에 로그인 상태가 아니라면 disabled 를 지정하여 입력자체를 불가능하도록 설정하였다.