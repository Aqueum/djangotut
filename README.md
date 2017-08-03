# djangoTut setup

## Setup vagrant box
- Pick box from [Vagrant](https://app.vagrantup.com/boxes/)
- I chose [ubuntu/xenial64](https://app.vagrantup.com/ubuntu/boxes/xenial64)
  - create new folder, I called mine `djangoTut`
  - I added the path to djangoTut as an alias called djangoTut by:
    - nano .bash_profile
    - add the line: `alias djangoTut='cd "/Users/PATH_TO_YOUR_DJANGOTUT_FOLDER"'
  - navigate to new folder in terminal
  - `vagrant init ubuntu/xenial64`
  - edit vagrant file
    - uncomment `# config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
`
    - change ports `80` and `8080` to `8000` as that's what the django development server uses
    - so you now have `config.vm.network "forwarded_port", guest: 8000, host: 8000, host_ip: "127.0.0.1`
  - `vagrant up` if first time or `vagrant reload` to load the new vagrant file
  - `vagrant box update` to get latest version
  - `vagrant ssh` to SSH into your new box
  
## Install Django
- `python3 -m django --version` to confirm no django installed (or check version)
  - [remove django](https://docs.djangoproject.com/en/1.11/topics/install/#removing-old-versions-of-django) if you have an out of date version
- [install database](https://docs.djangoproject.com/en/1.11/topics/install/#database-installation), if not SQLite, but tutorial uses SQLite
- install pip
  - `sudo apt-get update --fix-missing` 
  - `sudo apt install python3-pip` to install pip
  - `pip3 install --upgrade pip` to upgrade pip
- setup [virtualenv](https://virtualenv.pypa.io/en/stable/)
  - `sudo pip3 install virtualenv` to install virtualenv
  - `cd /vagrant`
  - `virtualenv ENV` to create a new virtual environment in directory ENV
  - `source ENV/bin/activate` to activate virtual env
- `pip install Django` to install Django
- `python -m django --version` to check installed version - I have 1.11.4
  
## Set up Django project
- `django-admin startproject mysite` to create django settings, database configuration & file structure
- check project working
  - `cd /vagrant/mysite`
  - `python manage.py runserver 0.0.0.0:8000` to run lightweight **non-production** development server - [here's why 0.0.0.0.:8000](https://stackoverflow.com/questions/33129651/access-web-server-on-virtualbox-vagrant-machine-from-host-browser)
  - Check 'It worked!' at [localhost:8000](http://localhost:8000/)
  
## Create app
- ctrl-c to exit server
- `cd /vagrant/mysite` to get to folder with manage.py
- `python manage.py startapp polls` to create polls app starter

## Add a view
- edit `polls/views.py` to: 
```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
- add new polls/urls.py with content: 
```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```  
- add url pattern `url(r'^polls/', include('polls.urls')),` and `from django.conf.urls import include` to mysite/urls.py so we have:
```
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]
```
- check it works:
  - `python manage.py runserver 0:8000`
  - browse `http://127.0.0.1:8000/polls/`
- url() takes for arguments: regex, view, kwargs & name - see [docs](https://docs.djangoproject.com/en/1.11/ref/urls/#url) and foot of [tutorial page](https://docs.djangoproject.com/en/1.11/intro/tutorial01/)

## Set up database
- If not SQLite see [databases](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-DATABASES), ensure database set up with `CREATE DATABASE database_name;` in database interactive prompt, and that the user given to `mysite/settings.py` has create database privelages.
- change `mysite/settings.py` to have `LANGUAGE_CODE = 'en-gb'` and `TIME_ZONE = 'Europe/London'`
- in terminal `cd /vagrant/mysite` & `source ../ENV/bin/activate`
- `python manage.py migrate` to create tables for INSTALLED_APPS

## Create models
- Edit `polls/models.py` to:
```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

## Activate models
- add `'polls.apps.PollsConfig',` to top of `INSTALLED_APPS`. list in `mysite/settings.py`
- `python manage.py makemigrations polls` to create migrations for model changes
- optional `python manage.py sqlmigrate polls 0001` to see what SQL Django thinks is needed
- optional `python manage.py check` to check for project problems
- `python manage.py migrate` to create model tables in database

## - Optional, play with API
- `python manage.py shell` to invoke Python shell with env variable & import bath
- `from polls.models import Question, Choice` to import model classes
- `Question.objects.all()` to check no questions
- `from django.utils import timezone`
- `q = Question(question_text="What's new?", pub_date=timezone.now())` to create new question
- `q.save()` to save question
- `q.id` to get question ID (1 in this case)
- `q.question_text` to get question text ("What's new?" in this case)
- `q.pub_date` to get question timestamp (datetime.datetime(2017, 8, 3, 10, 18, 23, 238260, tzinfo=<UTC>) in this case)
- `q.question_text = "What's up?"` change question
- `q.save()` to save question
- `Question.objects.all()` to get all questions



# To launch
- in terminal `djangoTut` or navigate to the folder with your vm
- `./rds1` (a bash script I set up to do the following)
  - `vagrant up`
  - `vagrant ssh`
- `/vagrant/rds2` (a bash script I set up to do the following)
  - `cd /vagrant/mysite`
  - `source ../ENV/bin/activate`
  - `python manage.py runserver 0:8000`
- [localhost:8000/polls](http://localhost:8000/polls)