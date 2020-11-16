# 15. 검색과 정렬

이번 장에서는 파이보에 검색과 정렬 기능을 적용해 보자.

### 조인과 서브쿼리

데이터베이스의 선행지식을 알아보자

#### 조인 (join)

조인은 동일한 데이터로 연결된 두 모델을 함께 조회하기 위해 사용된다.

예를들어 작성자 이름이 "홍길동"인 질문을 검색할 수 있는 방법에 대해서 살펴보자. 

아마 다음과 같은 절차가 필요할 것이다.

1. User 모델에서 username이 "홍길동"인 id번호를 조사
2. 1번에서 조사한 id번호와 Question 모델의 user_id 가 같은 건들을 조사

이렇게 단계별로 나누어서 찾을수도 있겠지만 "조인" 기능을 사용하면 다음처럼 보다 간편하게 검색할 수 있다.

`Question.query.join(User).filter(User.username=='홍길동')`

Question 모델과 User 모델을 join 함수를 사용하여 "조인"을 하면 User.username 처럼 User모델의 username속성을 필터에서 사용할 수 있게 된다.

#### 아우터조인 (outer join)

아우터조인은 조금 복잡하다.

참조

https://wikidocs.net/81067 

```
>>> Question.query.outerjoin(Answer).filter(
...     Question.content.ilike('%파이썬%') |
...     Answer.content.ilike('%파이썬%')).distinct()
```

#### 서브쿼리 (subquery)

Question 모델과 연결된 다른 모델을 검색하기 위해서는 Answer나 User와 같은 모델을 사용하여 조인하는 것이 일반적인 방법이다. 

하지만 이렇게 모델 그 자체를 사용하기 보다는 필요한 데이터를 조회하는 서브쿼리를 만들어 조인하는 방법도 있다.

우리는 파이보의 검색기능을 구현하기 위해 답변의 내용 뿐만 아니라 답변의 작성자도 검색조건에 포함해야 한다. 

답변의 내용으로 검색하는 것은 아우터조인에서 알아본 것과 같이 Answer 모델을 아우터조인하면 쉽게 해결할 수 있다. 

하지만 답변의 작성자를 조회조건에 추가하는 것은 쉽지 않다. 

왜냐하면 답변의 작성자는 이미 아우터조인한 Answer 모델과 User 모델을 다시한번 더 조인해야 하기 때문이다.

이런 복잡한 경우는 다음처럼 서브쿼리를 사용하는 것이 유리하다.

`sub_query = db.session.query(Answer.question_id, Answer.content, User.username)\
            .join(User, Answer.user_id == User.id).subquery()`

서브쿼리는 `db.session.query` 로 만들어진 쿼리에 `subquery()` 함수를 수행하여 만들 수 있다. 

위에서 만들어진 서브쿼리는 검색 조건에 사용될 답변의 내용(Answer.content)과 답변의 작성자(User.username)가 쿼리의 조회 항목으로 추가되었다. 

그리고 이 서브쿼리와 질문 모델을 연결할 수 있는 질문의 id번호(Answer.question_id)도 조회 항목에 추가되었다. 

그리고 답변 모델의 작성자명을 알 수 있도록 답변 모델과 User모델을 조인해 주었다.

위와 같이 서브쿼리를 생성하면 이제 다음처럼 질문 모델과 아우터조인하여 사용할 수 있다.

`Question.query.outerjoin(sub_query.c.question_id == Question.id).distinct()`

`sub_query.c.question_id` 에서 사용된 `c` 는 서브쿼리의 조회 항목을 의미한다. 

즉, `sub_query.c.question_id` 는 서브쿼리의 조회항목 중 question_id 항목을 의미한다.

이제 sub_query를 아우터조인 했으므로 다음처럼 sub_query의 조회 항목을 filter 조건에 추가할 수 있다.

