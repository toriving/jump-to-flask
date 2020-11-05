# 1. 네비게이션바

메인페이지로 돌아갈수 있는 방법이 현재 없음

따라서 네비게이션바를 만들어 보자

### 네비게이션바

네비게이션바는 모든 페이지에서 보여야 하므로 base.html 템플릿 `<body>` 태그 바로 밑에 추가핮

`pybo\templates\base.html`

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
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{{ url_for('main.index') }}">Pybo</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">계정생성</a>
            </li>
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
<!-- ---------------------------------------------------------------------------------------- -->
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
</body>
</html>
```

항상 메인으로 이동할 수 있는 "Pybo"라는 로고(navbar-brand)를 가장 좌측에 두었고 그리고 우측에는 미구현 상태인 "계정생성"과 "로그인" 링크를 추가함

이제 "Pybo" 로고를 클릭하면 항상 메인페이지로 돌아갈 수 있게 되었음

모든 페이지에서 네비게이션바가 나타나는 것을 확인할 수 있음

부트스트랩 네비게이션바는 창이 줄어들면 사라지는 반응형임

이때 메뉴바가 생기는데 메뉴바를 클릭해야 "계정생성"과 "로그인"이 나온다

하지만 현재 부트스트랩 자바스크립트 파일이 없어서 나오지 않음.

기존에 다운받은 `bootstrap-{ver}.zip`에서 `bootstrap.min.js` 을 static에 옮기자.

또한 제이쿼리도 다운받자

https://jquery.com/download/

base.html의 </body> 태그 바로위에 다음처럼 추가하자.

`pybo\templates\base.html`

```html
<...>
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<!-- jQuery JS -->
<script src="{{ url_for('static', filename='jquery-3.4.1.min.js') }}"></script>
<!-- Bootstrap JS -->
<script src="{{ url_for('static', filename='bootstrap.min.js') }}"></script>
<!-- ---------------------------------------------------------------------------------------- -->
</body>
</html>
```

이제 메뉴바를 누르면 잘 나온다

### Include 

플라스크에는 템플릿의 특정 위치에 HTML코드를 삽입할수 있는 include기능이 있다. 

이번에는 include기능을 이용하여 네비게이션바를 base.html 템플릿에 포함하도록 해보자.

먼저 navbar.html 파일을 다음과 같이 신규로 작성하도록 하자.

`\pybo\templates\navbar.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<!-- 네비게이션바 -->
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{{ url_for('main.index') }}">Pybo</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
        aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">계정생성</a>
            </li>
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
<!-- ---------------------------------------------------------------------------------------- -->
```

그리고 base.html을 다음과 같이 수정하자

`pybo\templates\base.html`

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
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% include "navbar.html" %}
<!-- ---------------------------------------------------------------------------------------- -->
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
{% block content %}
{% endblock %}
<!-- 기본 템플릿 안에 삽입될 내용 End -->
<!-- jQuery JS -->
<script src="{{ url_for('static', filename='jquery-3.4.1.min.js') }}"></script>
<!-- Bootstrap JS -->
<script src="{{ url_for('static', filename='bootstrap.min.js') }}"></script>
</body>
</html>
```