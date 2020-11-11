# 5. 답변갯수 표시하기

이번 장에서는 질문목로에 답변갯수를 표시하는 기능을 추가한다.

`\pybo\templates\question\question_list.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <table class="table">
        <...>
        <tr>
            <td>{{ question_list.total - ((question_list.page-1) * question_list.per_page) - loop.index0 }}</td>
            <td>
                <a href="{{ url_for('question.detail', question_id=question.id) }}">{{ question.subject }}</a>
                <!-- ---------------------------------------- [edit] ---------------------------------------- -->
                {% if question.answer_set|length > 0 %}
                <span class="text-danger small ml-2">{{ question.answer_set|length }}</span>
                {% endif %}
                <!-- ---------------------------------------------------------------------------------------- -->
            </td>
            <td>{{ question.create_date|datetime }}</td>
        </tr>
        <...>
    </table>
    <...>
</div>
{% endblock %}
```

제목부분 바로 옆에 답변갯수를 표시한다

{% if question.answer_set|length > 0 %} 코드를 사용하여 답변이 있을 경우에만 답변의 갯수({{ question.answer_set|length }})를 표시하도록 하였다.

