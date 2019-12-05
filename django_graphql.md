# GraphQL (with Django)

Created: Dec 05, 2019 11:18 AM

## Django 프로젝트 생성

    # 프로젝트 폴더 생성
    mkdir django_graphql
    cd django_graphql
    
    # 가상환경 설치 및 실행
    python -m venv venv
    venv\Scripts\activate
    
    # Django 설치
    pip install Django
    
    # graphene-django 설치
    pip install graphene-django
    
    # Django 프로젝트 생성
    django-admin startproject project
    
    # Django 앱 생성
    cd project
    django-admin startapp board

    # project/project/settings.py
    # graphene-django 설정
    INSTALLED_APPS = [
        ...
        # set graphene_django
    		'graphene_django',
    ]

## 기본구조 변경

- django project 생성시 기본적으로 project/project 처럼 중복 폴더명이 생기므로, 혼란을 피하고자 project/config로 변경
1. 상위 project 폴더 내의 project 폴더명을 config로 변경
2. project/config/settings.py 수정 (빨간표시 라인 추가)

        import os
        import sys
        
        # Build paths inside the project like this: os.path.join(BASE_DIR, ...)
        BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
        sys.path.append(os.path.dirname(BASE_DIR))

3. project/config/wsgi.py 수정 (주황표시 코드 수정)

        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')

4. project/manage.py 수정 (주황표시 코드 수정)

        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')

## Model 반영

1. models파일 정의

        # project/board/models.py
        from django.db import models
        
        
        class Category(models.Model):
            name = models.CharField(max_length=100)
        
            def __str__(self):
                return self.name
        
        
        class Notice(models.Model):
            title = models.TextField()
            content = models.TextField()
            category = models.ForeignKey(Category, related_name='notice', on_delete=models.CASCADE)
        
            def __str__(self):
                return self.title

2. settings에 반영

        # project/config/settings.py
        INSTALLED_APPS = [
            ...
            # Install the ingredients app
        		'project.board',
        ]

3. DB migrate

        # pymysql 설치
        pip install mysqlclient

        # project/config/settings.py
        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': 'django_graphql',
                'USER': 'root',
                'PASSWORD': '...',
                'HOST': '...',
                'PORT': '3306',
                'OPTIONS': {
                    'init_command': "SET " +
                                    "sql_mode='STRICT_TRANS_TABLES'," +
                                    "MAX_EXECUTION_TIME=15000," +
                                    "character_set_connection=utf8," +
                                    "collation_connection=utf8_unicode_ci",
                    'connect_timeout': 30
                }
            }
        }

        # db migrate
        python manage.py makemigrations
        python manage.py migrate

4. db 데이터 넣기

    project/board/fixtures/board.json 파일 생성 후 아래 값 입력

        [
          {
            "model": "board.category",
            "pk": 1,
            "fields": {
              "name": "event"
            }
          },
          {
            "model": "board.category",
            "pk": 2,
            "fields": {
              "name": "notice"
            }
          },
          {
            "model": "board.notice",
            "pk": 1,
            "fields": {
              "title": "event1",
              "content": "event open",
              "category": 1
            }
          },
          {
            "model": "board.notice",
            "pk": 2,
            "fields": {
              "title": "event2",
              "content": "event open",
              "category": 2
            }
          },
          {
            "model": "board.notice",
            "pk": 3,
            "fields": {
              "title": "notice",
              "content": "notice warning",
              "category": 2
            }
          }
        ]

        python manage.py loaddata board
        ⇒ Installed 5 object(s) from 1 fixture(s)

5. schema 생성

        # porject/board/schema.py
        import graphene
        
        from graphene_django.types import DjangoObjectType
        
        from project.board.models import Category, Notice
        
        
        class CategoryType(DjangoObjectType):
            class Meta:
                model = Category
        
        
        class NoticeType(DjangoObjectType):
            class Meta:
                model = Notice
        
        
        class Query(object):
            all_categories = graphene.List(CategoryType)
            all_notices = graphene.List(NoticeType)
        
            def resolve_all_categories(self, info, **kwargs):
                return Category.objects.all()
        
            def resolve_all_notices(self, info, **kwargs):
                return Notice.objects.select_related('category').all()

        # porject/config/schema.py
        import graphene
        
        from project.board.schema import Query as boardQuery
        
        
        class Query(boardQuery, graphene.ObjectType):
            pass
        
        
        schema = graphene.Schema(query=Query)

## 설정

1. settings 반영

        # project/project/settings.py
        GRAPHENE = {
            'SCHEMA': 'project.config.schema.schema'
        }

2. set URL

        # project/project/urls.py
        from django.conf.urls import url
        from django.contrib import admin
        
        from graphene_django.views import GraphQLView
        
        urlpatterns = [
            url(r'^admin/', admin.site.urls),
            url(r'^graphql$', GraphQLView.as_view(graphiql=True)),
        ]

## Django 실행

    python manage.py runserver

## 호출

1. GraphQL

        query {
          allNotices {
            id
            title
            content
            category {
                id
                name
            }
          }
        }

2. 결과

        {
            "data": {
                "allNotices": [
                    {
                        "id": "1",
                        "title": "event1",
                        "content": "event open",
                        "category": {
                            "id": "1",
                            "name": "event"
                        }
                    },
                    {
                        "id": "2",
                        "title": "event2",
                        "content": "event open",
                        "category": {
                            "id": "2",
                            "name": "notice"
                        }
                    },
                    {
                        "id": "3",
                        "title": "notice",
                        "content": "notice warning",
                        "category": {
                            "id": "2",
                            "name": "notice"
                        }
                    }
                ]
            }
        }

참고 : [https://docs.graphene-python.org/projects/django/en/latest/tutorial-plain/](https://docs.graphene-python.org/projects/django/en/latest/tutorial-plain/)