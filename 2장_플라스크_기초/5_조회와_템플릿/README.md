# 5. 조회와 템플릿

이번장에서는 파이보의 핵심기능이 될 목록조회와 상세조회를 구현해보자.

### 목록조회

현재 `http://localhost:5000` 에 접근하면 "Pybo Index" 라는 문구가 출력된다.

질문 목록이 출력되도록 `main_views.py` 파일의 index 함수를 다음과 같이 변경하자.

```python
from flask import Blueprint, render_template

from pybo.models import Question

...

@bp.route('/')
def index():
    question_list = Question.query.order_by(Question.create_date.desc())
    return render_template('question/question_list.html', question_list=question_list)
```

질문목록 데이터는 question_list = Question.query.order_by(Question.create_date.desc()) 로 얻을 수 있다. 

order_by는 조회 결과를 정렬하는 함수이다. 

order_by(Question.create_date.desc()) 의 의미는 조회된 데이터를 작성일시 기준으로 역순으로 정렬하라는 의미이다.

**※ 역순이 아닌 작성일시 순으로 조회하기 위해서는 order_by(Question.create_date.asc()) 또는 asc() 를 생략하여 order_by(Question.create_date)와 같이 사용하면 된다.**

render_template 함수는 템플릿 파일을 화면으로 렌더링 하는 함수이다.

이 때 조회된 질문목록 데이터를 템플릿으로 전달하여 템플릿에서 해당 데이터로 화면을 구성할 수 있게 된다. 

여기서 사용된 question/question_list.html 파일을 템플릿 파일이라고 부른다. 

템플릿 파일은 HTML파일과 비슷하지만 플라스크에서 사용하는 특별한 태그들을 사용할수 있는 HTML파일이다.

이 탬플릿 파일에 대해서는 잠시후 알아보자.

#### 템플릿 디렉터리

`render_template` 함수에서 사용할 `question/question_list.html` 템플릿 파일을 만들어야 한다.

템플릿을 저장할 디렉터리는 플라스크가 앱으로 지정한 모듈 하위에 `templates` 라는 디렉터리명으로 생성해준다.

즉 pybo 디렉터리 하위에 `templates` 라는 디렉터리를 생성해주자 (디폴트 템플릿 디렉터리)

#### 템플릿 파일

`render_template` 함수에서 템플릿 파일명은 `question/question_list.html` 이다.

따라서 `question_list.html` 을 `pybo/templates/question/question_list.html`에 저장한다.

그리고 아래와 같이 작성한다.

```html
{% if question_list %}
    <ul>
    {% for question in question_list %}
        <li><a href="/detail/{{ question.id }}/">{{ question.subject }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>질문이 없습니다.</p>
{% endif %}
```

템플릿을 보면 `{%` 와 `%}` 로 둘러싸인 문장들을 볼 수 있는데 이러한 것들을 템플릿 태그라고 한다.

`question_list.html`에 사용된 템플릿 태그들을 잠시 살펴보자.

`{% if question_list %}`는 다음처럼 해석된다.

: question_list 가 있다면 (※ question_list는 render_template 함수에서 전달받은 "질문목록"에 해당되는 데이터이다.)

`{% for question in question_list %}`는 다음처럼 해석된다.

: question_list 를 반복하며 순차적으로 하나씩 question에 대입

`{{ question.id }}` 는 다음처럼 해석된다.

: for문에 의해 대입된 question 객체의 id번호를 출력

`{{ question.subject }}` 는 다음처럼 해석된다.

: for문에 의해 대입된 question 객체의 제목을 출력

파이썬에 익숙하다면 여기에 사용된 태그들이 직관적으로 무엇을 의미하는지 쉽게 유추해 낼 수 있을 것이다.


#### 템플릿 태그

플라스크에서 사용되는 템플릿 태그는 사실 다음 3가지 유형만 알면 된다.

1. 분기

```html
{% if 조건문1 %}
    <p>조건문1에 해당되는 경우</p>
{% elif 조건문2 %}
    <p>조건문2에 해당되는 경우</p>
{% else %}
    <p>조건문1, 2에 모두 해당되지 않는 경우</p>
{% endif %}
```

2. 반복

```html
{% for item in list %}
    <p>순서: {{ loop.index }} </p>
    <p>{{ item }}</p>
{% endfor %}
```

템플릿 for문 안에서는 다음과 같은 `loop` 객체를 사용할 수 있다.

|loop 속성|설명|
|:---|:---|
|loop.index|루프내의 순서로 1부터 표시|
|loop.index()|루프내의 순서로 0부터 표시|
|loop.first|루프의 첫번째 순서인 경우 True|
|loop.last|루프의 마지막 순서인 경우 True|


3. 객체 출력

객체 출력하기 위한 태그의 사용법은 다음과 같다.

```html
{{ 객체 }}
```

예) `{{ item }}`, `{{ question.id }}`


#### 테스트

`http://localhost:5000/` 에 접속하면 변경된 화면을 볼 수 있다.

### 상세조회

조회된 화면에서 링크를 누르면 오류가 발생한다.

`http://localhost:5000/detail/2/` 와 같은 페이지에 대한 URL을 정의하지 않았기 때문이다.

#### 라우트 함수

이 URL이 동작할 수 있도록 main_views.py 파일에 다음과 같은 함수를 추가하자.

```python
@bp.route('/detail/<int:question_id>/')
def detail(question_id):
    question = Question.query.get(question_id)
    return render_template('question/question_detail.html', question=question)
```

