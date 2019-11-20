# Django시작하기 (with PyCharm)

1. project 폴더 생성
    - 폴더를 생성 했으면 pycharm에서 열어준다.

2. Django 가상환경 설치
    - pycharm 하단의 Terminal을 열어 'pythom -m venv venv' 명령어 실행
    - 프로젝트 내부에 venv(가상환경)폴더가 생긴다.

3. Pycharm에 현재 프로젝트 가상환경 셋팅
    - 메뉴에서 File => Setting (Ctrl + Alt + S)
        => Project: [프로젝트명] => Project Interpreter => 우측 톱니바퀴 => Add
        => Exisiting environment 체크 => 완료
        ![](https://github.com/tlsgud216/tlsgud216.github.io/blob/master/images/set_venv.png?raw=true)
    - Pychamr의 Terminal을 껐다가 다시 켰을때 경로 앞에 (venv)가 있으면 완료
     tip) cmd에서 수동으로 venv를 실행시키는 방법
        - 프로젝트 내 venv 폴더가 있는 위치에서 'venv\Scripts\activate.bat' 실행

+) Pycharm을 이용해서 New Project를 해주면 1, 2, 3을 한번에 해줄 수 있지만 장고는 주로 가상환경에서 실행하기 때문에 venv 폴더를 한번 짚고 넘어가고 싶어 풀어서 적어보았다.

4. requirements폴더 생성
    - Django 프로젝트 라이브러리들을 관리하기 위해 requirements 폴더를 생성 후에 txt 파일에 일괄적으로 관리할 수 있다.
    - base.txt 파일 생성후에 'django == 2.1.4' 입력 (여기서는 Django 2.1.4 버전 기준으로 작성.)
    - Terminal에서 'pip install -r requirements\base.txt' 명령어 실행 (이후 라이브러리 추가가 필요하다면 txt파일에 적고 명령어를 다시 실행해주면 된다.)
     tip) 운영환경별로 라이브러리를 다르게 하고싶다면 [environment_name].txt 파일을 생성 해서 '-r base.txt'를 적고 라이브러리를 적은 후 'pip install -r requirements\[environment_name].txt' 명령어를 실행해주면 된다. 예) production.txt => 'pip install -r requirements\production.txt'
    - Django 설치 버전확인 : cmd에서 'python -m django --version' 명령어 실행 후 '2.1.4' 노출되면 정상

5. Django 프로젝트 생성
    - cmd에서 'django-admin startproject project' 명령어를 실행하면 Django project 폴더가 생성이 된다.
    ![](https://github.com/tlsgud216/tlsgud216.github.io/blob/master/images/django_default_structure.png?raw=true)

    - Django 실행 : python manage.py runserver (manage.py가 있는 폴더 위치에서 실행)

6. 기본구조 변경
    - 앱/설정 폴더 분리
        1) 상위 'project' 폴더 내에 app폴더를 생성 (app을 별도의 폴더로 관리하기 위함.)
        2) 하위 'project' 폴더 명 config로 변경 (project라는 폴더명이 중복으로 2개가 생겨 구분하기 위함 + 설정폴더 관리)
            - 폴더명이 변경되었으므로 파일내부도 변경필요
                config/settings.py에 밑줄 친 줄 추가 및 파란글자로 수정
                    import os
                    import sys

                    # Build paths inside the project like this: os.path.join(BASE_DIR, ...)
                    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
                    sys.path.append(os.path.dirname(os.path.dirname(BASE_DIR)))


                    ROOT_URLCONF = 'project.config.urls'

                    WSGI_APPLICATION = 'project.config.wsgi.application'

                config/wsgi.py 파란글자 수정
                    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')

                manage.py 파란글자 수정
                    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
        ![](https://github.com/tlsgud216/tlsgud216.github.io/blob/master/images/seperate_app_settings.png?raw=true)
        3) 제대로 구조가 변경되었다면 DJango 재실행하여 확인.

7. 설정파일 분리
    - local / product 등의 환경 분리
        1) config폴더내에 settings폴더를 생성 후 settings.py를 base.py로 변경해서 settings폴더 안에 넣어준다.
            (config/settings.py => config/settings/base.py)

        2) 환경파일 생성
            - config/settings폴더 내에 local.py를 생성후에 base.py를 import 시킨다. (from .base import *)
            - 환경별로 다르게 설정할 값을 local.py에 옮긴다(예 : DEBUG, DATABASES)
            ![](https://github.com/tlsgud216/tlsgud216.github.io/blob/master/images/seperate_env.png?raw=true)
        
        3) Django 실행시 환경을 지정해줘야한다.
            python manage.py runserver --settings=config.settings.[환경]