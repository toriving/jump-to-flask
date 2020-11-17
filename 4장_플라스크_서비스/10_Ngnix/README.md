# 10. Nginx

이번장에서는 파이보가 사용할 웹서버인 엔진엑스(Nginx)를 설치하고 적용해 보자.

### Nginx에 대하여

Nginx는 요새 급성장하고 있는 웹서버로 아파치(Apache)를 대신할 차세대 웹서버로 주목받고 있다. 

Nginx는 높은 성능을 위해서 개발된 웹서버로 점점 사용자가 증가하는 추세이며 특히 파이썬 웹 프레임워크인 장고나 플라스크등에서 주로 사용되는 서버이다. 

또한 Nginx를 사용하기 위한 설정도 무척 간단하여 쉽게 사용할수 있다.

### Nginx 설치

Nginx를 다음과 같이 관리자 권한으로 설치하자.

`sudo apt install nginx`

### Nginx 설정

이제 Nginx에서 동적페이지 요청시 WSGI 서버를 호출하도록 다음과 같이 설정해 보자.

설정은 https://wikidocs.net/81078 를 참조하자

### Nginx 실행

Nginx는 설치할때 자동으로 실행된다. 

따라서 Nginx에 변경된 설정을 적용하기 위해 다음과 같이 Nginx를 재시작해 주어야 한다.

`sudo systemctl restart nginx`

### Nginx 파이보 동작확인 

테스트한다.
