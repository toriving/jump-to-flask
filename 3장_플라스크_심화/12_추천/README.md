# 12. 추천

이번장에서는 추천기능을 추가해보자

### 모델변경

질문이나 답변에 "추천"을 적용하려면 질문모델과 답변모델에 "추천인" 이라는 속성을 추가해야 한다.

하나의 질문에 여러명이 추천할 수 있고 한 명이 여러개의 질문에 추천할 수 있으므로 이런 경우에는 "다대다(N:N)" 관계를 의미하는 ManyToMany 관계를 사용해야 한다.

SQLAlchemy에서 ManyToMany 관계를 적용하는 방법은 다음과 같다.

`pybo\models.py`

```python
from pybo import db

# ---------------------------------------- [edit] ---------------------------------------- #
question_voter = db.Table(
    'question_voter',
    db.Column('user_id', db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), primary_key=True),
    db.Column('question_id', db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'), primary_key=True)
)
# ---------------------------------------------------------------------------------------- #

class Question(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    subject = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    user = db.relationship('User', backref=db.backref('question_set'))
    modify_date = db.Column(db.DateTime(), nullable=True)
    # ---------------------------------------- [edit] ---------------------------------------- #
    voter = db.relationship('User', secondary=question_voter, backref=db.backref('question_voter_set'))
    # ---------------------------------------------------------------------------------------- #
...
```

먼저 계정과 질문을 한 쌍으로 갖는 question_voter 테이블 객체를 생성하였다. 

계정과 질문이 모두 프라이머리키이므로 ManyToMany 관계가 성립되는 테이블이다.

이제 마찬가지 방법으로 Answer모델에도 추천인(voter) 속성을 다음처럼 추가하도록 하자.

`pybo\models.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
answer_voter = db.Table(
    'answer_voter',
    db.Column('user_id', db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), primary_key=True),
    db.Column('answer_id', db.Integer, db.ForeignKey('answer.id', ondelete='CASCADE'), primary_key=True)
)
# ---------------------------------------------------------------------------------------- #
...

class Answer(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    question_id = db.Column(db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'))
    question = db.relationship('Question', backref=db.backref('answer_set'))
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    user = db.relationship('User', backref=db.backref('answer_set'))
    modify_date = db.Column(db.DateTime(), nullable=True)
    # ---------------------------------------- [edit] ---------------------------------------- #
    voter = db.relationship('User', secondary=answer_voter, backref=db.backref('answer_voter_set'))
    # ---------------------------------------------------------------------------------------- #
...
```

migrate and update

### 질문추천

이번에는 질문을 추천할 수 있는 기능을 만들어 보자. 

추천을 할 수 있는 곳은 상세조회 화면이므로 상세조회 템플릿을 다음과 같이 수정하자.
#### 질문추천 버튼

이번에는 템플릿의 구조를 꽤 많이 변경해야 하므로 따라하기 어려울 수 있다. 

책 페이지 하단에 템플릿 파일의 풀소스를 볼 수 있는 링크가 있으니 실제 어떻게 변경되었는지 꼭 확인하고 따라해 보기로 하자.


`\pybo\templates\question\question_detail.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <div class="row my-3">
        <div class="col-1"> <!-- 추천영역 -->
            <div class="bg-light text-center p-3 border font-weight-bolder mb-1">{{ question.voter|length }}</div>
            <a href="#" data-uri="{{ url_for('vote.question', question_id=question.id) }}"
               class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
        </div>
        <div class="col-11"> <!-- 질문영역 -->
    <!-- ---------------------------------------------------------------------------------------- -->    
            <!-- 기존내용 -->
            <div class="card">
                <div class="card-body"> <!-- my-3 삭제 -->
                   ...
                </div>
            </div>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->            
        </div>
    </div>
    <!-- ---------------------------------------------------------------------------------------- -->
    <h5 class="border-bottom my-3 py-2">{{ question.answer_set|length }}개의 답변이 있습니다.</h5>
    {% for answer in question.answer_set %}
    <...>
<...>
```