http://localhost:5000/detail/2/ 와 같은 페이지가 요청되면 위 매핑룰에 의해 http://localhost:5000/detail/ `<int:question_id>` / 가 적용되어 question_id 에 2라는 값이 저장되고 detail 함수가 실행될 것이다. (※ 여기서 int는 숫자값이 매핑됨을 의미한다.)

detail함수는 index함수와 크게 다른 부분은 없다. 

다만 detail 함수에 매개변수 question_id가 추가된 것에 주목하자. 

매개변수 question_id에는 라우트 매핑시 사용된 <int:question_id>가 전달된다.

즉, http://localhost:5000/detail/2/ 페이지가 요청되면 main_views.py 파일의 detail 함수가 실행되고 detail 함수의 매개변수 question_id에는 2라는 값이 전달된다.


#### question_detail.html

question_detail.html 파일을 다음처럼 작성하자

```html
<h1>{{ question.subject }}</h1>

<div>
    {{ question.content }}
</div>
```

다시 Flask run을 한후 링크에 접속하면 잘 나오는 것을 볼 수 있다.

### 오류페이지

이번에는 아직 생성된 페이지가 아닌 곳에 접근하였을때 나오는 오류페이지를 만들도록 한다.

보통 이런 경우에는 "Not Found (404)" 오류를 리턴한다.

**HTTP 주요 응답코드의 종류**
|오류코드|설명|
|:---|:---|
|200|성공 (OK)|
|500|서버오류 (Internal Server Error)|
|404|서버가 요청한 페이지(Resource)를 찾을 수 없음 (Not Found)|

`main_views.py` 를 다음과 같이 수정하자

```python
@bp.route('/detail/<int:question_id>/')
def detail(question_id):
    question = Question.query.get_or_404(question_id)
    return render_template('question/question_detail.html', question=question)
```

Question.query.get(question_id) 로 사용했던 부분을 Question.query.get_or_404(question_id)로 바꾸어 주었다. 

get_or_404 함수는 해당 데이터를 찾을 수 없는 경우 404 페이지를 출력한다.

다시 http://localhost:5000/detail/30/ 페이지를 요청해 보자. 

404 페이지가 출력되는 것을 확인할 수 있을 것이다.

### 블루프린트

이번 장에서 작성한 질문목록과 상세조회 기능을 `main_views.py` 에 구현하였다. 

모든 기능을 `main_views.py` 에 구현할 수도 있겠지만 기능별로 블루프린트 파일로 분리하여 관리하는 것이 유리할 것이다.

질문목록과 상세조회 기능을 다음과 같이 `question_views.py` 파일로 분리해 보자.

quesiton_views.py 파일의 내용은 main_views.py 의 내용을 그대로 복사했다. 

다만 `bp = Blueprint('question', __name__, url_prefix='/question')` 처럼 블루프린트 생성시 `question` 이라는 이름을 사용하고 url_prefix도 `/question` 을 사용하여 main_views.py의 블루프린트와 구별을 하였다.

그리고 index 함수명을 `_list` 로 바꾸어 주고 라우트도 `/` 에서 `/list` 로 바꾸어 주었다.

그리고 question_views.py 에 등록한 블루프린트를 적용하기 위해서 `pybo/__init__.py` 파일도 다음처럼 수정해 주자.

```python
def create_app():
    ...
    # 블루프린트
    # ---------------------------------------- [edit] ---------------------------------------- #
    from .views import main_views, question_views
    app.register_blueprint(main_views.bp)
    app.register_blueprint(question_views.bp)
    # ---------------------------------------------------------------------------------------- #   
```

### url_for

그리고 main_views.py 파일은 질문목록과 상세조회 기능을 제거하여 다음과 같이 수정하자.

```python
from flask import Blueprint, url_for
from werkzeug.utils import redirect

bp = Blueprint('main', __name__, url_prefix='/')


@bp.route('/hello')
def hello_pybo():
    return 'Hello, Pybo!'


@bp.route('/')
def index():
    return redirect(url_for('question._list'))
```

detail함수는 제거하고 index함수는 `question._list` 에 해당하는 URL로 리다이렉트하게 수정해 주었다. 

redirect 함수는 입력으로 받은 URL로 리다이렉트해 주는 함수이다. 

url_for 함수는 라우트가 설정되어 있는 함수명으로 해당 URL을 역으로 찾아주는 함수이다. 

url_for 함수의 인수로 사용된 `question._list` 에서 question은 등록된 bp명에 해당되고 `_list`는 해당 bp에 등록되어 있는 함수명이다. 

`_list` 함수에 등록된 라우트는 `@bp.route('/list/')` 이므로 `url_for('question._list')` 는 bp의 프리픽스인 `/question/` 과 `/list/` 가 더해진 `/question/list/` 라는 URL을 리턴하게 된다.

마지막으로 템플릿에서 상세조회를 호출하는 부분을 다음과 같이 수정해 주도록 하자.

`question_list.html`

```html
{% if question_list %}
    <ul>
    {% for question in question_list %}
        <!-- ---------------------------------------- [edit] ---------------------------------------- -->
        <li><a href="{{ url_for('question.detail', question_id=question.id) }}">{{ question.subject }}</a></li>
        <!-- ---------------------------------------------------------------------------------------- -->
    {% endfor %}
    </ul>
{% else %}
    <p>질문이 없습니다.</p>
{% endif %}
```

index함수에서 사용했던 url_for를 템플릿에서도 그대로 사용할 수 있다. 

question.detail 함수는 question_id 매개변수를 필요로 하므로 question_id를 위와같이 전달해 주어야 한다.

이렇게 수정하고 목록조회 페이지에서 해당건을 클릭하면 잘 동작하는 것을 확인할 수 있을 것이다.

