# Лабораторная работа 3

## Дедлайн сдачи работы без пенальти

22.12.2024 00:00

## Как выполнять

-   Сделать форк **вашей** лабораторной работы номер два с названием `up09-lab3-{ваша_фамилия}`

## 1. Загрузка картинок

1. Добавьте путь для медиафайлов в `lab1/settings.py`

```python
MEDIA_URL = 'media/'

MEDIA_ROOT = os.path.join(BASE_DIR)
```

2. В `lab1/urls.py` добавьте пути для того, чтобы проект мог использовать медиа файлы

```python
from django.contrib import admin
from django.urls import path, include
from . import views
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.homepage),
    path('about/', views.about),
    path('posts/', include('posts.urls'))
    path('communities/', include('communities.urls'))
]

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

3. Установите пакет, который поможет в работе с картинками

```shell
python -m pip install Pillow
```

_Обратите внимание, что виртуальная среда Python должна быть активирована_

4. Добавьте к постам баннер

```python
class Post(models.Model):
    title = models.CharField(max_length=75)
    body = models.TextField()
    slug = models.SlugField()
    date = models.DateTimeField(auto_now_add=True)
    banner = models.ImageField(default='fallback.png', blank=True)

    def __str__(self):
        return self.title
```

_Не забудьте обновить базу данных путем миграции_

5. Проверьте, что в админке появилось поле `Banner`, где можно загрузить картинку

6. Загрузите всем постам по баннеру из папки `banners` в этом репозитории

7. Добавьте баннер на страницу `post_page.html`

```html
{% extends 'layout.html' %} {% block title %} {{ post.title }} {% endblock %} {%
block content %}
<section>
    <img class="banner" src="{{ post.banner.url }}" alt="{{ post.title }}" />
    <h1>{{ post.title }}</h1>
    <p>{{ post.date }}</p>
    <p>{{ post.body }}</p>
</section>
{% endblock %}
```

> [!WARNING]
>
> **Задание 1**
>
> Добавьте аватарки для сообществ

## 2. Создание пользователей

> [!WARNING]
>
> **Задание 2**
>
> Добавьте пользователей (`users`) к приложению
>
> -   Необходимо создать приложение `users`
> -   Создать представление `register_view`, которое будет располагаться по пути `localhost:8000/users/register` (_Представление должно использовать существующий шаблон сайта, но быть пока пустым_)

## 3. Создание авторизации

1. Добавьте в `users/views.py` код, который будет обрабатывать регистрацию пользователя

```python
from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm

# Create your views here.
def register_view(request):
    if request.method == "POST":
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("posts:list")
    else:
        form = UserCreationForm()
    return render(request, "users/register.html", { "form": form })
```

-   Импорт необходимых модулей и форм:
    -   `render` и `redirect` используются для отображения шаблонов и перенаправления.
    -   `UserCreationForm` — встроенная форма Django для создания пользователей.
-   Определение функции `register_view`:
    -   Это представление обрабатывает запросы, связанные с регистрацией пользователя.
-   Проверка метода запроса:
    -   Если запрос POST:
        -   Создаётся экземпляр `UserCreationForm`, заполняемый данными из запроса (request.POST).
        -   Если форма валидна (`form.is_valid()`):
            -   Сохраняются данные пользователя в базе данных (`form.save()`).
            -   Пользователь перенаправляется на URL с именем "posts:list".
    -   Если запрос не POST:
        -   Создаётся пустая форма `UserCreationForm` для заполнения.
-   Рендеринг шаблона:
    -   Возвращается HTML-страница `users/register.html`, где форма передаётся в контекст под ключом "form".

2. Добавьте форму в шаблон, который используется в представлении `register_view`

```html
{% extends 'layout.html' %} {% block title %} Регистрация {% endblock %} {%
block content %}
<h1>Регистрация нового пользователя</h1>
<form class="form-with-validation" action="/users/register/" method="post">
    {% csrf_token %} {{ form }}
    <button class="form-submit">Отправить</button>
</form>
{% endblock %}
```

> [!NOTE]
>
> `{% csrf_token %}` добавляет скрытое поле с уникальным токеном в HTML-форму, чтобы защитить её от атак типа "подделка межсайтовых запросов" (CSRF).
> Этот токен проверяется сервером, чтобы убедиться, что запрос отправлен авторизованным пользователем с доверенной страницы. Без этого механизма злоумышленник мог бы обманом заставить пользователя выполнить нежелательные действия, например, отправить форму или изменить данные, используя учётные данные пользователя.

3. Попробуйте перейти по `localhost:8000/users/register` и создать юзера. Проверьте создался ли юзер в админке (Таблица `Users`)

> [!WARNING]
>
> **Задание 3**
>
> Добавьте кнопку на форму регистрации в навигацию

## 4. Создание логин страницы

1. Добавьте новый маршрут в приложении `users`

```python
    path('login/', views.login_view, name="login"),
```

2. Создайте представление `login_view` в приложении `users`

```python
from django.contrib.auth import login

def login_view(request):
    if request.method == "POST":
        form = AuthenticationForm(data=request.POST)
        if form.is_valid():
            login(request, form.get_user())
            return redirect("posts:list")
    else:
        form = AuthenticationForm()
    return render(request, "users/login.html", { "form": form })
```

-   Проверка метода запроса:
    -   Если запрос POST:
        -   Создаётся объект AuthenticationForm, заполненный данными из запроса (data=request.POST).
        -   Проверяется, валидна ли форма (form.is_valid()):
            -   Если валидна:
                -   Авторизуется пользователь с использованием метода login и объекта пользователя, полученного из формы (form.get_user()).
                -   Пользователь перенаправляется на URL с именем "posts:list".
    -   Если запрос не POST:
        -   Создаётся пустая форма AuthenticationForm для заполнения.
-   Рендеринг шаблона:
    -   Возвращается HTML-страница users/login.html, где форма передаётся в контекст под ключом "form".

_Внутри представления `register_view` перед переходом на страницу "posts:list" добавьте автологин после создания аккаунта `login(request, form.save())`_

3. Создайте шаблон для логин формы

```html
{% extends 'layout.html' %} {% block title %} Вход {% endblock %} {% block
content %}
<h1>Вход</h1>
<form class="form-with-validation" action="/users/login/" method="post">
    {% csrf_token %} {{ form }}
    <button class="form-submit">Отправить</button>
</form>
{% endblock %}
```

4. Проверьте работает ли функция входа

> [!WARNING]
>
> **Задание 4**
>
> Добавьте кнопку для логина в навигацию
