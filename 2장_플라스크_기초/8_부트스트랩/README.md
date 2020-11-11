# 8. 부트스트랩

부트스트랩은 웹디자인을 도와주는 프레임워크이다.

부트스트랩을 사용해보자.

### 부트스트랩 설치

부트스트랩 다운 -https://getbootstrap.com/

Compiled CSS and JS 을 다운받았다.

압축파일에서 bootstrap.min.css 파일을 카피하여 스태틱 디렉터리에 저장.


### 부트스트랩 적용

목록조회 템플릿에 적용해보자

`pybo\templates\question\question_list.html`

```html
<link rel="stylesheet" href="{{ url_for('static', filename='bootstrap.min.css') }}">
```

템플릿 상단에 bootstrap.min.css 스타일시트를 링크하기만 하면 된다.

부트스트랩을 사용할 수 있게 되었으므로 템플릿을 수정해보자.

`pybo\templates\question\question_list.html`

```html
<link rel="stylesheet" href="{{ url_for('static', filename='bootstrap.min.css') }}">
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
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
<!-- ---------------------------------------------------------------------------------------- -->
```

부트스트랩의 자세한 내용은 https://getbootstrap.com/docs/4.4/getting-started/introduction/ 를 참조하자

상세조회 템플릿도 부트스트랩을 적용하여 다음과 같이 수정해 보자.

`templates\pybo\question_detail.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<link rel="stylesheet" href="{{ url_for('static', filename='bootstrap.min.css') }}">
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
<!-- ---------------------------------------------------------------------------------------- -->
```