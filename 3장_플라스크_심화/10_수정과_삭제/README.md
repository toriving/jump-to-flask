# 10. 수정과 삭제

이번장에서는 작성한 질문과 답변을 수정하고 삭제할 수 있는 기능을 추가해 보자.

### 수정일시 추가

먼저 질문이나 답변이 언제 수정되었는지 확인할 수 있도록 Question과 Answer 모델에 수정일시(modify_date) 속성을 추가하자.

`pybo\models.py`

```python
from pybo import db


class Question(db.Model):
    ...
    # ---------------------------------------- [edit] ---------------------------------------- #
    modify_date = db.Column(db.DateTime(), nullable=True)
    # ---------------------------------------------------------------------------------------- #


class Answer(db.Model):
    ...
    # ---------------------------------------- [edit] ---------------------------------------- #
    modify_date = db.Column(db.DateTime(), nullable=True)
    # ---------------------------------------------------------------------------------------- #

...
```

모델이 변경되면 반드시 실행해야 하는 migrate와 upgrade 명령을 다음처럼 실행하자.

`$ flask db migrate && flask db upgrade`

### 질문수정

이제 상세조회 화면에 질문을 수정할 수 있는 버튼을 추가해 보자.

#### 짊문수정 버튼

`pybo\templates\question\question_detail.html`

```html
<...>
<div class="card my-3">
    <div class="card-body">
        <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
        <div class="d-flex justify-content-end">
            <div class="badge badge-light p-2 text-left">
                <div class="mb-2">{{ question.user.username }}</div>
                <div>{{ question.create_date }}</div>
            </div>
        </div>
        <!-- -------------------------------------- [edit] -------------------------------------- -->
        {% if g.user == question.user %}
        <div class="my-3">
            <a href="{{ url_for('question.modify', question_id=question.id) }}"
               class="btn btn-sm btn-outline-secondary">수정</a>
        </div>
        {% endif %}
        <!-- ------------------------------------------------------------------------------------ -->
    </div>
</div>
```

#### 질문수정 함수

그리고 상세조회 템플릿에 `url_for('question.modify', question_id=question.id)` 이라는 URL이 추가되었으므로 다음처럼 라우트 함수를 작성해야 한다.

`pybo\views\question_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, render_template, request, url_for, g, flash
# ---------------------------------------------------------------------------------------- #
...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/modify/<int:question_id>', methods=('GET', 'POST'))
@login_required
def modify(question_id):
    question = Question.query.get_or_404(question_id)
    if g.user != question.user:
        flash('수정권한이 없습니다')
        return redirect(url_for('question.detail', question_id=question_id))
    if request.method == 'POST':
        form = QuestionForm()
        if form.validate_on_submit():
            form.populate_obj(question)
            question.modify_date = datetime.now()  # 수정일시 저장
            db.session.commit()
            return redirect(url_for('question.detail', question_id=question_id))
    else:
        form = QuestionForm(obj=question)
    return render_template('question/question_form.html', form=form)
# ---------------------------------------------------------------------------------------- #
```

`url_for('question.modify', question_id=question.id)` URL에 매핑되는 라우트 함수 modify를 question_views.py 파일에 생성하였다.

질문의 수정은 로그인이 필요하므로 `@login_required` 어노테이션을 추가해 주었다. 

로그인한 사용자와 질문의 작성자가 동일하지 않은 경우에는 수정이 불가능하도록 flash 오류를 발생시킨다.

modify 함수가 GET으로 요청되는 경우는 "질문수정" 버튼을 클릭했을 경우이다. 

modify 함수가 GET으로 요청되면 질문을 수정할 수 있는 `question_form.html` 템플릿을 브라우저에 렌더링할 것이다. 

이때 이미 저장된 "제목", "내용" 등의 데이터를 보여주어야 하기 때문에 QuestionForm에 "제목"과 "내용"을 저장하여 리턴해 주어야 할 것이다. 

