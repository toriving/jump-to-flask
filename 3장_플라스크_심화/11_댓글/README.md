# 11. 댓글

이번장에서는 댓글기능을 추가해보자

### 댓글 모델

댓글 작성을 위해서 먼저 준비해야 할 것은 댓글(Comment) 모델이다.

`myproject\pybo\models.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
class Comment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    user = db.relationship('User', backref=db.backref('comment_set'))
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
    modify_date = db.Column(db.DateTime())
    question_id = db.Column(db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'), nullable=True)
    question = db.relationship('Question', backref=db.backref('comment_set'))
    answer_id = db.Column(db.Integer, db.ForeignKey('answer.id', ondelete='CASCADE'), nullable=True)
    answer = db.relationship('Answer', backref=db.backref('comment_set'))
# ---------------------------------------------------------------------------------------- #
```

만약 질문에 댓글이 작성될 경우에는 question_id 속성에 값이 저장되고 답변에 댓글이 작성될 경우에는 answer_id 속성에 값이 저장될 것이다. 

따라서 댓글 모델의 question_id 또는 answer_id 둘 중에 하나에만 값이 저장되므로 두 개의 속성은 모두 nullable=True 를 설정해 주어야 한다. 

그리고 question_id, answer_id 속성에는 질문이나 답변이 삭제될 경우에 해당 댓글도 삭제될 수 있도록 ondelete='CASCADE' 옵션을 설정해 주었다.

모델이 추가되었으므로 migrate와 upgrade 명령을 하자

### 질문 댓글

#### 질문댓글 링크 추가

상세조회 템플릿에 댓글을 출력하는 부분과 댓글을 입력하는 링크를 다음과 같이 작성하자.

`pybo\templates\question\question_detail.html`

```html
<...>
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="card my-3">
        <div class="card-body">
            <...>
            {% endif %}
            <!-- ---------------------------------------- [edit] ---------------------------------------- -->
            <!-- 질문 댓글 Start -->
            {% if question.comment_set|length > 0 %}
            <div class="mt-3">
            {% for comment in question.comment_set %}
                <div class="comment py-2 text-muted">
                    <span style="white-space: pre-line;">{{ comment.content }}</span>
                    <span>
                        - {{ comment.user.username }}, {{ comment.create_date|datetime }}
                        {% if comment.modify_date %}
                        (수정:{{ comment.modify_date|datetime }})
                        {% endif %}
                    </span>
                    {% if g.user == comment.user %}
                    <a href="{{ url_for('comment.modify_question', comment_id=comment.id) }}" class="small">수정</a>,
                    <a href="#" class="small delete"
                       data-uri="{{ url_for('comment.delete_question', comment_id=comment.id) }}">삭제</a>
                    {% endif %}
                </div>
            {% endfor %}
            </div>
            {% endif %}
            <div>
                <a href="{{ url_for('comment.create_question', question_id=question.id) }}"
                   class="small"><small>댓글 추가 ..</small></a>
            </div>
            <!-- 질문 댓글 End -->
            <!-- ---------------------------------------------------------------------------------------- -->
        </div>
    </div>
    <...>
<...>
```

너무 많은 HTML코드가 추가된 것 같지만 어렵지 않은 내용이다. 가볍게 살펴보자.

`{% for comment in question.comment_set %} ... {% endfor %}` 부분은 질문에 등록된 댓글을 모두 보여주는 부분이다. 

댓글의 내용과 작성자, 작성일시 그리고 수정을 했다면 수정일시도 보여주게 되어 있다. 

그리고 댓글 작성자와 로그인한 사용자가 동일한 경우 "수정"과 "삭제" 링크가 노출되도록 하였다. 

그리고 for문 바깥쪽에 "댓글 추가.."라는 댓글을 작성할 수 있는 링크를 추가해 주었다.

for문에 의해 반복되는 댓글 각각은 comment라는 스타일 클래스를 따로 지정해 주었다. 

댓글 영역은 좀 작게 보여줄 필요가 있기 때문이다. 

지금까지 빈 파일로 되어있던 style.css에 comment 클래스에 대한 내용을 다음처럼 추가하도록 하자.

`myproject\pybo\static\style.css`

```css
.comment {
    border-top:dotted 1px #ddd;
    font-size:0.7em;
}
```

comment 클래스는 댓글 각각의 상단에 점선을 추가하고 폰트 사이즈를 0.7em으로 설정하는 스타일이다.

#### 질문댓글 폼

그리고 댓글 등록시 사용할 CommentForm을 forms.py 파일에 다음과 같이 추가하자.

`myproject\pybo\forms.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
class CommentForm(FlaskForm):
    content = TextAreaField('내용', validators=[DataRequired()])
# ---------------------------------------------------------------------------------------- #
```

댓글 작성시에는 content 필드만 사용되므로 간단하다.

#### 질문댓글 등록 함수

그리고 상세조회 템플릿에서 사용된 "댓글추가.."에 해당되는 create_question 함수를 다음과 같이 comment_views.py 파일에 작성하자.

