## 🔹 Что это?

**Паттерн "Репозиторий"** — это абстракция над механизмом хранения данных (БД, API, кэш и т.д.), которая изолирует бизнес-логику от конкретной технологии хранения.  
Позволяет работать с объектами как с коллекцией в памяти, скрывая детали доступа к данным.

Идет вместе с [[📚 Паттерн Specification]] и [[📚 Паттерн Unit of Work (UoW)]]

---

## 🔹 Цель

- **Инкапсуляция** логики доступа к данным
    
- **Изоляция** бизнес-логики от ORM (например, SQLAlchemy)
    
- Упрощение **тестирования** (можно мокать репозиторий)
    
- Создание **чистого интерфейса** для работы с сущностями
    

---

## 🔹 Пример использования

Без репозитория:

```python
user = db.session.query(User).filter(User.id == 1).first()
```

С репозиторием:

```python
user = user_repository.get_by_id(1)
```

---

## 🔹 Компоненты

- **Entity**: бизнес-сущность (например, `User`)
    
- **Repository**: класс, предоставляющий интерфейс доступа к сущности
    
- **Storage**: конкретный механизм хранения (ORM, raw SQL, API)
    

---

## 🔹 Интерфейс репозитория

Типичные методы:

```python
class UserRepository:
    def get_by_id(self, id: int) -> User: ...
    def get_all(self) -> list[User]: ...
    def add(self, user: User): ...
    def delete(self, user: User): ...
    def update(self, user: User): ...
```

---

## 🔹 Пример с SQLAlchemy

### models.py

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    age = db.Column(db.Integer)
```

### repository.py

```python
from models import User, db

class UserRepository:
    def __init__(self, session):
        self.session = session

    def get_all(self):
        return self.session.query(User).all()

    def get_by_id(self, user_id):
        return self.session.query(User).filter_by(id=user_id).first()

    def add(self, user):
        self.session.add(user)
        self.session.commit()

    def update(self):
        self.session.commit()

    def delete(self, user):
        self.session.delete(user)
        self.session.commit()
```

---

## 🔹 Общий базовый репозиторий (Generic) [[📚 Generic Repository  Service Layer]]

Можно вынести общую логику:

```python
class BaseRepository:
    def __init__(self, session, model):
        self.session = session
        self.model = model

    def get_by_id(self, id_):
        return self.session.query(self.model).get(id_)

    def get_all(self):
        return self.session.query(self.model).all()

    def add(self, obj):
        self.session.add(obj)
        self.session.commit()

    def delete(self, obj):
        self.session.delete(obj)
        self.session.commit()

    def update(self):
        self.session.commit()
```

Использование:

```python
user_repo = BaseRepository(session=db.session, model=User)
user_repo.get_all()
```

---

## 🔹 Интеграция со Specification (пример)

```python
def find_by_spec(self, spec):
    return self.session.query(self.model).filter(spec.to_expression()).all()
```

---

## 🔹 Преимущества

✅ Изоляция от SQLAlchemy/ORM  
✅ Упрощённое тестирование (можно заменить моками)  
✅ Более чистая и стабильная архитектура  
✅ Упрощение unit-тестов бизнес-логики  
✅ Возможность кеширования/логгирования/валидации

---

## 🔹 Недостатки

⚠️ Может показаться "избыточным" в маленьких проектах  
⚠️ Требует дисциплины в архитектуре  
⚠️ Повторение CRUD, если не использовать `BaseRepository`

---

## 🔹 Где часто применяется

- **Django, Flask, FastAPI**
    
- **Domain-Driven Design (DDD)**
    
- Микросервисы
    
- Чистая архитектура
    
- Когда нужно быстро заменить ORM или источник данных
    

---

## 🔹 Как тестировать?

```python
def test_get_by_id():
    repo = UserRepository(session=fake_session)
    user = repo.get_by_id(1)
    assert user.name == "Alice"
```

А в бизнес-логике можно использовать `MockRepository`, не подключая БД.

---

## 🔹 Резюме

|Характеристика|Описание|
|---|---|
|Назначение|Отделить бизнес-логику от хранения данных|
|Объекты|Репозиторий, модель, сессия (БД)|
|Плюсы|Тестируемость, читаемость, расширяемость|
|Минусы|Могут казаться "лишними" при простых сценариях|
|Отлично сочетается с|Unit of Work, Specification, DDD|

---
