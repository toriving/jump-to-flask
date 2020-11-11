# 4. 게시물 번호 

현재 파이보 목록조회 화면에서는 모든 페이지에서 게시물 번호가 1에서부터 시작한다.

이 오류를 해결해보고자 한다.

게시물 번호는 역순으로 보이도록 한다.

`번호 = 전체건수 - (현재페이지-1) * 펭지ㅣ당건수 - 나열인덱스`

이 공식을 목록조회 템플릿에 적용해보자

`pybo\templates\question\question_list.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <table class="table">
        <...>
        <tr>
            <!-- ---------------------------------------- [edit] ---------------------------------------- -->
            <td>{{ question_list.total - ((question_list.page-1) * question_list.per_page) - loop.index0 }}</td>
            <!-- ---------------------------------------------------------------------------------------- -->
            <td>
                <a href="{{ url_for('question.detail', question_id=question.id) }}">{{ question.subject }}</a>
            </td>
            <td>{{ question.create_date|datetime }}</td>
        </tr>
        <...>
    </table>
    <...>
</div>
{% endblock %}
```

각 항목은 다음과 같다.

|항목|설명|
|---|---|
|question_list.total|전체건수|
|question_list.page|현재페이지|
|question_list.per_page|페이지당 건수|
|loop.index0|나열인덱스 (0부터 시작하는 0-based index)|

적용하면 잘 된다.