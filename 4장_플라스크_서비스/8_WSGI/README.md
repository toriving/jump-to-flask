# 8.WSGI

이번장에서는 `flask run` 대신 세상에 오픈된 운영환경에서는 어떻게 서버를 운영해야 하는지에 대해서 알아보기로 하자.

### 웹브라우저

실제 사용자는 파이보 시스템에 접속하기 위해 웹브라우저를 이용하여 파이보 홈페이지에 접속할 것이다. 

이때 웹브라우저는 요청하는 URL정보와 필요한 데이터들을 웹서버에 전송한다.

웹브라우저가 웹서버에 요청하는 정보는 크게 두 가지로 분류된다.

#### 정적 페이지 요청

웹브라우저로 다음 URL을 요청해 보자.

`http://localhost:5000/static/style.css`

```css
.comment {
    border-top:dotted 1px #ddd;
    font-size:0.7em;
}
```

가 보인다.

항상 같은 파일이 보인다. js나 jpg, png 파일도 동일하다

이러한 파일을 정적파일이라고 한다.

이렇게 웹브라우저에서 css, js, jpg, png와 같은 정적파일을 요청하는 것을 정적페이지 요청이라고 한다.

#### 동적 페이지 요청

이번에는 웹브라우저에서 파이보 메인페이지(예:http://52.78.8.100:5000/)로 접속하는 경우를 생각해 보자. 

파이보 서버는 이런 요청이 들어오면 질문목록을 읽어서 리턴한다. 

이때 응답 데이터인 질문목록은 데이터베이스의 내용에 따라 수시로 변하게 된다.

이렇게 응답이 수시로 변하는 요청을 동적페이지 요청이라고 한다.

### 웹서버

웹 서버는 웹브라우저의 정적요청과 동적요청에 대한 처리를 담당하는 서버이다.

※ 파이보는 여러 웹서버중 요새 가장 인기있는 엔진엑스(Nginx) 웹서버를 사용할 것이다.

웹 서버에 정적페이지 요청이 들어오면 파일을 읽어서 응답해주면 되기 때문에 간단하다. 

하지만 동적페이지 요청인 경우는 조금 복잡해 진다.

웹 서버에 동적페이지 요청이 들어오면 웹 서버는 파이썬 프로그램을 호출해야 한다. 

하지만 대부분의 웹 서버는 파이썬 프로그램을 호출할 수 있는 기능이 없다. 

어떻게 파이썬 프로그램을 호출해야 하는지 모르기 때문이다.

이러한 이유로 생겨난 것이 바로 WSGI(Web Server Gateway Interface) 위스키이다.

웹 서버는 파이썬 프로그램을 호출하는 방법은 모르지만 WSGI 서버를 호출하는 방법은 알고 있다. 

웹 서버의 설정파일에 동적페이지 요청이 발생할 경우 WSGI 서버를 호출하라고 설정하기만 하면 된다.

따라서 웹 서버에서 파이썬 프로그램을 호출하기 위해서는 WSGI 서버가 필요하다.

### WSGI 서버

WSGI 서버는 웹서버에서 동적페이지 요청을 처리하기 위해 호출되는 서버이다. 

WSGI 서버의 종류에는 여러가지가 있지만 가장 많이 사용되는 WSGI 서버는 uwsgi와 Gunicorn이다.

※ 파이보 시스템은 Gunicorn을 WSGI 서버로 사용할 것이다.

웹서버에 동적페이지 요청이 발생하면 WSGI 서버가 호출되고 WSGI 서버는 다시 WSGI 어플리케이션을 호출하게 된다. 

여기서 알수 있는 중요한 사실은 실제 동적페이지 요청은 결국 WSGI 어플리케이션이 처리한다는 점이다. 

WSGI 어플리케이션에는 장고(Django), 플라스크(Flask), 토네이도(Tornado)등이 있다. 

파이보 시스템이 사용하는 WSGI 어플리케이션은 우리가 지금껏 공부해온 플라스크이다.

※ WSGI 서버는 웹서버와 WSGI 어플리케이션 중간에 있기 때문에 WSGI 미들웨어(Middleware) 또는 WSGI 컨테이너(Container)라고 불리우기도 한다.

### WSGI 순서도

![img](https://wikidocs.net/images/page/81076/4-08_1.png)

웹브라우저에서 정적페이지 요청이 발생하면 웹서버가 처리하고 동적페이지 요청이 발생하면 WSGI서버에 의해 플라스크 프로그램이 호출된다.

AWS Lightsail 서버에서 운영할 파이보 시스템은 위와 같은 그림대로 구축할 것이다.

>flask run 명령으로 동작하는 내장서버는 웹 서버와 WSGI 서버의 기능들을 모두 포함하고 있다. 
단, 기능들이 단순화되어 있고 대량 요청이나 동시 요청에 대한 기능들이 부족하기 때문에 운영환경에서 사용하기에는 적절하지 않다.