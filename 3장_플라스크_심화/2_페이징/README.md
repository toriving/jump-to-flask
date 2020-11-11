# 2. 페이징

파이보의 목록조회는 현재 페이징 처리가 안되어서 게시물 300개를 작성하면 목록조회 화면에 300개가 다 표시됨

이에 따라 페이징 기능을 적용하자

### 데이터 생성

페이징을 위한 데이터 생성하기 위해 플라스크 쉘을 이용하자

```shell script
$ flask shell
```

```python
>>> from pybo import db
>>> from pybo.models import Question
>>> from datetime import datetime

>>> for i in range(300):
...     q = Question(subject='테스트 데이터입니다:[%03d]' % i, content='내용무', create_date=datetime.now())
...     db.session.add(q)
...
>>>  db.session.commit()
>>>
```

300건 데이터를 만들자

db.session.commit() 은 마지막에 한번만 하면 된다.

### Paginator

페이징을 위해 질문목록을 조회하는 `_list` 함수에 다음처럼 페이징 기능을 적용해보자

`pybo\views\question_views.py`

```python
...
@bp.route('/list/')
def _list():
    # ---------------------------------------- [edit] ---------------------------------------- #
    page = request.args.get('page', type=int, default=1)  # 페이지
    # ---------------------------------------------------------------------------------------- #
    question_list = Question.query.order_by(Question.create_date.desc())
    # ---------------------------------------- [edit] ---------------------------------------- #
    question_list = question_list.paginate(page, per_page=10)
    # ---------------------------------------------------------------------------------------- #
    return render_template('question/question_list.html', question_list=question_list)

```

`_list` 함수에 추가된 내용를 가볍게 살펴보자.

`page = request.args.get('page', type=int, default=1)` 코드는 다음처럼 GET방식으로 요청한 URL에서 page의 값을 가져올때 사용한다.

`http://localhost:5000/question/list/?page=1`

만약 `http://localhost:5000/question/list/` 처럼 page값이 없을 경우에는 `default=1` 속성에 의하여 디폴트로 1이라는 값을 설정한다. 

`type=int` 속성의 의미는 page 파라미터의 데이터 타입이 정수(Integer)임을 말한다.

`question_list = question_list.paginate(page, per_page=10)` 코드는 조회된 데이터에 페이징을 적용해 준다. 

입력 파라미터 page는 현재 조회할 페이지번호를 의미하고 `per_page=10 `속성은 한 페이지에 보여줄 게시물의 갯수를 10건으로 하겠다는 의미이다.

이렇게 paginate 함수로 조회된 데이터를 한번 더 감싸주면 Pagination 객체가 리턴되어 다음과 같은 속성들을 사용할 수 있게 된다.

|항목|설명|
|------|---|
|items|현재 페이지에 해당되는 게시물 리스트|
|total|게시물의 전체건수|
|per_page|페이지당 보여줄 건수|
|page|현재페이지 번호|
|iter_pages|페이지의 범위|
|prev_num|이전페이지 번호|
|next_num|다음페이지 번호|
|has_prev|이전페이지 존재여부|
|has_next|다음페이지 존재여부|


### 템플릿에 페이징 적용하기

이제 Pagination 객체의 속성들을 사용하여 템플릿에서 어떻게 페이징을 처리하는지 알아보자.

먼저 템플릿에서 사용하는 question_list가 Pagination 객체로 변경되었으므로 목록을 출력하는 부분을 다음과 같이 수정해야 한다.

`pybo\templates\question\question_list.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <table class="table">
        <...>
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        {% for question in question_list.items %}
        <!-- ---------------------------------------------------------------------------------------- -->
        <...>
    </table>
    <a href="{{ url_for('question.create') }}" class="btn btn-primary">질문 등록하기</a>
</div>
{% endblock %}
```

원래 코드인 `{% for question in question_list %}`를 `{% for question in question_list.items %}` 처럼 .items 를 추가하여 수정해 주었다. 

Pagination객체는 현재 조회된 목록데이터를 가져오기 위해서는 items 함수를 호출해야 한다.

이렇게 수정하면 이제 300건의 대량 데이터가 모두 표시되지 않고 한페이지에 10건만 출력되는 것을 확인할 수 있다.

이제 페이지 이동을 위해 페이징 리스트를 템플릿에 구현해 보자. question_list.html 템플릿의 테이블을 닫는 태그(`</table>`) 바로밑에 다음을 추가하자.

`pybo\templates\question\question_list.html`

```html
<...>
    </table>
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <!-- 페이징처리 시작 -->
    <ul class="pagination justify-content-center">
        <!-- 이전페이지 -->
        {% if question_list.has_prev %}
        <li class="page-item">
            <a class="page-link" href="?page={{ question_list.prev_num }}">이전</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">이전</a>
        </li>
        {% endif %}
        {% for page_num in question_list.iter_pages() %}
            {% if page_num %}
                {% if page_num != question_list.page %}
                <li class="page-item">
                    <a class="page-link" href="?page={{ page_num }}">{{ page_num }}</a>
                </li>
                {% else %}
                <li class="page-item active" aria-current="page">
                    <a class="page-link" href="#">{{ page_num }}</a>
                </li>
                {% endif %}
           {% else %}
                <li class="disabled">
                    <a class="page-link" href="#">...</a>
                </li>
           {% endif %}
        {% endfor %}
        <!-- 다음페이지 -->
        {% if question_list.has_next %}
        <li class="page-item">
            <a class="page-link" href="?page={{ question_list.next_num }}">다음</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">다음</a>
        </li>
        {% endif %}
    </ul>
    <!-- 페이징처리 끝 -->
    <!-- ---------------------------------------------------------------------------------------- -->
    <a href="{% url 'pybo:question_create' %}" class="btn btn-primary">질문 등록하기</a>
</div>
{% endblock %}

```

페이징을 보기 좋게 표시하기 위해 부트스트랩의 pagination 컴포넌트를 이용하였다.

부트스트랩 pagination - https://getbootstrap.com/docs/4.4/components/pagination/