폼에 조회된 데이터의 값을 적용하는 가장 간단한 방법은 `QuestionForm(obj=question)` 처럼 조회된 객체를 전달하여 폼을 생성하는 방법이다. 

이렇게 하면 QuestionForm의 subject, content에 question 객체의 subject와 content값이 적용된다.

modify 함수가 POST로 요청되는 경우는 "질문수정" 화면에서 데이터를 수정하고 "저장하기" 버튼을 클릭한 경우일 것이다. 

modify 함수가 POST로 요청되면 `form.validate_on_submit` 를 사용하여 QuestionForm을 먼저 검증하고 이상이 없을 경우 변경된 데이터를 저장하게 된다. 

이때 변경을 위해서 `form.populate_obj(question)` 를 사용했는데 이것은 form으로 전달받은 데이터를 question 객체에 적용하는 역할을 한다. 

GET방식에서 조회된 객체를 폼에 적용하는 방법과 반대의 경우라고 생각하면 된다. 

그리고 앞서 추가한 modify_date 속성에 현재일시를 대입하여 수정일시를 생성해 주었다.


#### 테스트

이제 작성한 글을 수정하는 테스트를 해 보자. 

글 수정을 위해서는 먼저 로그인을 해야 한다. 

로그인한 사용자와 글쓴이가 동일인이라면 상세조회 화면에서 다음처럼 "수정" 버튼이 보일 것이다.

### 질문삭제

이제 질문을 삭제하는 기능을 구현해 보자. 다음처럼 상세조회 템플릿에 작성한 글을 삭제할 수 있는 버튼을 추가해 보자.

#### 질문삭제 버튼

`pybo\templates\question\question_detail.html`


```html
<...>
{% if g.user == question.user %}
<div class="my-3">
    <a href="{{ url_for('question.modify', question_id=question.id) }}"
       class="btn btn-sm btn-outline-secondary">수정</a>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <a href="#" class="delete btn btn-sm btn-outline-secondary"
       data-uri="{{ url_for('question.delete', question_id=question.id) }}">삭제</a>
    <!-- ---------------------------------------------------------------------------------------- -->
</div>
{% endif %}
<...>
```

#### jQuery

삭제 버튼은 조금 생각해야 할것이 많다. 

삭제 버튼을 클릭했을때 "정말로 삭제하시겠습니까?" 와 같은 확인창이 필요하기 때문이다. 

삭제 버튼을 클릭했을때 확인창을 호출하기 위해서는 다음과 같은 자바스크립트 코드가 필요하다.

```javascript
<script type='text/javascript'>
$(document).ready(function(){
    $(".delete").on('click', function() {
        if(confirm("정말로 삭제하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
});
</script>
```

이 자바스크립트는 jQuery로 작성되었다. 

jQuery가 생소하더라도 이 코드를 이해하는것은 어렵지 않다. 

이 자바스크립트의 의미는 "delete" 라는 클래스가 정의되어 있는 컴포넌트(예:버튼이나 링크)를 클릭하면 "정말로 삭제하시겠습니까?" 라는 질문을 하고 "확인"을 선택했을때 해당 컴포넌트에 속해있는 data-uri의 값으로 URL 호출을 하라는 의미이다. 

"확인" 대신 "취소"를 선택하면 아무런 일도 발생하지 않을 것이다.

#### 스크립트 블록

하지만 이 자바스크립트는 jQuery로 작성되었기 때문에 jQuery 라이브러리가 먼저 로드된 후 호출되어야 한다. 

jQuery 라이브러리는 부트스트랩의 기능을 사용하기 위해 이미 base.html에 포함되어 있다.

템플릿에서 jQuery를 사용하려면 다음처럼 base.html을 좀 수정해 주어야 한다.

`pybo\templates\base.html`