`Question.query.outerjoin(sub_query.c.question_id == Question.id) \
    .filter(sub_query.c.content.ilike('%파이썬%') |   # 답변내용
           sub_query.c.username.ilike('%파이썬%')    # 답변작성자
           ).distinct()`
           
           
이렇게 하면 이제 질문의 답변내용 또는 질문의 작성자명 중에서 "파이썬"이라는 문자열을 포함하는 질문들만 조회할 수 있게 될 것이다.

### 검색기능

이제 위에서 살펴본 조인, 아우터조인, 서브쿼리등을 모두 조합하여 검색기능을 다음처럼 완성해 보자. 

실제 코드는 조금 후에 작성해 보기로 하자. 

아래코드는 우선 눈으로만 살펴보자.

```python
kw = request.args.get('kw', type=str, default='')  # 검색어

# 검색
search = '%%{}%%'.format(kw)
sub_query = db.session.query(Answer.question_id, Answer.content, User.username)\
    .join(User, Answer.user_id == User.id).subquery()
question_list = Question.query\
    .join(User)\
    .outerjoin(sub_query, sub_query.c.question_id == Question.id)\
    .filter(Question.subject.ilike(search) |      # 질문제목
            Question.content.ilike(search) |      # 질문내용
            User.username.ilike(search) |         # 질문작성자
            sub_query.c.content.ilike(search) |   # 답변내용
            sub_query.c.username.ilike(search)    # 답변작성자
            )\
    .distinct()
```

kw는 화면으로부터 전달받은 검색어이다. 검색어는 양쪽에 `%` 기호를 삽입하여 ilike 함수의 인수로 전달되도록 하였다.

Question 모델을 기준으로 조회하기 때문에 질문제목과 질문내용은 별도의 조인없이 검색 조건으로 사용할 수 있다. 질문의 작성자는 User 모델과 조인하여 검색 조건으로 사용할 수 있다. User 모델은 Question 모델과 1대 1의 관계이므로 아우터조인을 사용할 필요가 없다. 그리고 답변내용과 답변작성자로 검색하기 위해서 서브쿼리를 작성한후 Question 모델과 아우터조인을 했다.

그리고 각각의 검색 조건은 OR조건으로 조회하기 위해서 filter 함수 내에서 `|` 기호를 사용하였다.

#### GET

위에서 살펴본 코드에서 kw는 화면으로 부터 전달받은 검색어에 해당된다. 

kw는 keywords의 약어로 검색어를 의미한다. GET방식은 URL에 노출되고 길이 제한이 있기때문에 보통 약어를 많이 사용한다.

kw 값은 다음처럼 POST가 아닌 GET으로 읽도록 했다.

`kw = request.args.get('kw', type=str, default='')  # 검색어`

즉, kw는 다음처럼 GET방식으로 전달되어야 index함수에서 읽을 수 있다.

* http://localhost:5000/?kw=파이썬&page=1

kw를 GET이 아닌 POST로 전달하는 방법은 추천하고 싶지 않다. 

kw를 POST방식으로 전달한다면 page파라미터도 역시 POST방식으로 전달해야 한다. (※ kw는 POST로 전달하고 page는 GET으로 전달하는 방법은 존재하지 않는다.)

이렇게 POST방식으로 검색과 페이징이 적용되면 웹브라우저에서 "새로고침" 또는 "뒤로가기" 버튼을 클릭할 때 "만료된 페이지입니다."라는 오류를 만나게 된다. 

왜냐하면 POST방식은 동일한 POST요청이 발생할 경우 중복요청을 방지하기 위해 "만료된 페이지입니다." 오류를 발생시키기 때문이다. 

2페이지에서 3페이지로 갔다가 웹브라우저 뒤로가기 버튼을 클릭했을 때 2페이지로 가는것이 아니라 오류가 발생된다면 엉망이 될 것이다.

이러한 이유로 게시판을 조회하는 목록함수를 호출할 때는 GET방식을 사용해야 한다. 앞으로 추가될 정렬기능에 대한 파라미터 so도 GET방식으로 요청될 것이다.