`pybo\views\comment_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from datetime import datetime

from flask import Blueprint, url_for, request, render_template, g
from werkzeug.utils import redirect

from pybo import db
from pybo.forms import CommentForm
from pybo.models import Question, Comment
from pybo.views.auth_views import login_required

bp = Blueprint('comment', __name__, url_prefix='/comment')


@bp.route('/create/question/<int:question_id>', methods=('GET', 'POST'))
@login_required
def create_question(question_id):
    form = CommentForm()
    question = Question.query.get_or_404(question_id)
    if request.method == 'POST' and form.validate_on_submit():
        comment = Comment(user=g.user, content=form.content.data, create_date=datetime.now(), question=question)
        db.session.add(comment)
        db.session.commit()
        return redirect(url_for('question.detail', question_id=question_id))
    return render_template('comment/comment_form.html', form=form)
# ---------------------------------------------------------------------------------------- #
```

질문에 댓글을 추가하는 create_question 함수를 위와같이 작성하였다. 

질문이나 답변을 작성하는 것과 동일한 방법이므로 자세한 설명은 생략하기로 한다.

여기서 한가지 주의해서 볼 점은 질문에 대한 댓글이므로 Comment 모델 객체 생성시 question속성에만 값을 설정했다는 점이다. 

만약 답변에 대한 댓글이었다면 question 속성이 아닌 answer속성에 값을 설정해야 할 것이다.

#### 블루프린트 추가

comment_views.py에` bp = Blueprint('comment', __name__, url_prefix='/comment')` 라는 블루프린트가 추가되었으므로 `pybo/__init__.py` 파일에 블루프린트를 등록해 주어야 한다. 

다음과 같이 `pybo/__init__.py` 파일을 수정하자.

```python
...

def create_app():
    ...

    # 블루프린트
    # ---------------------------------------- [edit] ---------------------------------------- #
    from .views import main_views, question_views, answer_views, auth_views, comment_views
    # ---------------------------------------------------------------------------------------- #
    app.register_blueprint(main_views.bp)
    app.register_blueprint(question_views.bp)
    app.register_blueprint(answer_views.bp)
    app.register_blueprint(auth_views.bp)
    # ---------------------------------------- [edit] ---------------------------------------- #
    app.register_blueprint(comment_views.bp)
    # ---------------------------------------------------------------------------------------- #

    ...

```

comment_views.py 파일의 블루프린트 객체를 등록해 주었다.

#### 질문댓글 템플릿

댓글을 등록하기 위한 comment_form.html 템플릿은 다음과 같이 작성하자.

`pybo\templates\comment\comment_form.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <h5 class="border-bottom pb-2">댓글등록하기</h5>
    <form method="post" class="post-form my-3">
        {{ form.csrf_token }}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="content">댓글내용</label>
            <textarea class="form-control"name="content" id="content"
                      rows="3">{{ form.content.data or '' }}</textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}

<!-- ---------------------------------------------------------------------------------------- -->
```

댓글의 내용만 입력할 수 있는 매우 간단한 폼이다.

#### 질문댓글 수정함수

그리고 코멘트를 수정하는 `url_for('comment.modify_question', comment_id=comment.id)` URL에 매핑되는 modify_question 함수를 다음과 같이 작성하자.

`pybo\views\comment_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, request, render_template, g, flash
# ---------------------------------------------------------------------------------------- #
...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/modify/question/<int:comment_id>', methods=('GET', 'POST'))
@login_required
def modify_question(comment_id):
    comment = Comment.query.get_or_404(comment_id)
    if g.user != comment.user:
        flash('수정권한이 없습니다')
        return redirect(url_for('question.detail', question_id=comment.question.id))
    if request.method == 'POST':
        form = CommentForm()
        if form.validate_on_submit():
            form.populate_obj(comment)
            comment.modify_date = datetime.now()  # 수정일시 저장
            db.session.commit()
            return redirect(url_for('question.detail', question_id=comment.question.id))
    else:
        form = CommentForm(obj=comment)
    return render_template('comment/comment_form.html', form=form)
# ---------------------------------------------------------------------------------------- #
```

질문의 댓글을 등록하는 함수와 크게 다르지 않다. 

GET방식일 경우 기존의 코멘트를 조회하여 리턴하고 POST방식일 경우 폼으로 받은 내용으로 댓글을 업데이트 해 준다. 

댓글 수정이므로 modify_date에도 수정일시를 업데이트 해 주었다.

#### 질문댓글 삭제함수

그리고 코멘트를 삭제하는` url_for('comment.delete_question', comment_id=comment.id)` URL에 매핑되는 delete_question 함수를 다음과 같이 작성하자.

`pybo\views\comment_views.py`

```python
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/delete/question/<int:comment_id>')
@login_required
def delete_question(comment_id):
    comment = Comment.query.get_or_404(comment_id)
    question_id = comment.question.id
    if g.user != comment.user:
        flash('삭제권한이 없습니다')
        return redirect(url_for('question.detail', question_id=question_id))
    db.session.delete(comment)
    db.session.commit()
    return redirect(url_for('question.detail', question_id=question_id))
# ---------------------------------------------------------------------------------------- #
```