```html
<...>
<!-- jQuery JS -->
<script src="{{ url_for('static', filename='jquery-3.4.1.min.js') }}"></script>
<!-- Bootstrap JS -->
<script src="{{ url_for('static', filename='bootstrap.min.js') }}"></script>
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<!-- 자바스크립트 Start -->
{% block script %}
{% endblock %}
<!-- 자바스크립트 End -->
<!-- ---------------------------------------------------------------------------------------- -->
</body>
</html>
```

이제 question_detail.html 하단에 `{% block script %}{% endblock %}` 블록을 다음처럼 추가하자.

`pybo\templates\question\question_detail.html`

```html
<...>
{% endblock %}
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    $(".delete").on('click', function() {
        if(confirm("정말로 삭제하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
});
</script>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

#### 질문삭제 함수

그리고 data-uri에 url_for('question.delete', question_id=question.id) 라는 URL이 추가되었으므로 다음처럼 라우트 함수를 추가해야 한다.

`pybo\views\question_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/delete/<int:question_id>')
@login_required
def delete(question_id):
    question = Question.query.get_or_404(question_id)
    if g.user != question.user:
        flash('삭제권한이 없습니다')
        return redirect(url_for('question.detail', question_id=question_id))
    db.session.delete(question)
    db.session.commit()
    return redirect(url_for('question._list'))
# ---------------------------------------------------------------------------------------- #
```

delete 함수 역시 로그인이 필요하므로 `@login_required` 어노테이션을 적용하고 로그인한 사용자와 글쓴이가 동일한 경우에만 삭제가 가능하도록 했다.

#### 테스트

"삭제" 버튼을 클릭하여 삭제가 정상적으로 동작하는지 확인해 보도록 하자.

### 답변수정

이번에는 답변을 수정하고 삭제할 수 있도록 기능을 추가해 보자. 

질문을 수정하고 삭제하는 것과 동일한 방법을 적용하면 된다. 

다만 질문은 질문을 등록하는 템플릿(question_form.html)을 재활용했지만 답변을 수정할때는 답변등록을 하는 템플릿이 따로 없기 때문에 답변 수정을 위한 템플릿을 추가로 작성해야만 한다.


#### 답변수정 버튼

`pybo\templates\question\question_detail.html`

```html
<...>
{% for answer in question.answer_set %}
<div class="card my-3">
    <div class="card-body">
        <div class="card-text" style="white-space: pre-line;">{{ answer.content }}</div>
        <div class="d-flex justify-content-end">
            <div class="badge badge-light p-2 text-left">
                <div class="mb-2">{{ answer.user.username }}</div>
                <div>{{ answer.create_date }}</div>
            </div>
        </div>
        <!-- -------------------------------------- [edit] -------------------------------------- -->
        {% if g.user == answer.user %}
        <div class="my-3">
            <a href="{{ url_for('answer.modify', answer_id=answer.id) }}"
                class="btn btn-sm btn-outline-secondary">수정</a>
        </div>
        {% endif %}
        <!-- ------------------------------------------------------------------------------------ -->
    </div>
</div>
{% endfor %}
<...>
```

#### 답변수정 함수

`pybo\views\answer_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Blueprint, url_for, request, render_template, g, flash
# ---------------------------------------------------------------------------------------- #

...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/modify/<int:answer_id>', methods=('GET', 'POST'))
@login_required
def modify(answer_id):
    answer = Answer.query.get_or_404(answer_id)
    if g.user != answer.user:
        flash('수정권한이 없습니다')
        return redirect(url_for('question.detail', question_id=answer.question.id))
    if request.method == "POST":
        form = AnswerForm()
        if form.validate_on_submit():
            form.populate_obj(answer)
            answer.modify_date = datetime.now()  # 수정일시 저장
            db.session.commit()
            return redirect(url_for('question.detail', question_id=answer.question.id))
    else:
        form = AnswerForm(obj=answer)
    return render_template('answer/answer_form.html', answer=answer, form=form)