이제 화면에서 kw와 page를 동시에 GET방식으로 호출하는 방법에 대해서 알아보자.

#### 검색창

먼저 question_list.html 템플릿에 검색어를 입력할 수 있는 텍스트창을 다음과 같이 추가해 주자.

`pybo\templates\question\question_list.html`

```html
{% extends 'pybo/base.html' %}
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <div class="row justify-content-end my-3">
        <div class="col-4 input-group">
            <input type="text" class="form-control kw" value="{{ kw or '' }}">
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button" id="btn_search">찾기</button>
            </div>
        </div>
    </div>
    <!-- ---------------------------------------------------------------------------------------- -->
    <table class="table">
    <...>
```

#### 검색폼

그리고 page와 kw를 동시에 GET으로 요청할 수 있는 searchForm을 다음과 같이 추가해 준다.

`pybo\templates\question\question_list.html`

```html
<...>
    <!-- 페이징처리 끝 -->
    <a href="{% url 'pybo:question_create' %}" class="btn btn-primary">질문 등록하기</a>
</div>
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<form id="searchForm" method="get" action="{{ url_for('question._list') }}">
    <input type="hidden" id="kw" name="kw" value="{{ kw or '' }}">
    <input type="hidden" id="page" name="page" value="{{ page }}">
</form>
<!-- ---------------------------------------------------------------------------------------- -->
{% endblock %}
```

#### 페이징

그리고 기존 페이징 처리하는 부분도 ?page=1 처럼 직접 파라미터를 코딩하는 방식에서 값을 읽을 수 있도록 다음처럼 변경해 주어야 한다.

`pybo\templates\question\question_list.html`

```html
<...>
<!-- 페이징처리 시작 -->
<ul class="pagination justify-content-center">
    <!-- 이전페이지 -->
    {% if question_list.has_prev %}
    <li class="page-item">
        <!-- -------------------------------------- [edit] -------------------------------------- -->
        <a class="page-link" data-page="{{ question_list.prev_num }}" href="#">이전</a>
        <!-- ------------------------------------------------------------------------------------ -->
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
                <!-- -------------------------------------- [edit] -------------------------------------- -->
                <a class="page-link" data-page="{{ page_num }}" href="#">{{ page_num }}</a>
                <!-- ------------------------------------------------------------------------------------ -->
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
        <!-- -------------------------------------- [edit] -------------------------------------- -->
        <a class="page-link" data-page="{{ question_list.next_num }}" href="#">다음</a>
        <!-- ------------------------------------------------------------------------------------ -->
    </li>
    {% else %}
    <li class="page-item disabled">
        <a class="page-link" tabindex="-1" aria-disabled="true" href="#">다음</a>
    </li>
    {% endif %}
</ul>
<!-- 페이징처리 끝 -->
<...>
```

모든 페이지 링크를 href 속성에 직접 링크를 거는 방식 대신에 data-page 속성을 추가하여 값을 읽을수 있도록 변경하였다.

#### 스크립트

그리고 페이징과 검색을 처리할 수 있는 자바스크립트 블록을 다음과 같이 추가하자.

`pybo\templates\question\question_list.html`