댓글을 삭제한 후에는 댓글을 삭제한 상세조회 화면으로 리다이렉트 하였다.

#### 테스트

이상과 같이 수정하면 다음처럼 상세조회 화면에 "댓글추가.." 라는 링크가 노출될 것이다.

"댓글추가.." 링크를 클릭하여 댓글을 추가해 보자. 그리고 수정과 삭제도 정상적으로 동작하는지 확인해 보자.


### 답변 댓글

답변에 댓글을 추가하는 것은 질문에 댓글을 추가하는 것과 큰 차이가 없다. 빠르게 알아보도록 하자.

#### 답변댓글 링크

먼저 상세조회 템플릿의 답변 목록을 다음과 같이 수정하자.

`pybo\templates\question\question_detail.html`

```html
<...>
    <...>
    {% for answer in question.answer_set %}
    <div class="card my-3">
        <div class="card-body">
            <...>
            {% endif %}
            <!-- -------------------------------------- [edit] -------------------------------------- -->
            <!-- 답변 댓글 Start -->
            {% if answer.comment_set|length > 0 %}
            <div class="mt-3">
            {% for comment in answer.comment_set %}
                <div class="comment py-2 text-muted">
                    <span style="white-space: pre-line;">{{ comment.content }}</span>
                    <span>
                        - {{ comment.user.username }}, {{ comment.create_date|datetime }}
                        {% if comment.modify_date %}
                        (수정:{{ comment.modify_date|datetime }})
                        {% endif %}
                    </span>
                    {% if g.user == comment.user %}
                    <a href="{{ url_for('comment.modify_answer', comment_id=comment.id) }}" class="small">수정</a>,
                    <a href="#" class="small delete"
                       data-uri="{{ url_for('comment.delete_answer', comment_id=comment.id) }}">삭제</a>
                    {% endif %}
                </div>
            {% endfor %}
            </div>
            {% endif %}
            <div>
                <a href="{{ url_for('comment.create_answer', answer_id=answer.id) }}"
                   class="small"><small>댓글 추가 ..</small></a>
            </div>
            <!-- 답변 댓글 End -->
            <!-- ------------------------------------------------------------------------------------ -->
        </div>
    </div>
    {% endfor %}
    <...>
<...>
```

질문 댓글과 차이가 없다. 다만 `question.comment_set` 대신 `answer.comment_set`이 이용되었을 뿐이다. 

그리고 답변의 댓글을 추가, 수정, 삭제시 호출하는 URL함수명이 살짝 다르다.

#### 답변댓글 함수

답변 댓글에 사용된 등록, 수정, 삭제 함수는 질문 댓글과 큰 차이가 없으므로 한번에 알아보기로 하자.

`pybo\views\comment_views.py`

```python
...
from pybo.models import Question, Comment, Answer
...
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/create/answer/<int:answer_id>', methods=('GET', 'POST'))
@login_required
def create_answer(answer_id):
    form = CommentForm()
    answer = Answer.query.get_or_404(answer_id)
    if request.method == 'POST' and form.validate_on_submit():
        comment = Comment(user=g.user, content=form.content.data, create_date=datetime.now(), answer=answer)
        db.session.add(comment)
        db.session.commit()
        return redirect(url_for('question.detail', question_id=answer.question.id))
    return render_template('comment/comment_form.html', form=form)


@bp.route('/modify/answer/<int:comment_id>', methods=('GET', 'POST'))
@login_required
def modify_answer(comment_id):
    comment = Comment.query.get_or_404(comment_id)
    if g.user != comment.user:
        flash('수정권한이 없습니다')
        return redirect(url_for('question.detail', question_id=comment.answer.id))
    if request.method == 'POST':
        form = CommentForm()
        if form.validate_on_submit():
            form.populate_obj(comment)
            comment.modify_date = datetime.now()  # 수정일시 저장
            db.session.commit()
            return redirect(url_for('question.detail', question_id=comment.answer.question.id))
    else:
        form = CommentForm(obj=comment)
    return render_template('comment/comment_form.html', form=form)


@bp.route('/delete/answer/<int:comment_id>')
@login_required
def delete_answer(comment_id):
    comment = Comment.query.get_or_404(comment_id)
    question_id = comment.answer.question.id
    if g.user != comment.user:
        flash('삭제권한이 없습니다')
        return redirect(url_for('question.detail', question_id=question_id))
    db.session.delete(comment)
    db.session.commit()
    return redirect(url_for('question.detail', question_id=question_id))
# ---------------------------------------------------------------------------------------- #
```

답변 댓글에 해당하는 create_answer, modify_answer, delete_answer 총 3개의 함수를 추가하였다.

질문댓글 함수들과 거의 차이가 없으므로 별도의 설명은 생략하기로 한다. 단, 답변의 댓글인 경우 question_id 값을 알기 위해 `comment.answer.question.id` 처럼 답변(answer)을 통해 질문(question)을 얻어야 한다는 점이 조금 다르다.