# ---------------------------------------------------------------------------------------- #
```


#### 답변수정 폼

`pybo\templates\answer\answer_form.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <form method="post" class="post-form">
        {{ form.csrf_token }}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="content">답변내용</label>
            <textarea class="form-control" name="content" id="content"
                      rows="10">{{ form.content.data or '' }}</textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

#### 테스트

답변도 질문과 마찬가지로 답변을 등록한 사용자와 로그인한 사용자가 동일할 경우에만 "수정" 버튼이 노출된다. 

답변을 작성하고 "수정" 버튼을 이용하여 답변 수정이 잘 동작하는지 확인해 보도록 하자.

### 답변삭제

이번에는 답변을 삭제하는 기능을 추가해 보자.

#### 답변삭제 버튼

`pybo\templates\question\question_detail.html`


```html
<...>
{% for answer in question.answer_set %}
<div class="card my-3">
    <div class="card-body">
        <...>
        {% if g.user == answer.user %}
        <div class="my-3">
            <a href="{{ url_for('answer.modify', answer_id=answer.id) }}"
                class="btn btn-sm btn-outline-secondary">수정</a>
            <!-- ------------------------------------ [edit] ------------------------------------ -->
            <a href="#" class="delete btn btn-sm btn-outline-secondary "
                data-uri="{{ url_for('answer.delete', answer_id=answer.id) }}">삭제</a>
            <!-- ---------------------------------------------------------------------------------- -->
        </div>
        {% endif %}
    </div>
</div>
{% endfor %}
<...>
```

#### 답변삭제함수

`pybo\views\answer_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/delete/<int:answer_id>')
@login_required
def delete(answer_id):
    answer = Answer.query.get_or_404(answer_id)
    question_id = answer.question.id
    if g.user != answer.user:
        flash('삭제권한이 없습니다')
    else:
        db.session.delete(answer)
        db.session.commit()
    return redirect(url_for('question.detail', question_id=question_id))
# ---------------------------------------------------------------------------------------- #
```

#### 테스트

답변을 작성한 사용자와 로그인한 사용자가 동일한 경우 "삭제" 버튼이 노출될 것이다. 

답변의 "삭제" 버튼을 클릭하여 삭제가 정상적으로 동작하는지 확인해 보도록 하자.

### 수정일시 표시

이번에는 상세조회 화면에서 수정일시를 확인할 수 있도록 템플릿을 수정해 보자. 

질문과 답변에는 이미 작성일시를 표시하고 있다. 

이 작성일시 바로 왼쪽에 수정일시를 추가해 보자.

`pybo\templates\question\question_detail.html`

```html
<...>
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
            <div class="d-flex justify-content-end">
                <!-- -------------------------------------- [edit] -------------------------------------- -->
                {% if question.modify_date %}
                <div class="badge badge-light p-2 text-left mx-3">
                    <div class="mb-2">modified at</div>
                    <div>{{ question.modify_date|datetime }}</div>
                </div>
                {% endif %}
                <!-- ------------------------------------------------------------------------------------ -->
                <div class="badge badge-light p-2 text-left">
                    <div class="mb-2">{{ question.user.username }}</div>
                    <div>{{ question.create_date }}</div>
                </div>
            </div>
            <...>
        </div>
    </div>
    <h5 class="border-bottom my-3 py-2">{{ question.answer_set|length }}개의 답변이 있습니다.</h5>
    {% for answer in question.answer_set %}
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ answer.content }}</div>
            <div class="d-flex justify-content-end">
                <!-- -------------------------------------- [edit] -------------------------------------- -->
                {% if answer.modify_date %}
                <div class="badge badge-light p-2 text-left mx-3">
                    <div class="mb-2">modified at</div>
                    <div>{{ answer.modify_date|datetime }}</div>
                </div>
                {% endif %}
                <!-- ------------------------------------------------------------------------------------ -->
                <...>
            </div>
            <...>
        </div>
    </div>
    {% endfor %}
<...>
```

질문이나 답변에 수정이 발생하면 이제 다음처럼 수정일시가 표시되는 것을 확인할 수 있을 것이다.