```html
<...>
{% endblock %}
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    $(".page-link").on('click', function() {
        $("#page").val($(this).data("page"));
        $("#searchForm").submit();
    });
    $("#btn_search").on('click', function() {
        $("#kw").val($(".kw").val());
        $("#page").val(1);  // 검색버튼을 클릭할 경우 1페이지부터 조회한다.
        $("#searchForm").submit();
    });
});
</script>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

다음과 같이 class 속성값으로 "page-link"라는 값을 가지고 있는 링크를 클릭하면

`<a class="page-link" data-page="{{ question_list.prev_num }}" href="#">이전</a>`
이 링크의 data-page 속성값을 읽어 폼 page 필드에 설정하여 폼을 요청하도록 다음과 같은 자바스크립트를 추가해 주었다.

```
$(".page-link").on('click', function() {
    $("#page").val($(this).data("page"));
    $("#searchForm").submit();
});
```

그리고 검색버튼을 클릭하면 검색어 텍스트창에 입력된 값을 폼 kw필드에 설정하여 폼(searchForm)을 요청하도록 다음과 같은 자바스크립트를 추가해 주었다.

```
$("#btn_search").on('click', function() {
    $("#kw").val($(".kw").val());
    $("#page").val(1);  // 검색버튼을 클릭할 경우 1페이지부터 조회한다.
    $("#searchForm").submit();
});
```

검색버튼을 클릭하는 경우는 새로운 검색에 해당되므로 page 필드에 항상 1을 설정하여 폼(searchForm)을 요청하도록 했다.


#### 검색함수

이제 검색어가 조회에 적용될 수 있도록 question_views.py의 `_list` 함수를 다음처럼 수정하자.

`pybo\views\question_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from ..models import Question, Answer, User
# ---------------------------------------------------------------------------------------- #    
...

# ---------------------------------------- [edit] ---------------------------------------- #
@bp.route('/list/')
def _list():
    # 입력 파라미터
    page = request.args.get('page', type=int, default=1)
    kw = request.args.get('kw', type=str, default='')

    # 조회
    question_list = Question.query.order_by(Question.create_date.desc())
    if kw:
        search = '%%{}%%'.format(kw)
        sub_query = db.session.query(Answer.question_id, Answer.content, User.username) \
            .join(User, Answer.user_id == User.id).subquery()
        question_list = question_list \
            .join(User) \
            .outerjoin(sub_query, sub_query.c.question_id == Question.id) \
            .filter(Question.subject.ilike(search) |  # 질문제목
                    Question.content.ilike(search) |  # 질문내용
                    User.username.ilike(search) |  # 질문작성자
                    sub_query.c.content.ilike(search) |  # 답변내용
                    sub_query.c.username.ilike(search)  # 답변작성자
                    ) \
            .distinct()

    # 페이징
    question_list = question_list.paginate(page, per_page=10)
    return render_template('question/question_list.html', question_list=question_list, page=page, kw=kw)
# ---------------------------------------------------------------------------------------- #    
...
```


#### 테스트

`마크다운`으로 테스트해보자


### 정렬기능

이번에는 질문목록을 정렬할 수 있는 기능을 추가해 보자. 생각할 수 있는 정렬 기준은 다음과 같다.

- 최신순 - 최근 등록된 질문을 먼저 보여주는 방식
- 추천순 - 추천을 많이 받은 질문을 먼저 보여주는 방식
- 인기순 - 질문에 등록된 답변이 많은것을 먼저 보여주는 방식

이와 같은 정렬기준에 해당되는 파라미터 역시 page와 kw 파라미터와 함께 GET방식으로 요청되어야 한다. 그래야만 페이징, 검색, 정렬기능이 함께 동작할 수 있다.

#### 정렬조건

먼저 화면에 정렬조건을 다음처럼 추가하자.

`pybo\templates\question\question_list.html`

```html
{% extends 'pybo/base.html' %}
{% load pybo_filter %}
{% block content %}
<div class="container my-3">
    <div class="row justify-content-between my-3">  <!-- 양쪽정렬 justify-content-between로 변경 -->
        <!-- -------------------------------------- [edit] -------------------------------------- -->
        <div class="col-2">
            <select class="form-control so">
                <option value="recent" {% if so=='recent' %}selected{% endif %}>최신순</option>
                <option value="recommend" {% if so=='recommend' %}selected{% endif %}>추천순</option>
                <option value="popular" {% if so=='popular' %}selected{% endif %}>인기순</option>
            </select>
        </div>
        <!-- ------------------------------------------------------------------------------------ -->
        <div class="col-4 input-group">
            <input type="text" class="form-control kw" value="{{ kw|default_if_none:"" }}">
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button" id="btn_search">찾기</button>
            </div>
        </div>
    </div>
    <table class="table">
    <...>
