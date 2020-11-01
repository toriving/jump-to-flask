# 4. 모델

**이번 장은 원문을 그대로 가져온 표현들이 많습니다.**

파이보는 질문과 답변 서비스이다. 질문을 작성하거나 답변을 작성하면 필수적으로 데이터가 생성된다. 따라서 데이터를 저장하고 읽고 수정하는 등의 기능들이 반드시 필요함

보통 파이보와 같은 웹서비스는 이러한 데이터 처리를 위해 데이터베이스를 사용함

### ORM

ORM(Object Relational Mapping)은 개발자가 직접 쿼리문을 작성하지 않고 테이블과 매핑된 모델 객체를 통해서 데이터 작업을 처리하는 방식이다.

간단히 예를 들면 질문 테이블(question)에 데이터를 한 건 입력하려면 다음과 같은 쿼리를 수행해야 한다.

```mysql
insert into question (subject, content) values ('제목', '내용');
```

ORM을 사용하면 쿼리문 대신 모델을 이용하여 다음과 같이 데이터를 한 건 입력할 수 있다.

```python
question = Question(subject='제목', content='내용')
db.session.add(question)
```

ORM을 사용하더라도 내부적으로 쿼리문이 만들어 진다.

**ORM의 장점**


전통적으로 데이터베이스를 사용하는 프로그램들은 데이터베이스의 데이터를 조회하거나 저장하기 위해 쿼리문을 사용해야 했다. 이 방식은 여전히 많이 사용되고 있는 방식이지만 몇 가지 단점이 있다. 개발자마다 다양한 쿼리문이 만들어지고 또 잘못 작성된 쿼리가 시스템의 성능을 저하시키기도 하기 때문이다. 그리고 만약 데이터베이스가 MySQL에서 오라클로 변경된다면 프로그램에서 사용한 쿼리문을 모두 해당 데이터베이스의 규칙에 맞게 수정해야 하는 어려움도 발생한다.

ORM(Object Relational Mapping)을 사용하면 데이터베이스의 테이블을 모델화하여 사용하기 때문에 위에서 열거한 SQL방식의 단점이 모두 없어진다. ORM을 사용하면 개발자별로 독특한 쿼리문이 만들어질수가 없고 또 쿼리를 잘못 작성할 가능성도 적어진다. 그리고 데이터베이스 종류가 변경되더라도 쿼리문이 아닌 모델을 사용하기 때문에 프로그램을 수정할 필요가 없어진다.


### 플라스크 ORM 라이브러리

장고는 ORM이 내장되어 있지만 플라스크는 ORM이 내장되어 있지 않아서 파이썬 ORM중 가장 많이 사용되는 SQLAlchemy를 사용할 것이다.

또한, 데이터베이스를 직접 수정하는 대신 모델을 통해 변경할 수 있도록 도와주는 Flask-Migrate 라이브러리를 사용한다.

Flask-Migrate 라이브러리를 설치하면 SQLAlchemy도 함께 설치된다.

```shell script
$ pip install Flask-Migrate
```


### ORM 적용

파이보에 ORM을 적용하기 위해 `config.py` 파일을 pybo 프로젝트 내에 생성하자

```python
import os

BASE_DIR = os.path.dirname(__file__)

SQLALCHEMY_DATABASE_URI = 'sqlite:///{}'.format(os.path.join(BASE_DIR, 'pybo.db'))
SQLALCHEMY_TRACK_MODIFICATIONS = False
```

BASE_DIR에 `pybo.db`라는 데이터베이스를 만들어 사용한다.

SQLite는 주로 개발시스템 또는 소규모 프로젝트에서 사용되는 가벼운 파일기반의 데이터베이스이다. 개발시에는 SQLite를 사용하여 빠르게 개발하고 실제 운영시스템에서는 좀 더 규모있는 DB를 사용하는 것이 일반적인 개발 패턴이다.

