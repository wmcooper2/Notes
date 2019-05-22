# List View Tutorial

Make a project named "firstproject" with an app named "someapp"
These notes were adapted from "Django for Beginners" (William Vincent)

I changed the name of the model `Post` to `SomeModel`,
`mb_project` to `firstproject`, and
`posts` app to `someapp` to be more general.

### Environment Setup
* install pipenv: `pip3 install pipenv`
* start venv: `pipenv shell`
* make venv: `pip3 install django==2.1.5`

### Django Setup
* Start a project: `django-admin startproject firstproject .`
* Check: `python manage.py runserver`
* Start an app: `python manage.py startapp someapp`  
* Edit: `firstproject/settings.py`
    ```python
    INSTALLED_APPS = ['someapp.apps.SomeappConfig',]
    ```  
* Create database: `python manage.py migrate`
* Check: `python manage.py runserver`

### Create Model
* Edit: `someapp/models.py`
    ```python
    from django.db import models
    class SomeModel(models.Model):
        text = models.TextField()
        def __str__(self):
            return self.text[:50]
    ```

### Activate Model
* Run
    ```bash
    python manage.py makemigrations someapp
    python manage.py migrate
    ```

### Setup Admin
* Run: `python manage.py createsuperuser`
* Check: `python manage.py runserver`
* Link app to admin, edit `someapp/admin.py`
    ```python
    from django.contrib import admin
    from .models import SomeModel
    admin.site.register(SomeModel)
    ```

### Create View
* Edit: `someapp/views.py`
    ```python
    from django.views.generic import ListView
    from .models import SomeModel
    class HomePageView(ListView):
        model = SomeModel
        template_name = 'home.html'
        context_object_name = 'special_context_name'  # optional
    ```

### Create Template
* Setup template dir/file
    ```bash
    mkdir templates
    touch templates/home.html
    ```
* Edit: `firstproject/settings.py`
    ```python
    TEMPLATES = [{
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
    }]
    ```
* Edit: `templates/home.html`
    ```html
    <h1>First Project Homepage</h1>
      <ul>
        {% for thing in object_list %}
          <li>{{ thing.text }}</li>
        {% endfor %}
      </ul>
    ```

### Create URLs
* Edit: `firstproject/urls.py`
    ```python
    from django.contrib import admin
    from django.urls import path, include
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('someapp.urls')),
    ]
    ```
* Edit: `someapp/urls.py`
    ```python
    from django.urls import path
    from .views import HomePageView
    urlpatterns = [
        path('', HomePageView.as_view(), name='home'),
    ]
    ```
* Check: `python manage.py runserver`
* You should see the home page with "First Project Homepage".

### Create Tests
* Edit: `someapp/tests.py`
    ```python
    from django.test import TestCase
    from django.urls import reverse
    from .models import SomeModel
    class SomeModelTest(TestCase):
        def setUp(self):
            SomeModel.objects.create(text='just a test')
        def test_text_content(self):
            thing = SomeModel.objects.get(id=1)
            expected_object_name = f'{thing.text}'
            self.assertEqual(expected_object_name, 'just a test')
    class HomePageViewTest(TestCase):
        def setUp(self):
            SomeModel.objects.create(text='this is another test')
        def test_view_url_exists_at_proper_location(self):
            resp = self.client.get('/')
            self.assertEqual(resp.status_code, 200)
        def test_view_url_by_name(self):
            resp = self.client.get(reverse('home'))
            self.assertEqual(resp.status_code, 200)
        def test_view_uses_correct_template(self):
            resp = self.client.get(reverse('home'))
            self.assertEqual(resp.status_code, 200)
            self.assertTemplateUsed(resp, 'home.html')
    ```
* Run tests: `python manage.py test`

### Heroku
* Edit: `projectroot/Pipfile`
    ```bash
    [requires]
    python_version = "3.7"
    ```
* Run: `pipenv lock`
* Edit: `projectroot/Procfile`
    ```bash
    web: gunicorn firstproject.wsgi --log-file -
    ```
* Run: `pipenv install gunicorn==19.9.0`
* Edit: `firstproject/settings.py`
    ```python
    ALLOWED_HOSTS = ['*']
    ```
* Make Heroku Account
    ```bash
    heroku create
    heroku git:remote -a <the name it gave you from 'heroku create'>
    heroku config:set DISABLE_COLLECTSTATIC=1
    git push heroku master
    heroku ps:scale web=1
    heroku open
    ```