추천영역은 강조하기 위해 영역을 다음과 같이 분리해 주었다

```html
<div class="row">
    <div class="col-1">
        추천영역
    </div>
    <div class="col-11">
        질문영역
    </div>
</div>
```

1/12 만큼은 추천영역이 차지하고 11/12 만큼은 기존 질문영역이 차지할 수 있도록 부트스트랩의 row와 col-1, col-11을 이용하여 구성하였다.

추천영역에는 이 질문을 추천한 건수를 보여주고 추천을 할 수 있는 버튼을 추가하였다. 

질문영역에 해당되는 부분은 기존의 내용을 그대로 복사하였다.

그리고 추천버튼을 클릭했을때 "정말로 추천하시겠습니까?" 라는 확인창이 나타나도록 다음과 같은 자바스크립트를 추가해 주자.

`pybo\templates\question\question_detail.html`

```html
<...>
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    ...
    /* ---------------------------------------- [edit] ---------------------------------------- */
    $(".recommend").on('click', function() {
        if(confirm("정말로 추천하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
    /* ---------------------------------------------------------------------------------------- */
});
</script>
{% endblock %}
```

추천버튼에 `class="recommend"`가 적용되어 있으므로 추천버튼을 클릭하면 "정말로 추천하시겠습니까?"라는 질문이 나타나고 "확인"을 선택하면 data-uri 속성에 정의된 URL이 호출될 것이다.

#### 질문추천 함수

추천등록시 `url_for('vote.question', question_id=question.id)` 라는 URL이 추가되었으므로 라우트 함수를 추가해야 한다. 

vote_views.py 파일을 신규로 생성하고 다음처럼 question 함수를 작성하자.

`pybo\views\vote_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, flash, g
from werkzeug.utils import redirect

from pybo import db
from pybo.models import Question
from pybo.views.auth_views import login_required

bp = Blueprint('vote', __name__, url_prefix='/vote')


@bp.route('/question/<int:question_id>/')
@login_required
def question(question_id):
    _question = Question.query.get_or_404(question_id)
    if g.user == _question.user:
        flash('본인이 작성한 글은 추천할수 없습니다')
    else:
        _question.voter.append(g.user)
        db.session.commit()
    return redirect(url_for('question.detail', question_id=question_id))
# ---------------------------------------------------------------------------------------- #    
```

본인 추천을 방지하기 위해 로그인한 사용자와 추천하려는 질문의 글쓴이가 동일할 경우에는 추천시 오류가 발생하도록 하였다.

Question 모델의 vorter는 여러사람을 추가할 수 있는 ManyToMany 관계이므로 `_question.voter.add(request.user)` 처럼 add 함수를 사용하여 추천인을 추가해야 한다.

#### 블루프린트 추가

블루프린트 파일(vote_views.py)이 생성되었으므로 잊지말고 `pybo/__init__.py` 파일에 다음처럼 해당내용을 추가해 주자.

`pybo\__init__.py`

```python
...

def create_app():
    ...

    # 블루프린트
    # ---------------------------------------- [edit] ---------------------------------------- #
    from .views import main_views, question_views, answer_views, auth_views, comment_views, vote_views
    # ---------------------------------------------------------------------------------------- #    
    app.register_blueprint(main_views.bp)
    app.register_blueprint(question_views.bp)
    app.register_blueprint(answer_views.bp)
    app.register_blueprint(auth_views.bp)
    app.register_blueprint(comment_views.bp)
    # ---------------------------------------- [edit] ---------------------------------------- #
    app.register_blueprint(vote_views.bp)
    # ---------------------------------------------------------------------------------------- #    

    # 필터
    from .filter import format_datetime
    app.jinja_env.filters['datetime'] = format_datetime

    return app
```

vote_views의 블루프린트 객체 bp를 등록하였다.

#### 오류표시

그리고 "본인이 작성한 글은 추천할수 없습니다" 라는 오류를 표시하기 위해 상세조회 화면 상단에 다음처럼 오류를 표시할수 있는 영역을 추가해 주자.