SQLALCHEMY_TRACK_MODIFICATIONS는 SQLAlchemy의 이벤트들을 처리하기 위한 옵션인데 추가적인 메모리를 사용하기도 하고 파이보 서비스에는 필요없는 기능이므로 False로 비활성화시키도록 하자.

그리고 `pybo/__init__.py`를 수정하자.

```python
from flask import Flask
from flask_migrate import Migrate
from flask_sqlalchemy import SQLAlchemy

import config

db = SQLAlchemy()
migrate = Migrate()

def create_app():
    app = Flask(__name__)
    app.config.from_object(config)

    # ORM
    db.init_app(app)
    migrate.init_app(app, db)

    # 블루프린트
    from .views import main_views
    app.register_blueprint(main_views.bp)

    return app
```

config.py에 작성한 항목들을 app.config 환경변수로 읽어들이기 위해 app.config.from_object(config) 문장을 추가해 주었다. 

그리고 전역 변수로 db, migrate 객체를 만들고 create_app 함수에서 init_app 메서드를 이용하여 초기화 해 주었다. 

**이것은 플라스크에서 자주 사용되는 패턴이다.**

db 객체를 create_app 함수내에서 생성하면 블루프린트와 같은 다른 모듈에서 db객체를 import하여 사용할수 없기 때문에 이처럼 create_app 함수 밖에서 생성하고 실제 객체 초기화는 create_app에서 수행하는 패턴이다.


### ORM 수행

ORM을 사용할 준비가 되었다면 `flask db init` 명령을 수행해 보자.

```shell script
$ flask db init
  Creating directory /mnt/c/workspace/jump-to-flask/project/migrations ...  done
  Creating directory /mnt/c/workspace/jump-to-flask/project/migrations/versions ...  done
  Generating /mnt/c/workspace/jump-to-flask/project/migrations/alembic.ini ...  done
  Generating /mnt/c/workspace/jump-to-flask/project/migrations/env.py ...  done
  Generating /mnt/c/workspace/jump-to-flask/project/migrations/README ...  done
  Generating /mnt/c/workspace/jump-to-flask/project/migrations/script.py.mako ...  done
  Please edit configuration/connection/logging settings in '/mnt/c/workspace/jump-to-flask/project/migrations/alembic.ini'
  before proceeding.
```

`flask db init` 명령을 수행하면 데이터베이스 관리를 위한 초기파일들이 migrations라는 디렉터리에 자동으로 생성된다.
`flask db init` 은 최초 한번만 수행하면 되며, 모델을 추가하거나 변경할때는 `flask db migrate`와 `flask db upgrade` 명령 두개만 반복적으로 사용하면 된다.

1. flask db migrate - 모델을 신규로 생성하거나 변경할때 사용
2. flask db upgrade - 변경된 내용을 적용할때 사용

물론 이 외에도 여러 명령이 있긴 하지만 특별한 경우가 아니라면 사용할 일이 많지 않을 것이다. 위 2개의 명령외에 어떤 것들이 있는지 확인을 위해서는 다음과 같이 flask db 명령을 수행하면 된다.

```shell script
$ flask db
Usage: flask db [OPTIONS] COMMAND [ARGS]...

  Perform database migrations.

Options:
  --help  Show this message and exit.

Commands:
  branches   Show current branch points
  current    Display the current revision for each database.
  downgrade  Revert to a previous version
  edit       Edit a revision file
  heads      Show current available heads in the script directory
  history    List changeset scripts in chronological order.
  init       Creates a new migration repository.
  merge      Merge two revisions together, creating a new revision file
  migrate    Autogenerate a new revision file (Alias for 'revision...
  revision   Create a new revision file.
  show       Show the revision denoted by the given symbol.
  stamp      'stamp' the revision table with the given revision; don't run...
  upgrade    Upgrade to a later version

```

### 모델 생성

파이보가 사용할 데이터 모델을 만들어 보자

#### 모델의 속성

질문(Question) 모델에는 다음과 같은 속성이 필요할 것이다.

