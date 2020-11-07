# 2. 템플릿 필터

이번장에서는 템플릿 필터를 수동으로 작성하는 방법에 대해서 알아본다. 

템플릿 필터는 이미 사용했던 `{{ question.answer_set|length }}` 코드처럼 템플릿에서 사용할 수 있는 객체에 파이프라인(|) 문자를 사용하여 필터기능을 수행하는 것을 말한다.

날짜는 datatime 객체라서 문자열로 표기하면 현재와 같이 표시된다.

하지만 게시물을 표기하기에 적당하지 않다.

따라서 적당한 문자열로 나타내 보자

filter.py 을 생성하자

`\myproject\pybo\filter.py`

```python
def format_datetime(value, fmt='%Y년 %m월 %d일 %H:%M'):
    return value.strftime(fmt)
```

format_datetime 함수는 전달받은 datetime 객체(value 매개변수)를 날짜포맷형식(fmt 매개변수)에 맞게 변환하여 리턴하는 함수다

만약 fmt가 전달되지 않을 경우 디폴트 값인 '%Y년 %m월 %d일 %H:%M' 을 사용하여 변환한다.

그리고 `create_app` 함수를 수정하자

`myproject\pybo\__init__.py`

```python
...

def create_app():
    ...

    # 블루프린트
    from .views import main_views, question_views, answer_views
    app.register_blueprint(main_views.bp)
    app.register_blueprint(question_views.bp)
    app.register_blueprint(answer_views.bp)

    # ---------------------------------------- [edit] ---------------------------------------- #
    # 필터
    from .filter import format_datetime
    app.jinja_env.filters['datetime'] = format_datetime
    # ---------------------------------------------------------------------------------------- #

    return app
```

format_datetime 함수를 import하여 `app.jinja_env.filters['datetime']` 처럼 `datetime` 이라는 이름의 필터로 등록해 주었다.

목록조회 화면에 datatime 필터를 적용하자

`pybo\templates\question\question_list.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <table class="table">
        <...>
        <tr>
            <td>{{ loop.index }}</td>
            <td>
                <a href="{{ url_for('question.detail', question_id=question.id) }}">{{ question.subject }}</a>
            </td>
            <!-- ---------------------------------------- [edit] ---------------------------------------- -->
            <td>{{ question.create_date|datetime }}</td>
            <!-- ---------------------------------------------------------------------------------------- -->
        </tr>
        <...>
    </table>
    <...>
</div>
{% endblock %}
```

그리고 상세조회 화면도 필터를 적용하자

`pybo\templates\question\question_detail.html`

```html
<...>
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
            <div class="d-flex justify-content-end">
                <div class="badge badge-light p-2">
                    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
                    {{ question.create_date|datetime }}
                    <!-- ---------------------------------------------------------------------------------------- -->
                </div>
            </div>
        </div>
    </div>
    <h5 class="border-bottom my-3 py-2">{{ question.answer_set|length }}개의 답변이 있습니다.</h5>
    {% for answer in question.answer_set %}
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ answer.content }}</div>
            <div class="d-flex justify-content-end">
                <div class="badge badge-light p-2">
                    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
                    {{ answer.create_date|datetime }}
                    <!-- ---------------------------------------------------------------------------------------- -->
                </div>
            </div>
        </div>
    </div>
    {% endfor %}
    <...>
</div>
{% endblock %}
```

이제 확인해보자

빌트인 필터는 아래에서 확인 할 수 있다.

https://jinja.palletsprojects.com/en/2.11.x/templates/#builtin-filters