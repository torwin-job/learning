**практический пример совмещения паттернов:**

- ✅ **Specification** — бизнес-правила фильтрации
    
- ✅ **Repository** — абстракция над SQLAlchemy
    
- ✅ Flask + SQLAlchemy
    

---

## 📦 Структура проекта

```
project/
├── app.py               # Flask-приложение
├── models.py            # SQLAlchemy модели
├── repository.py        # Репозиторий
├── specifications.py    # Паттерн "Спецификация"
```

---

## 🔹 `models.py` — модель пользователя

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = "users"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
    age = db.Column(db.Integer)
    country = db.Column(db.String(50))
```

---

## 🔹 `specifications.py` — реализация паттерна Specification

```python
from abc import ABC, abstractmethod
from sqlalchemy import and_, or_, not_
from models import User

# Абстрактный базовый класс для спецификаций
class Specification(ABC):
    @abstractmethod
    def to_expression(self):
        """Метод должен возвращать SQLAlchemy-выражение"""
        pass

    # Комбинирование спецификаций через логическое И
    def __and__(self, other):
        return AndSpecification(self, other)

    # Комбинирование спецификаций через логическое ИЛИ
    def __or__(self, other):
        return OrSpecification(self, other)

    # Логическое отрицание спецификации
    def __invert__(self):
        return NotSpecification(self)


# Спецификация: возраст больше указанного
class AgeGreaterThan(Specification):
    def __init__(self, age):
        self.age = age

    def to_expression(self):
        return User.age > self.age


# Спецификация: страна равна указанной
class CountryIs(Specification):
    def __init__(self, country):
        self.country = country

    def to_expression(self):
        return User.country == self.country


# Комбинация нескольких спецификаций через AND
class AndSpecification(Specification):
    def __init__(self, *specs):
        self.specs = specs

    def to_expression(self):
        return and_(*(spec.to_expression() for spec in self.specs))


# Комбинация через OR
class OrSpecification(Specification):
    def __init__(self, *specs):
        self.specs = specs

    def to_expression(self):
        return or_(*(spec.to_expression() for spec in self.specs))


# Отрицание (NOT)
class NotSpecification(Specification):
    def __init__(self, spec):
        self.spec = spec

    def to_expression(self):
        return not_(self.spec.to_expression())

```

---

## 🔹 `repository.py` — реализация паттерна Repository

```python
from models import User, db

# Репозиторий инкапсулирует логику доступа к данным
class UserRepository:
    def __init__(self, session):
        self.session = session  # Обычно это db.session

    # Получить всех пользователей
    def get_all(self):
        return self.session.query(User).all()

    # Найти пользователей по спецификации
    def find_by_specification(self, specification):
        # Преобразуем спецификацию в SQLAlchemy выражение
        expr = specification.to_expression()

        # Применяем фильтр к запросу
        return self.session.query(User).filter(expr).all()

```

---

## 🔹 `app.py` — точка входа и Flask API

```python
from flask import Flask, jsonify
from models import db, User
from repository import UserRepository
from specifications import AgeGreaterThan, CountryIs

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///users.db"  # SQLite БД
db.init_app(app)  # Инициализация SQLAlchemy с Flask приложением

# Создание таблиц и добавление данных перед первым запросом
@app.before_first_request
def setup_db():
    db.create_all()  # Создаём таблицы

    # Если таблица пустая, добавим несколько пользователей
    if not User.query.first():
        db.session.add_all([
            User(name="Alice", age=30, country="USA"),
            User(name="Bob", age=22, country="Canada"),
            User(name="Charlie", age=28, country="USA"),
            User(name="David", age=19, country="Mexico"),
        ])
        db.session.commit()

# Эндпоинт с фильтрацией пользователей по спецификациям
@app.route("/users/filtered")
def get_filtered_users():
    # Создаём репозиторий с текущей сессией
    repo = UserRepository(db.session)

    # Составляем спецификацию: старше 25 и из США
    spec = AgeGreaterThan(25) & CountryIs("USA")

    # Получаем пользователей, удовлетворяющих спецификации
    users = repo.find_by_specification(spec)

    # Возвращаем JSON-ответ
    return jsonify([
        {"id": u.id, "name": u.name, "age": u.age, "country": u.country}
        for u in users
    ])

# Запуск приложения
if __name__ == "__main__":
    app.run(debug=True)

```

---

## 🧪 Результат

Когда ты откроешь `http://127.0.0.1:5000/users/filtered`, ты получишь:

```json
[
  {
    "id": 1,
    "name": "Alice",
    "age": 30,
    "country": "USA"
  },
  {
    "id": 3,
    "name": "Charlie",
    "age": 28,
    "country": "USA"
  }
]
```

---

## ✅ Что мы получили

|Паттерн|Назначение|
|---|---|
|**Specification**|Инкапсуляция логики фильтрации в классы|
|**Repository**|Изоляция SQLAlchemy и интерфейс к данным|
|**Flask + SQLAlchemy**|REST API + работа с БД|
