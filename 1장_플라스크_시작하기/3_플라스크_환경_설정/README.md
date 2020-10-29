# 3. 플라스크 환경 설정

1. 가상환경 (virtualenv)
2. 플라스크 설치
3. pip 업그레이드

### 가상환경 (virtualenv)
윈도우 및 venv를 이용한 방법은 [여기](https://wikidocs.net/81041) 를 참조하자.

virtualenv가 설치되어 있는 wsl2 / mac / linux 계열 환경에서는 venv라는 폴더명, python3.8을 이용하여 다음과 같이 설치 및 실행

```shell script
$ virtualenv venv --python=python3.8
$ source venv/bin/activate
```

### 플라스크 설치

가상환경에 들어가 있는 상태에서 pip를 이용

```shell script
$ pip install Flask
```

### pip 업그레이드

따로 하지 않았다.