# 9. 글쓴이 표시

목록조회, 상세조회 화면에도 글쓴이를 적용해보자

### 목록조회

`pybo\templates\question\question_list.html`

```html
<...>
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<tr class="text-center thead-dark">
<!-- ---------------------------------------------------------------------------------------- -->
    <th>번호</th>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <th style="width:50%">제목</th>
    <th>글쓴이</th>
    <!-- ---------------------------------------------------------------------------------------- -->
    <th>작성일시</th>
</tr>
<...>
```

그리고 for문으로 반복되는 부분에도 다음처럼 글쓴이를 적용하자.

`pybo\templates\question\question_list.html`

```html
<...>
{% for question in question_list.items %}
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<tr class="text-center">
<!-- ---------------------------------------------------------------------------------------- -->
    <td>{{ question_list.total - ((question_list.page-1) * question_list.per_page) - loop.index0 }}</td>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <td class="text-left">
    <!-- ---------------------------------------------------------------------------------------- -->
        <a href="{{ url_for('question.detail', question_id=question.id) }}">{{ question.subject }}</a>
        {% if question.answer_set|length > 0 %}
        <span class="text-danger small ml-2">{{ question.answer_set|length }}</span>
        {% endif %}
    </td>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <td>{{ question.user.username }}</td>  <!-- 글쓴이 추가 -->
    <!-- ---------------------------------------------------------------------------------------- -->
    <td>{{ question.create_date|datetime }}</td>
</tr>
{% endfor %}
<...>
```

### 상세조회

그리고 상세조회 화면도 다음처럼 글쓴이를 추가하자.

`pybo\templates\question\question_detail.html`

```html
<...>
<div class="card-body">
    <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
    <div class="d-flex justify-content-end">
        <!-- ------------------------------------- [edit] ------------------------------------- -->
        <div class="badge badge-light p-2 text-left">
            <div class="mb-2">{{ question.user.username }}</div>
            <div>{{ question.create_date }}</div>
        </div>
        <!-- ---------------------------------------------------------------------------------- -->
    </div>
</div>
<...>
```

답변에도 추가하자
`pybo\templates\question\question_detail.html`

```html
<...>
<div class="card-body">
    <div class="card-text" style="white-space: pre-line;">{{ answer.content }}</div>
    <div class="d-flex justify-content-end">
        <!-- ------------------------------------- [edit] ------------------------------------- -->
        <div class="badge badge-light p-2 text-left">
            <div class="mb-2">{{ answer.user.username }}</div>
            <div>{{ answer.create_date }}</div>
        </div>
        <!-- ---------------------------------------------------------------------------------- -->
    </div>
</div>
<...>
```