```

#### 정렬폼

그리고 searchForm에 정렬기준 파라미터에 해당되는 so 필드를 다음과 같이 추가하자.

`pybo\templates\question\question_list.html`

```html
<...>
<form id="searchForm" method="get" action="{{ url_for('question._list') }}">
    <input type="hidden" id="kw" name="kw" value="{{ kw or '' }}">
    <input type="hidden" id="page" name="page" value="{{ page }}">
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <input type="hidden" id="so" name="so" value="{{ so }}">
    <!-- ---------------------------------------------------------------------------------------- -->
</form>
<...>
```

#### 자바스크립트

그리고 정렬기준 콤보박스를 변경할때 searchForm 요청이 발생할 수 있도록 다음과 같은 jQuery 자바스크립트를 추가하자.

`pybo\templates\question\question_list.html`

#### 정렬함수

이제 입력으로 받은 so 파라미터를 이용하여 질문목록을 정렬할 수 있도록 question_views.py의 `_list`함수를 다음처럼 수정하자.

`pybo\views\question_views.py`

```python
...
# ---------------------------------------- [edit] ---------------------------------------- #
from sqlalchemy import func
# ---------------------------------------------------------------------------------------- #
...
# ---------------------------------------- [edit] ---------------------------------------- #
from ..models import Question, Answer, User, question_voter
# ---------------------------------------------------------------------------------------- #
...

@bp.route('/list/')
def _list():
    # 입력 파라미터
    page = request.args.get('page', type=int, default=1)
    kw = request.args.get('kw', type=str, default='')
    # ---------------------------------------- [edit] ---------------------------------------- #
    so = request.args.get('so', type=str, default='recent')

    # 정렬
    if so == 'recommend':
        sub_query = db.session.query(question_voter.c.question_id, func.count('*').label('num_voter')) \
            .group_by(question_voter.c.question_id).subquery()
        question_list = Question.query \
            .outerjoin(sub_query, Question.id == sub_query.c.question_id) \
            .order_by(sub_query.c.num_voter.desc(), Question.create_date.desc())
    elif so == 'popular':
        sub_query = db.session.query(Answer.question_id, func.count('*').label('num_answer')) \
            .group_by(Answer.question_id).subquery()
        question_list = Question.query \
            .outerjoin(sub_query, Question.id == sub_query.c.question_id) \
            .order_by(sub_query.c.num_answer.desc(), Question.create_date.desc())
    else:  # recent
        question_list = Question.query.order_by(Question.create_date.desc())

    # 검색
    # ---------------------------------------------------------------------------------------- #
    if kw:
        search = '%%{}%%'.format(kw)
        sub_query = db.session.query(Answer.question_id, Answer.content, User.username) \
            .join(User, Answer.user_id == User.id).subquery()
        question_list = question_list \
            .join(User) \
            .outerjoin(sub_query, sub_query.c.question_id == Question.id) \
            .filter(Question.subject.ilike(search) |  # 질문제목
                    Question.content.ilike(search) |  # 질문내용
                    User.username.ilike(search) |  # 질문작성자
                    sub_query.c.content.ilike(search) |  # 답변내용
                    sub_query.c.username.ilike(search)  # 답변작성자
                    ) \
            .distinct()

    # 페이징
    question_list = question_list.paginate(page, per_page=10)
    # ---------------------------------------- [edit] ---------------------------------------- #
    return render_template('question/question_list.html', question_list=question_list, page=page, kw=kw)
    # ---------------------------------------------------------------------------------------- #
...    
```

설명은 https://wikidocs.net/81067 을 확인하자

#### 테스트

인기순 정렬을 해보자


