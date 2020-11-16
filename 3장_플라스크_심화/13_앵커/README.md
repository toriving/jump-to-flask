# 13. 앵커

HTML에는 URL호출시 원하는 위치로 이동시켜 주는 앵커(anchor) 태그라는 것이 있다. 

예를 들어 HTML코드 중간에 `<a name="flask"></a>` 라는 앵커태그가 있다면 이 HTML을 호출하는 URL 뒤에 `#flask` 라고 붙여주면 해당 페이지가 호출되면서 해당 앵커로 스크롤이 이동되게 된다.

### 답변앵커

먼저 상세조회 화면에 있는 답변들에 앵커기능을 추가해 보자.

#### 답변앵커 추가

답변 작성, 수정시에 이동해야할 앵커 태그를 상세조회 화면에 다음과 같이 추가하자.

`pybo\templates\question\question_detail.html`

```html
<...>
<h5 class="border-bottom my-3 py-2">{{ question.answer_set|length }}개의 답변이 있습니다.</h5>
{% for answer in question.answer_set %}
<!-- ---------------------------------------- [edit] ---------------------------------------- -->
<a name="answer_{{ answer.id }}"></a>
<!-- ---------------------------------------------------------------------------------------- -->
<...>
```

답변이 반복되어 표시되는 for문 바로 다음에 `<a name="answer_{{ answer.id }}"></a>` 앵커태그를 추가해 주었다. 

앵커 태그의 name은 유일한 값이어야 하므로 `answer_{{ answer.id }}` 처럼 답변의 id번호가 포함되게 만들어 주었다.

#### redirect 수정

이제 답변등록 또는 답변수정시 해당 앵커 태그로 이동될 수 있도록 해당 함수들을 수정해 보자.

답변등록 또는 답변수정후에 사용했던 기존의 방법은 다음과 같다.

`return redirect(url_for('question.detail', question_id=question_id))`

위 방식에 앵커태그를 포함하려면 다음과 같이 수정해야 한다.

`return redirect('{}#answer_{}'.format(
    url_for('question.detail', question_id=question_id), answer.id))`

리다이렉트되는 URL 뒷 부분에 `#answer_2`와 같은 앵커태거를 추가하기 위해 format 함수가 사용되었다.

따라서 answer_views.py의 create, modify 함수는 다음과 같이 변경해야 한다.

`pybo\views\answer_views.py`

```python
...
@bp.route('/create/<int:question_id>', methods=('POST',))
@login_required
def create(question_id):
    ...
    if form.validate_on_submit():
        ...
        # ---------------------------------------- [edit] ---------------------------------------- #
        return redirect('{}#answer_{}'.format(
            url_for('question.detail', question_id=question_id), answer.id))
        # ---------------------------------------------------------------------------------------- #
    ...

@bp.route('/modify/<int:answer_id>', methods=('GET', 'POST'))
@login_required
def modify(answer_id):
    ...
    if request.method == "POST":
        ...
        if form.validate_on_submit():
            ...
            # ---------------------------------------- [edit] ---------------------------------------- #
            return redirect('{}#answer_{}'.format(
                url_for('question.detail', question_id=answer.question.id), answer.id))
            # ---------------------------------------------------------------------------------------- #
    ...
...
```

modify 함수에 redirect를 사용하는 부분이 두군데 있는데 오류가 발생하여 redirect되는 첫번째 부분은 앵커태그로 이동할 필요가 없으므로 수정할 필요가 없다.

#### 테스트

테스트하자

### 댓글앵커

댓글앵커 기능도 답변앵커와 동일한 방법으로 구현하면 된다.

#### 댓글앵커 추가

댓글이 반복되는 구간에 댓글앵커를 다음처럼 추가해 주자.

`pybo\templates\question\question_detail.html`

```html
<...>
<!-- 질문 댓글 Start -->
{% if question.comment_set|length > 0 %}
<div class="mt-3">
{% for comment in question.comment_set %}
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <a name="comment_{{ comment.id }}"></a>
    <!-- ---------------------------------------------------------------------------------------- -->

<...>

<!-- 답변 댓글 Start -->
{% if answer.comment_set|length > 0 %}
<div class="mt-3">
{% for comment in answer.comment_set %}
    <!-- ---------------------------------------- [edit] ---------------------------------------- -->
    <a name="comment_{{ comment.id }}"></a>
    <!-- ---------------------------------------------------------------------------------------- -->
<...>
```


#### redirect 수정

그리고 다음처럼 comment_views.py 파일을 다음처럼 수정하자.

`pybo\views\comment_views.py`

```python
...

@bp.route('/create/question/<int:question_id>', methods=('GET', 'POST'))
@login_required
def create_question(question_id):
    ...
    if request.method == 'POST' and form.validate_on_submit():
        ...
        # ---------------------------------------- [edit] ---------------------------------------- #
        return redirect('{}#comment_{}'.format(
            url_for('question.detail', question_id=question_id), comment.id))
        # ---------------------------------------------------------------------------------------- #
    ...


@bp.route('/modify/question/<int:comment_id>', methods=('GET', 'POST'))
@login_required
def modify_question(comment_id):
    ...
    if request.method == 'POST':
        ...
        if form.validate_on_submit():
            ...
            # ---------------------------------------- [edit] ---------------------------------------- #
            return redirect('{}#comment_{}'.format(
                url_for('question.detail', question_id=comment.question.id), comment.id))
            # ---------------------------------------------------------------------------------------- #
    else:
        ...
    ...

...


@bp.route('/create/answer/<int:answer_id>', methods=('GET', 'POST'))
@login_required
def create_answer(answer_id):
    ...
    if request.method == 'POST' and form.validate_on_submit():
        ...
        # ---------------------------------------- [edit] ---------------------------------------- #
        return redirect('{}#comment_{}'.format(
            url_for('question.detail', question_id=answer.question.id), comment.id))
        # ---------------------------------------------------------------------------------------- #
    ...


@bp.route('/modify/answer/<int:comment_id>', methods=('GET', 'POST'))
@login_required
def modify_answer(comment_id):
    ...
    if request.method == 'POST':
        ...
        if form.validate_on_submit():
            ...
            # ---------------------------------------- [edit] ---------------------------------------- #
            return redirect('{}#comment_{}'.format(
                url_for('question.detail', question_id=comment.answer.question.id), comment.id))
            # ---------------------------------------------------------------------------------------- #
    else:
        ...
    ...
```

질문댓글을 등록, 수정했을 때 그리고 답변댓글을 등록, 수정했을 때 해당 앵커태그로 이동할 수 있도록 수정하였다. 

댓글을 삭제할 경우에는 앵커태그가 없으므로 앵커태그를 추가할 필요가 없다.

#### 테스트

테스트하자