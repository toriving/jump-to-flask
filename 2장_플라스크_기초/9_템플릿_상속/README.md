# 9. 템플릿 상속

목록조회와 상세조회 템플릿은 표준 HTML 문서가 아님

### HTML 구조

표준 HTML 문서의 구조는 다음과 같아야함

```html
<!doctype html>
<html lang="ko">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="{{ url_for('static', filename='bootstrap.min.css') }}">
    <!-- pybo CSS -->
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <title>Hello, pybo!</title>
</head>
<body>

... 생략 ...

</body>
</html>
```

즉 html, head, body 태그가 있어야하고 스타일시트 파일은 head 태그 안에 존재햐아 한다.

head 태그 안에는 필요한 meta 태그와 title 태그가 포함되어야함.

### 템플릿 상속

위와 같은 표준 HTML 구조가 되도록 템플릿 수정

위와 같은 구조로 변경한다면 `<body>` 태그 바깥 부분은 모두 동일한 내용으로 중복 될것.

따라서 중복과 불편함을 없애기 위해 템플릿 상속(extend)  하도록 한다

#### base.html

다음처럼 기본 틀이 되는 `base.html` 템플릿을 작성해보자

`templates\base.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<!doctype html>
<html lang="ko">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="{{ url_for('static', filename='bootstrap.min.css') }}">
    <!-- pybo CSS -->
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <title>Hello, pybo!</title>
</head>
<body>
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
</body>
</html>
<!-- ---------------------------------------------------------------------------------------- -->
```

`base.html` 은 모든 템플릿이 상속해야하는 표준 HTML 기본 틀

`{% block content %}` 와 `{% endblock %}`는 `base.html`을 상속한 템플릿에서 구현하는 영역

#### question_list.html

`question_list`를 변경하자

`templates\question\question_list.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends 'base.html' %}
{% block content %}
<!-- ---------------------------------------------------------------------------------------- -->
<div class="container my-3">
    <table class="table">
        <thead>
        <tr class="thead-dark">
            <th>번호</th>
            <th>제목</th>
            <th>작성일시</th>
        </tr>
        </thead>
        <tbody>
        {% if question_list %}
        {% for question in question_list %}
        <tr>
            <td>{{ loop.index }}</td>
            <td>
                <a href="{{ url_for('question.detail', question_id=question.id) }}">{{ question.subject }}</a>
            </td>
            <td>{{ question.create_date }}</td>
        </tr>
        {% endfor %}
        {% else %}
        <tr>
            <td colspan="3">질문이 없습니다.</td>
        </tr>
        {% endif %}
        </tbody>
    </table>
</div>
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

`base.html` 을 상속하기 위해 `{% extends 'base.html' %}` 을 사용한다.

그리고 {% block content %} 와 {% endblock %} 사이에 question_list.html에서만 쓰이는 내용을 작성한다. 

이렇게 하면 이제 question_list.html은 base.html 템플릿을 상속받아 표준 HTML문서로 바뀌게 될 것이다.

#### question_detail.html

`question_detail.html` 도 마찬가지 방법으로 수정하자.

`templates\question\question_detail.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends 'base.html' %}
{% block content %}
<!-- ---------------------------------------------------------------------------------------- -->
<div class="container my-3">
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
            <div class="d-flex justify-content-end">
                <div class="badge badge-light p-2">
                    {{ question.create_date }}
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
                    {{ answer.create_date }}
                </div>
            </div>
        </div>
    </div>
    {% endfor %}
    <form action="{{ url_for('answer.create', question_id=question.id) }}" method="post" class="my-3">
        <div class="form-group">
            <textarea name="content" id="content" class="form-control" rows="10"></textarea>
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary">
    </form>
</div>
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

### style.css

부트스트랩을 사용하면서 `style.css` 의 내용은 필요없어졌으므로 모두 삭제하고 빈 파일로 남기자

`pybo\static\style.css`