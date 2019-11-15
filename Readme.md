# Day8

- 어제 했던 것들
  - Naver Api사용해보기
    - 외부 사이트에 Request를 보낼 때, Post방식으로 요청하는 방법을 알았음
    - Request Body에 단순히 파라미터명과 파라미터값으로 이루어진 쌍이 아니라 json형식으로 파라미터를 보내는 방법
  - ORM 기초
    - Create, Read를 Django Shell 에서 실행시켜보기
    - ORM(Object Relational Mapping)이 무엇인지? 왜 사용해야 하는지?
    - [Wiki]( https://en.wikipedia.org/wiki/Object-relational_mapping )
- 오늘 해야할 것들
  - 기본 게시판 만들기
    - URL 분리하기
      - `urls.py`에다가 우리가 접속할 모든 주소를 명시했는데,  CRUD를 하다보면 만들어야 할 페이지가 점점 많아서 구분하기가 어려워 지기 때문에 각 역할을 하는 App마다 `urls.py`파일을 생성할 예정
    - 공용으로 사용할 수 있는(공유할 수 있는) HTML 파일 만들기
      - 반복되는 HTML 구조를 계속해서 새로 만들지 말고, 공통되는 부분은 하나의 파일로 묶어서 반복해서 사용함
    - CRUD 계속

- `django-admin startproject crudtest`

- `cd crudtest`

- `python manage.py startapp boards`

- *crudtest/settings.py*

- ```python
  INSTALLED_APPS = [
      'boards',
      ...
  ]
  ```

- *crudtest/urls.py*

- ```python
  # boards에 대한 처리를 분리시키기 위해 include를 사용
  from django.contrib import admin
  from django.urls import path, include
  
  urlpatterns = [
      path('admin/', admin.site.urls),
      path('boards/', include('boards.urls'))   
  ]
  
  
  ```

- *boards/urls.py*

- ```python
  from django.urls import path
  from . import views as boards_views  
  urlpatterns = [
      # 게시판의 메인페이지, 전체 리스트 페이지
      path('', boards_views.index),
      # 게시판의 새 글을 작성하는 페이지
      path('new/', boards_views.new),
  	# DB에 실제로 데이터를 등록하는 곳
      path('create/', boards_views.create),
      # 게시판의 글 하나를 상세히 보는 페이지
      # parameter는 기본적으로 string인데, path에서 먼저 형을 지정해두면 views에서 받을 때 형변환을 할 필요가 없다.
      path('<int:id>/', boards_views.show),
      # 기존의 글을 확인하고 수정하는 페이지
      path('<int:id>/edit/', boards_views.edit),
      # 수정된 내용을 DB에 반영하는 페이지
      path('<int:id>/update/', boards_views.update),
      # 실제로 DB에서 row를 삭제하는 곳
      path('<int:id>/delete/', boards_views.delete),
  ]
  ```

- *boards/models.py*

- ```python
  from django.db import models
  
  # Create your models here.
  class Board(models.Model):
      objects = models.Manager()
      title = models.CharField(max_length=32)
      contents = models.TextField()
      creator = models.CharField(max_length=12)
  ```

- *boards/views.py*

- ```python
  from django.shortcuts import render, redirect
  from .models import Board
  # Create your views here.
  
  def index(request):
      # Board 모델에 담긴 모든 글들을 가져와서 보여줌
      boards = Board.objects.all()
      context = {
          'boards': boards
      }
      return render(request, 'index.html', context)
  
  def new(request):
      return render(request, 'new.html')
  
  def create(request):
      title = request.GET['title']
      contents = request.GET['contents']
      creator = request.GET['creator']
      # new_board = Board(title=title, contents=contents, creator=creator)
      # new_board.save()
      # 위 두줄을 하나로 만드는 또하나의 방법
      new_board = Board.objects.create(title=title, contents=contents, creator=creator)
      return redirect(f'/boards/{new_board.id}')
  
  def show(request, id):
      # 테이블에서 유일한 값을 갖는 컬럼으로 검색을 할 수 있다. .filter같은 경우에 여러개의 결과가 나올 수도 있기 때문에 return 값이 기본적으로 list 형식을 띄고 있다. 하지만 .get의 경우 무조건 1개만 나온다. 여러개가 검색될 경우 에러가 발생한다.
      board = Board.objects.get(id=id)
      context = {
          'board': board
      }
      return render(request, 'show.html', context)
  
  def edit(request, id):
      # 원래 있던 내용이 들어있는 Form
      # Form에 기존에 있었던 내용을 삽입해서 수정할 수 있도록 한다.
      board = Board.objects.get(id=id)
      context = {
          'board': board
      }
      return render(request, 'edit.html', context)
  
  
  def update(request, id):
      # 실제로 update가 일어나는 곳
      board = Board.objects.get(id=id)
      title = request.GET['title']
      contents = request.GET['contents']
  
      board.title = title
      board.contents = contents
      board.save()
      return redirect(f'/boards/{board.id}')
  
  def delete(request, id):
      board = Board.objects.get(id=id)
      board.delete()
      return redirect('/boards')
  ```

- *boards/templates/base.html*

- ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>CRUD Test</title>
      <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
  </head>
  <body>
      <div class="d-flex flex-column flex-md-row align-items-center p-3 px-md-4 mb-3 bg-white border-bottom shadow-sm">
          <h5 class="my-0 mr-md-auto font-weight-normal">(주) 백수만세</h5>
              <nav class="my-2 my-md-0 mr-md-3">
              <a class="p-2 text-dark" href="/boards">Home</a>
              <a class="p-2 text-dark" href="/boards">Board</a>
              </nav>
          <a class="btn btn-outline-primary" href="#">Sign up</a>
      </div>
      <div class="container">
          {% block content %}
          {% endblock %}
      </div>
      <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
      <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
  </body>
  </html>
  ```

- 반복되는 코드의 사용을 막기 위해 `{% block content %}` 와  `{% endblock %}`를 사용했다.

- 실제 우리가 작성할 코드는 각 파일에 넣도록 한다.

- *boards/templates/index.html*

  ```html
  {% extends 'base.html' %} 
  
  {% block content %}
  <table class="table table-hover">
      <thead>
          <tr>
              <th scope="col">#</th>
              <th scope="col">제목</th>
              <th scope="col">작성자</th>
              <th scope="col">작성일자</th>
          </tr>
      </thead>
      <tbody>
          {% for board in boards %}
          <tr onclick="location.href='/boards/{{board.id}}'">
              <th scope="row">{{ board.id }}</th>
              <td>{{ board.title }}</td>
              <td>{{ board.creator }}</td>
              <td>2019-11-15</td>
          </tr>
          {% endfor %}
      </tbody>
  </table>
  <a href="/boards/new"><button type="button" class="btn btn-primary">새글쓰기</button></a>
  {% endblock %}
  ```

- *boards/templates/show.html*

- ```python
  {% extends 'base.html' %} 
  
  {% block content %}
  <table class="table table-hover">
      <thead>
          <tr>
              <th scope="col">#</th>
              <th scope="col">제목</th>
              <th scope="col">작성자</th>
              <th scope="col">작성일자</th>
          </tr>
      </thead>
      <tbody>
          {% for board in boards %}
          <tr onclick="location.href='/boards/{{board.id}}'">
              <th scope="row">{{ board.id }}</th>
              <td>{{ board.title }}</td>
              <td>{{ board.creator }}</td>
              <td>2019-11-15</td>
          </tr>
          {% endfor %}
      </tbody>
  </table>
  <a href="/boards/new"><button type="button" class="btn btn-primary">새글쓰기</button></a>
  {% endblock %}
  ```

- *boards/templates/new.html*

- ```html
  {% extends 'base.html' %}
  
  {% block content %}
  <form action="/boards/create">
      <div class="form-group">
          <label for="title">제목</label>
          <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요" name="title">
      </div>
      <div class="form-group">
          <label for="contents">내용</label>
          <textarea class="form-control" id="contents" rows="3" name="contents"></textarea>
      </div>
      <div class="form-group">
          <label for="creator">작성자</label>
          <input type="text" class="form-control" id="creator" name="creator">
      </div>
      <div class="form-group text-center">
          <input type="submit" value="작성하기" class="btn btn-success">
      </div>
  </form>
  {% endblock %}
  ```

- *boards/templates/show.html*

- ```html
  {% extends 'base.html' %}
  {% block content %}
  <h2>
      {{board.title}}
  </h2>
  <hr/>
  <p class="h4">{{board.contents}}</p>
  <p class="text-right"> created by {{board.creator}} </p>
  <div class="mt-3 text-center">
      <a href="/boards/{{board.id}}/edit" class="btn btn-danger">수정하기</a>
      <a href="/boards/{{board.id}}/delete" class="btn btn-warning text-white">삭제하기</a>
  </div>
  {% endblock %}
  ```

- *boards/templates/edit.html*

- ```html
  {% extends 'base.html' %}
  
  {% block content %}
  <form action="/boards/{{board.id}}/update">
      <div class="form-group">
          <label for="title">제목</label>
          <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요" name="title" value="{{board.title}}">
      </div>
      <div class="form-group">
          <label for="contents">내용</label>
          <textarea class="form-control" id="contents" rows="3" name="contents">{{board.contents}}</textarea>
      </div>
      <div class="form-group">
          <label for="creator">작성자</label>
          <input type="text" class="form-control" id="creator" name="creator" value="{{board.creator}}" readonly>
      </div>
      <div class="form-group text-center">
          <input type="submit" value="작성하기" class="btn btn-success">
      </div>
  </form>
  {% endblock %}
  ```

- 나머지는 로직만 있고 view를 사용하지 않는다. view를 사용하지 않을 경우 redirect를 통해 다른 위치로 보내준다.

- 

