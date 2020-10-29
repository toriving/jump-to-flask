# 1. 플라스크란 무엇인가?

![img](../../asset/flask-logo.png)

플라스크(Flask)는 2004년에 오스트리아의 아르민 로나허(Armin Ronacher)에 의해 만들어진 웹 프레임워크이다.

### 플라스크의 특징
- 마이크로 웹 프레임 워크 : 플라스크 코어(Core)의 심플함(Simple)과 확장성(Extensible) 있는 설계를 의미

- 심플 
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

- 확장성 : 장고(Django)가 모든 기능을 포함하고 있는 덩치 큰 프레임워크라고 한다면 플라스크(Flask)는 최소한의 기능만 포함하고 있는 프레임 워크

- 자유로움

[Reference](https://wikidocs.net/81039)