|속성명|설명|
|:---|:---|
|id|질문의 고유번호|
|subject|질문의 제목|
|content|질문의 내용|
|create_data|질문을 작성한 일시|

답변(Answer) 모델에는 다음과 같은 속성이 필요하다.

|속성명|설명|
|:---|:---|
|id|답변의 고유번호|
|question_id|질문의 고유번호 (어떤 질문의 답변인지 알아야하므로 질문의 고유번호가 필요하다.)|
|content|답변의 내용|
|create_data|답변을 작성한 일시|

#### models.py

이렇게 생각한 속성을 바탕으로 질문(Question)과 답변(Answer)에 해당되는 모델을 pybo/models.py 파일에 정의해 보자.

```python
from pybo import db


class Question(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    subject = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)


class Answer(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    question_id = db.Column(db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'))
    question = db.relationship('Question', backref=db.backref('answer_set'))
    content = db.Column(db.Text(), nullable=False)
    create_date = db.Column(db.DateTime(), nullable=False)
```

Question과 Answer 모델은 db.Model 클래스를 상속하여 만든 클래스이다. 

여기서 사용된 db는 pybo/__init__.py 에서 생성한 SQLAlchemy의 객체이다. 

모델 클래스의 각각의 속성은 db.Column을 사용하여 생성할 수 있다.

Question 모델은 고유번호(id), 제목(subject), 내용(content) 그리고 작성일시(create_date)를 속성으로 갖도록 작성했다. 

고유번호(id) 속성은 중복되지 않는 유일한 값을 의미한다. 

즉, 여러개의 Question 모델객체가 생성되더라도 id 값은 중복될 수 없음을 의미한다. 

테이블의 개념으로는 프라이머리키(Primary Key)라고 한다. 

db.Column 속성으로 primary_key=True 로 설정하면 해당 속성은 프라이머리키가 된다.

그리고 id, subject, content, create_time과 같은 속성에는 db.Integer, db.String, db.Text, db.DateTime과 같은 컬럼타입을 반드시 설정해 주어야 한다. d

b.Integer는 고유번호(id)와 같은 숫자값에 사용되고 db.String은 제목(subject)처럼 글자수의 길이가 제한된 텍스트에 사용된다. 

내용(context)처럼 글자수를 제한할 수 없는 텍스트는 db.Text를 사용해야 한다. 

생성일시(create_date)는 날짜와 시간에 해당되는 db.DateTime을 사용해야 한다.

**※ SQLAlchemy에서 primary=True 이고 db.Integer로 생성된 첫번째 속성은 자동증가 항목으로 설정된다. 즉, 모델 객체 생성시 id에 값을 지정하지 않고 저장하더라도 값이 자동으로 1씩 증가하여 저장된다.**

위에서 사용된 nullable=False 는 해당 속성에 비어있는 값이 들어올 수 없음을 의미한다. 즉, Null 값을 허용하지 않겠다는 의미이다. nullable=False 를 지정하지 않으면 디폴트로 nullable=True가 적용되어 Null 값을 허용하게 된다.

Answer 모델은 질문에 대한 답변에 해당되므로 Question 모델을 속성으로 가져갈 수 있다는 점이 특이하다. 기존 모델을 속성으로 가져가기 위해서는 다음처럼 2개의 추가 속성이 필요하다.

```python
question_id = db.Column(db.Integer, db.ForeignKey('question.id', ondelete='CASCADE'))
question = db.relationship('Question', backref=db.backref('answer_set'))
```

question_id 는 Question모델의 id값을 의미하며 이를 나타내기 위해 db.ForeignKey를 사용해야 한다. 

db.ForeignKey는 다른 모델과의 연결을 의미한다. 

ondelete=CASCADE의 의미는 이 답변과 연결된 질문(Question)이 삭제될 경우 답변(Answer)도 함께 삭제된다는 의미이다.

question 속성은 답변모델에서 질문모델을 참조하기 위해서 추가된 속성이다. 

