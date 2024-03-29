# YaMDb API

Проект YaMDb собирает отзывы пользователей на произведения. Сами произведения в YaMDb не хранятся, здесь нельзя посмотреть фильм или послушать музыку.
Произведения делятся на категории, такие как «Книги», «Фильмы», «Музыка». Например, в категории «Книги» могут быть произведения «Винни-Пух и все-все-все» и «Марсианские хроники», а в категории «Музыка» — песня «Давеча» группы «Жуки» и вторая сюита Баха. Список категорий может быть расширен (например, можно добавить категорию «Изобразительное искусство» или «Ювелирка»). 

Произведению может быть присвоен жанр из списка предустановленных (например, «Сказка», «Рок» или «Артхаус»). 
Добавлять произведения, категории и жанры может только администратор.

Благодарные или возмущённые пользователи оставляют к произведениям текстовые отзывы и ставят произведению оценку в диапазоне от одного до десяти (целое число); из пользовательских оценок формируется усреднённая оценка произведения — рейтинг (целое число). На одно произведение пользователь может оставить только один отзыв.
Пользователи могут оставлять комментарии к отзывам.
Добавлять отзывы, комментарии и ставить оценки могут только аутентифицированные пользователи. 

### Ресурсы API YaMDb

- Ресурс **auth**: аутентификация.
- Ресурс **users**: пользователи.
- Ресурс **titles**: произведения, к которым пишут отзывы (определённый фильм, книга или песенка).
- Ресурс **categories**: категории (типы) произведений («Фильмы», «Книги», «Музыка»). Одно произведение может быть привязано только к одной категории.
- Ресурс **genres**: жанры произведений. Одно произведение может быть привязано к нескольким жанрам.
- Ресурс **reviews**: отзывы на произведения. Отзыв привязан к определённому произведению.
- Ресурс **comments**: комментарии к отзывам. Комментарий привязан к определённому отзыву.

### Пользовательские роли

- **Аноним** — может просматривать описания произведений, читать отзывы и комментарии.
- **Аутентифицированный пользователь (user)** — может, как и Аноним, читать всё, дополнительно он может публиковать отзывы и ставить оценку произведениям (фильмам/книгам/песенкам), может комментировать чужие отзывы; может редактировать и удалять свои отзывы и комментарии. Эта роль присваивается по умолчанию каждому новому пользователю.
- **Модератор (moderator)** — те же права, что и у Аутентифицированного пользователя плюс право удалять любые отзывы и комментарии.
- **Администратор (admin)** — полные права на управление всем контентом проекта. Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям.
- **Суперюзер Django** — обладет правами администратора (admin)

### Алгоритм регистрации пользователей

1. Пользователь отправляет POST-запрос на добавление нового пользователя с параметрами `email` и `username` на эндпоинт `/api/v1/auth/signup/`.
2. **YaMDB** отправляет письмо с кодом подтверждения (`confirmation_code`) на адрес `email`.
3. Пользователь отправляет POST-запрос с параметрами `username` и `confirmation_code` на эндпоинт `/api/v1/auth/token/`, в ответе на запрос ему приходит `token` (JWT-токен).
4. При желании пользователь отправляет PATCH-запрос на эндпоинт `/api/v1/users/me/` и заполняет поля в своём профайле.

### Установка и запуск проекта

1. Склонируйте репозиторий;
2. Находясь в папке с кодом создайте виртуальное окружение `python -m venv venv`;
3. Активируйте виртуальное окружение (Windows: `source venv\scripts\activate`; Linux/Mac: `sorce venv/bin/activate`);
4. Установите зависимости `python -m pip install -r requirements.txt`.

Для запуска сервера разработки, находясь в директории проекта, выполните команды:
```
python manage.py migrate
python manage.py runserver
```

Для создания учётной записи superuser, находясь в директории проекта, выполните команду:
```
python manage.py createsuperuser
```
С помощью учётной записи superuser возможно использовать графический интерфейс администратора для наполнения БД минимально необходимыми для работы проекта данными.

Интерфейс администратора проекта доступен по адресу:
`http://127.0.0.1:8000/admin/`

### Пример

#### 1. Получение списка всех произведений

`http://127.0.0.1:8000/api/v1/titles/`

- **GET**: Получить список всех объектов. Права доступа: **Доступно без токена**

Вывод:
```
{
  "count": 0,
  "next": "string",
  "previous": "string",
  "results": [
    {
      "id": 0,
      "name": "string",
      "year": 0,
      "rating": 0,
      "description": "string",
      "genre": [
        {
          "name": "string",
          "slug": "string"
        }
      ],
      "category": {
        "name": "string",
        "slug": "string"
      }
    }
  ]
}
```

#### 2. Добавление произведения

`http://127.0.0.1:8000/api/v1/titles/`

- **POST**: Добавить новое произведение. Права доступа: **Администратор**. Нельзя добавлять произведения, которые еще не вышли (год выпуска не может быть больше текущего). При добавлении нового произведения требуется указать уже существующие категорию и жанр.

Используется аутентификация с использованием JWT-токенов

**Header parameter name**: `Bearer`

**Required scopes**: `write:admin`

Ввод:
```
{
  "name": "string",
  "year": 0,
  "description": "string",
  "genre": [
    "string"
  ],
  "category": "string"
}
```

Вывод:
```
{
  "id": 0,
  "name": "string",
  "year": 0,
  "rating": 0,
  "description": "string",
  "genre": [
    {
      "name": "string",
      "slug": "string"
    }
  ],
  "category": {
    "name": "string",
    "slug": "string"
  }
}
```

### В проекте используется следующий стек технологий:
- Python 3.7.9
- Django 3.2
- Django REST Framework 3.12.4

### Авторы проекта:
- Дмитрий Краснокуцкий (`https://github.com/DKrasnokutsky/`)
- Владислав Тутункин (`https://github.com/TutunkinVladislav`)
- Павел Лыгин (`https://github.com/StanDinButters`)
