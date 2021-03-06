# 12. 오류페이지

플라스크 웹 어플리케이션은 라우트 함수로 등록되지 않은 페이지의 요청을 받을 경우 다음과 같은 404 오류페이지를 보여주게 되어 있다.

하지만 404 페이지가 매우 밋밋함을 볼 수 있다. 다음의 방법을 이용하면 404(Page not found) 또는 500(Server error) 오류코드 발생시 보여지는 화면을 수정할 수 있다.

다음처럼 `pybo/__init__.py` 파일을 수정하자.

```python
# ---------------------------------------- [edit] ---------------------------------------- #
from flask import Flask, render_template
# ---------------------------------------------------------------------------------------- #
...

# ---------------------------------------- [edit] ---------------------------------------- #
def page_not_found(e):
    return render_template('404.html'), 404
# ---------------------------------------------------------------------------------------- #    


def create_app():
    ...
    # ---------------------------------------- [edit] ---------------------------------------- #
    # 오류페이지
    app.register_error_handler(404, page_not_found)
    # ---------------------------------------------------------------------------------------- #    

    return app

```

404 오류가 발생할 경우 page_not_found 함수가 호출되도록 수정하였다. 

page_not_found 함수의 매개변수 e는 오류를 의미하며 오류코드(code)와 오류내용(description) 속성을 포함한다. 

만약 오류의 내용을 표시하고 싶다면 템플릿에 이 오류를 전달하여 표시할 수 있을 것이다.

그리고 page_not_found 함수에서 return render_template('404.html'), 404 처럼 return문의 두 번째 인수에 404라고 명시적으로 전달해 주었다. 

이렇게 하면 클라이언트, 즉 브라우저에도 404 오류코드가 전달되게 된다. 

만약 404를 생략할 경우 404.html 템플릿이 렌더링되어 404 오류페이지가 보이긴 하지만 클라이언트는 200(정상) 오류코드를 수신하게 된다.

그리고 404 오류시 표시할 템플릿을 다음처럼 생성하자.

`pybo\templates\404.html`

```html
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
{% extends 'base.html' %}
{% block content %}
<div class="container">
    <div class="row justify-content-center">
        <div class="col-12 text-center">
            <span class="display-1 d-block">404</span>
            <div class="mb-4 lead">페이지를 찾을 수 없습니다.</div>
            <a href="/" class="btn btn-link">홈으로 돌아가기</a>
        </div>
    </div>
</div>
{% endblock %}
<!-- ---------------------------------------------------------------------------------------- -->
```

존재하지 않는 url을 호출 해보자

이처럼 500 (internal server error 도 설정할 수 있다.)