즉, answer.question.subject 처럼 답변 모델 객체(answer)를 통해서 질문모델 객체(question)를 참조할 수 있게 된다.

이를 위해서는 db.relationship 을 이용하여 속성을 추가해 주어야 한다. 

db.relationship에서 사용된 backref 속성은 answer.question.subject 와는 반대로 질문에서 답변모델을 참조하기 위해서 사용되는 속성이다. 

하나의 질문에는 여러개의 답변이 작성될 수 있는데 어떤 질문에 해당되는 객체가 a_question 이라면 이 질문에 작성된 답변들을 참조하기 위해서 a_question.answer_set 과 같이 사용할 수 있다.

SQLAlchemy에서 사용하는 속성(Field)의 타입은 이것 외에도 많다. 

[여기](https://docs.sqlalchemy.org/en/13/core/type_basics.html) 에서 어떤것들이 있는지 참고하도록 하자.


### 테이블 생성

이제 모델을 생성했으므로 플라스크의 Migrate 기능을 이용하여 데이터베이스 테이블을 생성할 수 있게 되었다. 하지만 테이블 생성전에 한가지 먼저 해 주어야 할 일이 있다.

테이블을 생성하기 위해서는 플라스크의 Migrate를 사용해야 하는데 이 Migrate가 우리가 작성한 모델을 인식할 수 있도록 다음과 같이 `pybo/__init__.py` 파일을 수정해야 한다.

```python
from flask import Flask
from flask_migrate import Migrate
from flask_sqlalchemy import SQLAlchemy

import config

db = SQLAlchemy()
migrate = Migrate()


def create_app():
    app = Flask(__name__)
    app.config.from_object(config)

    # ORM
    db.init_app(app)
    migrate.init_app(app, db)
    # ---------------------------------------- [edit] ---------------------------------------- #
    from . import models
    # ---------------------------------------------------------------------------------------- #

    # 블루프린트
    from .views import main_views
    app.register_blueprint(main_views.bp)

    return app
```

위와 같이 from . import models 라는 문장을 추가하면 migrate 객체가 우리가 작성한 모델인 models.py 파일을 참조할 수 있게 된다.

이제 다음과 같이 데이터베이스 변경을 위해 `flask db migrate` 명령을 수행하도록 하자.

```shell script
$ flask db migrate
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'question'
INFO  [alembic.autogenerate.compare] Detected added table 'answer'
  Generating /mnt/c/workspace/jump-to-flask/project/migrations/versions/6187f85e2ccb_.py ...  done
```

그리고 변경사항을 적용하기 위해 `flask db upgrade`를 수행하자.

```shell script
$ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 6187f85e2ccb, empty message
```

이렇게 수행하면 C:\projects\myproject\pybo.db 파일이 생성될 것이다. 

pybo.db 파일은 SQLite 데이터베이스의 데이터파일이다. 

아마도 여기에 question과 answer라는 테이블이 생성되었을 것이다. 

테이블명은 모델 클래스의 이름과 동일하게 생성된다. 

따라서 Question 모델의 테이블명은 question, Answer 모델의 테이블명은 answer가 될 것이다.


### DB Browser for SQLite

`pybo.db` 를 열어 확인해보려면 SQLite의 GUI 도구인 "DB Browser for SQLite"를 사용하면 확인해 볼 수 있다.

SQLite는 [여기](https://sqlitebrowser.org/dl/) 에서 다운 받을 수 있다.

개인적으로 포터블을 좋아해서 포터블을 받았다.

해당 툴의 대한 설명은 [원문](https://wikidocs.net/81045) 을 참조하자.


### 모델 사용하기

이번에는 플라스크 쉘을 이용하여 모델을 사용하는 법에 대해 알아보자.

`flask shell` 명령어를 이용해 flask shell 을 실행하자

```shell script
$ flask shell
Python 3.8.2 (default, Apr 27 2020, 15:53:34)
[GCC 9.3.0] on linux
App: pybo [development]
Instance: /mnt/c/workspace/jump-to-flask/project/instance
>>>
```

#### Question 생성

Qusetion과 Answer 모델은 플라스크 쉘에서 다음처럼 import 하여 사용할 수 있다.

```shell script
>>> from pybo.models import Question, Answer
```

먼저 Question 모델을 이용하여 질문 데이터를 만들어 보자.

```shell script
>>> from datetime import datetime
>>> q = Question(subject='당신은 누구신가요?', content='당신에 대해서 알고 싶습니다.', create_date=datetime.now())
```

Question 모델의 create_date 컬럼타입은 DateTime이므로 datetime.now()로 현재일시를 대입하였다. 

위처럼 생성한 Question 모델의 q객체를 데이터베이스에 저장하기 위해서는 다음과 같이 SQLAlchemy의 db객체를 사용해야 한다.

```shell script
>>> from pybo import db
>>> db.session.add(q)
>>> db.session.commit()
```

db객체의 session은 db와 연결된 세션을 의미한다.

이 세션을 이용하여 데이터를 저장하고 삭제하는등의 데이터베이스 처리를 할 수 있다. 

데이터를 신규로 저장할때는 위처럼 db.session.add(q) 와 같이 add 함수를 사용한다. 

그리고 세션을 통해서 데이터를 저장하거나 수정하거나 삭제한 이후에는 반드시 db.session.commit() 함수로 커밋을 해주어야 한다. 

만약 세션을 통해서 처리한 작업을 취소하고 싶을 때에는 db.session.rollback() 함수로 롤백을 수행하면 처리했던 작업들이 취소된다. 

한가지 주의할 점은 이미 커밋된 작업은 취소가 불가능하다는 점이다.

**※ SQLAlchemy 설정에 의해 자동커밋(autocommit)을 하도록 설정할 경우에는 db.session.commit() 을 수행하지 않더라도 항상 자동으로 커밋이 된다.**

데이터가 1건 생성되면 다음처럼 id값이 생성된다.

```shell script
>>> q.id
1
```

두번째 질문을 만들어보자

```shell script
>>> q = Question(subject='플라스크 모델 질문입니다.', content='id는 자동으로 생성되나요?', create_date=datetime.now())
>>> db.session.add(q)
>>> db.session.commit()
>>> q.id
2
```

#### Question 조회

아래와 같은 방식으로 저장된 Question을 조회할 수 있다.

```shell script
>>> Question.query.all()
[<Question 1>, <Question 2>]
>>>
```

첫번째 Question 데이터만 조회하려면 다음과 같은 필터를 사용해야 한다.

```shell script
>>> Question.query.filter(Question.id==1).all()
[<Question 1>]
```

필터에 일치하는 데이터를 리턴해주기에 다건을 의미하는 리스트가 리턴된다.

id는 유일한 값이므로 filter 대신 get을 이용하여 조회할 수 있다.

```shell script
>>> Question.query.get(1)
<Question 1>
>>>
```

get으로 조회할 경우 리스트가 아닌 Question 모델 객체가 리턴되었다. filter는 다건을 리턴하지만 get은 한건만 리턴하기 때문이다.

이번에는 subject에 "플라스크"라는 문자열이 포함된 데이터만 조회하는 방법에 대해서 알아보자.

```shell script
>>> Question.query.filter(Question.subject.like('%플라스크%')).all()
[<Question 2>]
>>>
```

"플라스크"라는 문자열이 포함된 두번째 Question 데이터만 리턴되는 것을 확인할 수 있다. 

Question.subject.like('%플라스크%')의 의미는 "Question 모델의 subject 속성에 '플라스크'라는 문자열이 포함되어 있는가?" 라고 해석할 수 있다. 

이 문장에서 사용된 % 는 좌우측에 다른 문자열을 포함하는지에 대한 식별자로 만약 Question.subject.like('플라스크%') 와 같이 사용하면 "플라스크"라는 문자열로 시작되는지를 의미하고 Question.subject.like('%플라스크') 와 같이 사용되면 "플라스크"라는 문자열로 끝나는지를 의미한다.

**※ 만약 대소문자를 구분하지 않고 포함된 문자열을 찾으려면 like 대신 ilike를 사용하면 된다.**

데이터를 조회하는 다양한 사용법은 SQLAlchemy 문서를 참조하도록 하자. [이 문서](https://docs.sqlalchemy.org/en/13/orm/query.html) 는 개발시 필수적으로 항상 참조해야 하는 문서이다.


#### Question 수정

이번에는 저장한 Question 모델 데이터를 수정해 보도록 하자.

먼저 다음과 같이 2번째 데이터를 조회한다.

```shell script
>>> q = Question.query.get(2)
>>> q
<Question 2>
>>>
```

그리고 subject속성을 다음과 같이 수정하자.

```shell script
>>> q.subject = 'Flask Model Question'
>>>
```

여기까지만 해서는 수정이 되지 않는다. 반드시 다음처럼 커밋을 수행해 주어야 변경된 데이터가 반영된다는 것에 주의하도록 하자.

```shell script
>>> from pybo import db
>>> db.session.commit()
```

#### Question 삭제

이번에는 첫번째 질문을 삭제해 보도록 하자.

```shell script
>>> q = Question.query.get(1)
>>> db.session.delete(q)
>>> db.session.commit()
```

`db.session.delete(q)` 를 수행하면 객체 q에 해당하는 데이터가 삭제된다.

실제로 삭제되었는지 다음처럼 `Question.query.all()` 로 확인해 보자.

```shell script
>>> Question.query.all()
[<Question 2>]
>>>
```

첫번째 질문은 삭제되고 두번째 질문만 조회되는 것을 확인할 수 있다.


#### Answer 작성

이번에는 답변(Answer) 모델 데이터를 생성해 보자.

```shell script
>>> from datetime import datetime
>>> from pybo.models import Question, Answer
>>> from pybo import db
>>> q = Question.query.get(2)
>>> a = Answer(question=q, content='네 자동으로 생성됩니다.', create_date=datetime.now())
>>> db.session.add(a)
>>> db.session.commit()
>>>
```

답변 데이터를 만들기 위해서는 질문이 필요하므로 id가 2인 질문을 먼저 조회하여 q라는 변수로 저장한 후 Answer 모델의 question 속성에 대입해 주었다. 

Answer 모델에는 question_id 속성이 있는데 위처럼 question=q 와 같이 relationship으로 연결된 Question 모델속성인 question에 값을 대입하면 question_id 에 값을 지정하지 않아도 자동으로 저장된다.

Answer모델도 Question모델과 마찬가지로 유일한 값을 의미하는 id값이 자동으로 생성되었음을 아래와 같이 확인할 수 있다.

```shell script
>>> a.id
1
```

#### Answer 조회

답변을 조회하는 방법은 질문과 마찬가지로 Answer의 id값을 사용하면 된다.

```shell script
>>> a = Answer.query.get(1)
>>> a
<Answer 1>
>>>
```

Answer객체인 a를 사용하면 답변에 연결된 질문도 조회해 볼수 있다.

```shell script
>>> a.question
<Question 2>
>>>
```

Answer 모델 객체인 a를 통해서 질문을 찾는것은 모델에 정의되어 있기 때문에 매우 쉽다. 그렇다면 질문을 이용하여 답변을 찾는 것은 가능할까?

가능하다. 다음처럼 하면 된다.

```shell script
>>> q.answer_set
[<Answer 1>]
>>>
```

Question과 Answer는 서로 연결되어 있기 때문에 Answer모델에서 backref로 지정한 answer_set을 사용하면 질문과 연결된 답변을 가져올 수 있다.

backref는 정말 신통방통한 SQLAlchemy의 기능이다. 이 방법은 매우 자주 사용하니 꼭 기억해 두도록 하자.

