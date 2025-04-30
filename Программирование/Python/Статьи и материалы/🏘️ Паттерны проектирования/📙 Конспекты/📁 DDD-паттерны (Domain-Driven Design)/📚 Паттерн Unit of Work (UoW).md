## 🔹 Что это?

**Unit of Work (UoW)** — это паттерн, который **собирает изменения в бизнес-объектах**, отслеживает их и **координирует коммит/откат** этих изменений как одной транзакции.

> Это как "корзина", в которую ты складываешь изменения, а потом одним действием — сохраняешь или откатываешь всё сразу.

---

## 🔹 Зачем нужен?

- Управление **транзакциями**
    
- Координация **нескольких репозиториев**
    
- **Целостность данных** (все или ничего)
    
- Явное разделение начала и завершения работы с данными
    

---

## 🔹 Пример жизненного цикла

1. Создаём `UnitOfWork`
    
2. Получаем доступ к репозиториям через UoW
    
3. Выполняем изменения
    
4. Вызываем `commit()` или `rollback()`
    

---

## 🔹 Простая структура UoW

```python
class UnitOfWork:
    def __init__(self, session_factory):
        self.session_factory = session_factory

    def __enter__(self):
        self.session = self.session_factory()
        return self

    def __exit__(self, *args):
        self.session.close()

    def commit(self):
        self.session.commit()

    def rollback(self):
        self.session.rollback()
```

---

## 🔹 Связь с репозиториями

Репозитории используют сессию UoW:

```python
class UserRepository:
    def __init__(self, session):
        self.session = session

    def add(self, user):
        self.session.add(user)
```

---

## 🔹 Полный пример с SQLAlchemy

### 1. `models.py`

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
```

---

### 2. `repositories.py`

```python
from models import User

class UserRepository:
    def __init__(self, session):
        self.session = session

    def get_by_id(self, id_):
        return self.session.query(User).get(id_)

    def add(self, user):
        self.session.add(user)

    def list(self):
        return self.session.query(User).all()
```

---

### 3. `unit_of_work.py`

```python
from sqlalchemy.orm import sessionmaker
from repositories import UserRepository

class UnitOfWork:
    def __init__(self, session_factory: sessionmaker):
        self.session_factory = session_factory

    def __enter__(self):
        self.session = self.session_factory()
        self.users = UserRepository(self.session)  # Привязка репозитория
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.rollback()
        else:
            self.commit()
        self.session.close()

    def commit(self):
        self.session.commit()

    def rollback(self):
        self.session.rollback()
```

---

### 4. Использование в бизнес-логике

```python
def create_user(name: str, uow: UnitOfWork):
    with uow as u:
        user = User(name=name)
        u.users.add(user)
```

---

## 🔹 Интеграция с FastAPI

```python
from fastapi import FastAPI, Depends

app = FastAPI()

SessionFactory = sessionmaker(bind=engine)

@app.post("/users")
def create_user(name: str, uow: UnitOfWork = Depends(lambda: UnitOfWork(SessionFactory))):
    with uow:
        uow.users.add(User(name=name))
```

---

## 🔹 Преимущества

✅ Централизованное управление транзакцией  
✅ Согласованность данных  
✅ Легко масштабируется (добавь репозиторий — и всё работает)  
✅ Упрощает тестирование  
✅ Поддерживает паттерн DDD и Чистую архитектуру

---

## 🔹 Недостатки

⚠️ Немного "лишнего" кода в маленьких проектах  
⚠️ Требует понимания управления сессиями SQLAlchemy  
⚠️ Сложнее, если не использовать ORM

---

## 🔹 Когда использовать

- Микросервисы
    
- Чистая архитектура
    
- DDD (Domain-Driven Design)
    
- Несколько репозиториев в одной операции
    
- Необходимость в rollback/commit контроле
    

---

## 🔹 Расширения

- Внедрить асинхронный UoW (для `FastAPI` + `async SQLAlchemy`)
    
- Добавить логгирование в `commit`, `rollback`
    
- Связать с DI контейнером (например, `dependency_injector`)
    
- Ввести `EventCollector` внутри `UnitOfWork`
    

---

## 🔹 Итог

| Аспект         | Описание                                                 |
| -------------- | -------------------------------------------------------- |
| Назначение     | Управление транзакциями, агрегирование работы с БД       |
| Использование  | В паре с репозиториями для чистой архитектуры            |
| Реализация     | Через контекстный менеджер (`with`) и session SQLAlchemy |
| Часто вместе с | Repository, Specification, Entity, DomainService         |
