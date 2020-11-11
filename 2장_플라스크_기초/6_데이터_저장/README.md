# 6. 데이터 저장

이번에는 페이지 요청을 통해 데이터를 저장하는 법을 알아보자.

### 답변등록 버튼

상세조회 템플릿 파일에 답변을 저장할 수 있는 기능을 추가하자

`question_detail.html`

```html
<h1>{{ question.subject }}</h1>

<div>
    {{ question.content }}
</div>

<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<form action="{{ url_for('answer.create', question_id=question.id) }}" method="post">
<textarea name="content" id="content" rows="15"></textarea>
<input type="submit" value="답변등록">
</form>
<!-- ---------------------------------------------------------------------------------------- -->
```

답변 내용을 입력할 수 있는 텍스트창이 추가되고 답변을 저장할 수 있는 "답변등록" 버튼이 추가됨.

답변을 저장하는 URL은 form 태그의 action 속성에 지정된 `url_for('answer.create', question_id=question.id)` 임


### 블루프린트 추가

현재 상세조회 페이지를 조회하면 `answer.create`에 해당하는 URL을 찾을수 없다는 오류가 발생함

answer에 해당하는 블루프린트 파일을 작성하고 create 함수를 생성해야함

다음과 같이 answer_views.py를 작성하자

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from datetime import datetime

from flask import Blueprint, url_for, request
from werkzeug.utils import redirect

from pybo import db
from pybo.models import Question, Answer

bp = Blueprint('answer', __name__, url_prefix='/answer')


@bp.route('/create/<int:question_id>', methods=('POST',))
def create(question_id):
    question = Question.query.get_or_404(question_id)
    content = request.form['content']
    answer = Answer(content=content, create_date=datetime.now())
    question.answer_set.append(answer)
    db.session.commit()
    return redirect(url_for('question.detail', question_id=question_id))
# ---------------------------------------------------------------------------------------- #
```
create함수의 매개변수 question_id는 URL매핑에 의해 전달된다. 

만약 http://locahost:5000/answer/create/2/ 라는 페이지가 요청되었다면 question_id에는 2라는 값이 전달될 것이다.

그리고 @bp.route의 methods속성에 "POST"라는 값을 지정해 주어야 한다.

methods는 브라우저에서 요청하는 GET이나 POST와 같은 폼 방식을 의미한다. 

답변을 저장하는 템플릿의 폼은 다음과 같이 "post" 방식이었다.

### 블루프린트 적용

그리고 answer_views.py의 블루프린트 객체를 pybo/__init__.py 파일에 다음처럼 등록해 주자.

```python
...

def create_app():
    ...

    # 블루프린트
    # ---------------------------------------- [edit] ---------------------------------------- #
    from .views import main_views, question_views, answer_views
    app.register_blueprint(main_views.bp)
    app.register_blueprint(question_views.bp)
    app.register_blueprint(answer_views.bp)
    # ---------------------------------------------------------------------------------------- #

    return app

```

### 답변저장

이렇게 수정한 후 상세조회 화면을 호출하면 다음과 같은 화면을 볼 수 있음

답변을 등록해도 반응이 없음 기능을 추가하지 않았기 때문


### 답변조회


등록된 답변을 화면에 표시하려면 다음과 같이 상세조회 템플릿을 수정해야 함

`pybo\templates\question\question_detail.html`

```html
<h1>{{ question.subject }}</h1>

<div>
    {{ question.content }}
</div>

<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<h5>{{ question.answer_set|length }}개의 답변이 있습니다.</h5>
<div>
    <ul>
    {% for answer in question.answer_set %}
        <li>{{ answer.content }}</li>
    {% endfor %}
    </ul>
</div>
<!-- ---------------------------------------------------------------------------------------- -->

<form action="{{ url_for('answer.create', question_id=question.id) }}" method="post">
<textarea name="content" id="content" rows="15"></textarea>
<input type="submit" value="답변등록">
</form>
```

`{{ question.answer_set|length }}` 은 답변의 총 갯수를 나타냄

length는 템플릿 필터 중 하나이고 필터는 `|` 문자 뒤에 추가하여 사용 가능

더 많은 필터는 https://jinja.palletsprojects.com/en/2.11.x/templates/#builtin-filters 여기서 확인가능

추가하고 확인하면 답변을 확인할 수 있으며 저장이 가능함.