`pybo\templates\question\question_detail.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    {% for message in get_flashed_messages() %}
    <div class="alert alert-danger" role="alert">
        {{ message }}
    </div>
    {% endfor %}
    <!-- ---------------------------------------------------------------------------------------- -->
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <...>
<...>
```

상세조회 화면 상단에 라우트 함수에서 flash로 생성된 오류를 표시할 수 있는 영역을 추가해 주었다.

#### 테스트

테스트해보자


### 답변추천

답변추천은 질문추천과 동일하므로 빠르게 작성해 보자.

#### 답변추천 버튼

답변의 추천수를 표시하고 답변 추천을 할 수있는 버튼을 질문상세 화면에 다음과 같이 추가하자.

`pybo\templates\question\question_detail.html`

```html
<...>
<h5 class="border-bottom my-3 py-2">{{ question.answer_set|length }}개의 답변이 있습니다.</h5>
{% for answer in question.answer_set %}
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<div class="row my-3">
    <div class="col-1">  <!-- 추천영역 -->
        <div class="bg-light text-center p-3 border font-weight-bolder mb-1">{{ answer.voter|length }}</div>
        <a href="#" data-uri="{{ url_for('vote.answer', answer_id=answer.id) }}"
            class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
    </div>
    <div class="col-11">  <!-- 답변영역 -->
<!-- ---------------------------------------------------------------------------------------- -->    
        <!-- 기존내용 -->
        <div class="card">  <!-- my-3 삭제 -->
            <div class="card-body">
                <...>
            </div>
        </div>
<!-- ---------------------------------------- [edit] ---------------------------------------- -->        
    </div>
</div>
<!-- ---------------------------------------------------------------------------------------- -->
{% endfor %}
<...>
```

#### 답변추천 함수

그리고 `url_for('vote.answer', answer_id=answer.id)` URL이 추가되었으므로 다음처럼 vote_views.py 파일에 answer 함수를 추가하자.

`pybo\views\vote_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from pybo.models import Question, Answer
# ---------------------------------------------------------------------------------------- #
...
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/answer/<int:answer_id>/')
@login_required
def answer(answer_id):
    _answer = Answer.query.get_or_404(answer_id)
    if g.user == _answer.user:
        flash('본인이 작성한 글은 추천할수 없습니다')
    else:
        _answer.voter.append(g.user)
        db.session.commit()
    return redirect(url_for('question.detail', question_id=_answer.question.id))
# ---------------------------------------------------------------------------------------- #
```

#### 테스트

테스트해보자

### 질문목록에 추천수 표시

질문에 추천수가 추가되었으므로 질문목록에도 추천수를 표시해 주면 좋을 것이다. 

다음처럼 질문목록 화면에도 질문의 추천수를 추가해 보자.

`pybo\templates\question\question_list.html`

```html
<...>
<table class="table">
    <thead>
    <tr class="text-center thead-dark">
        <th>번호</th>
        <!-- -------------------------------------- [edit] -------------------------------------- -->
        <th>추천</th>
        <!-- ------------------------------------------------------------------------------------ -->
        <th style="width:50%">제목</th>
        <th>글쓴이</th>
        <th>작성일시</th>
    </tr>
    </thead>
    <tbody>
    {% if question_list %}
    {% for question in question_list.items %}
    <tr class="text-center">
        <td>{{ question_list.total - ((question_list.page-1) * question_list.per_page) - loop.index0 }}</td>
        <!-- ------------------------------------ [edit] ------------------------------------ -->
        <td>
            {% if question.voter|length > 0 %}
            <span class="badge badge-warning px-2 py-1">{{ question.voter|length }}</span>
            {% endif %}
        </td>
        <!-- -------------------------------------------------------------------------------- -->
        <...>
    </tr>
    {% endfor %}
    {% else %}
    <...>
    {% endif %}
    </tbody>
</table>
<...>
```
