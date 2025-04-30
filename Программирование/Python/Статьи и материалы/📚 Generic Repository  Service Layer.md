Ты описываешь довольно распространённую проблему — **повторяющаяся логика доступа к данным** и работы с ними. Подход, который ты ищешь, называется:

---

### ✅ **Generic Repository / Service Layer**

или

### ✅ **Repository + Service + Unit of Work Pattern**

(в DDD это часто называют просто **"Application Layer [[📚 Слоистая архетектура]] "** или **"Use Case Layer"**)

---

## 💡 Идея:

Создать **универсальный слой**, который можно использовать **с разными моделями**, чтобы не дублировать код в каждом сервисе/репозитории.

---

## 🔧 Пример архитектуры:

```
project/
│
├── models/
│   ├── user.py
│   └── signal.py
│
├── repositories/
│   ├── base.py        ← GenericRepository
│   ├── user.py        ← Наследует base
│   └── signal.py
│
├── services/
│   ├── base.py        ← GenericService (или UseCase)
│   ├── user.py
│   └── signal.py
│
└── db/
    └── session.py     ← сессия SQLAlchemy
```

---

## 🧱 1. Базовый репозиторий (`repositories/base.py`)

```python
from sqlalchemy.orm import Session
from typing import Type, TypeVar, Generic

T = TypeVar("T")

class BaseRepository(Generic[T]):
    def __init__(self, model: Type[T], db: Session):
        self.model = model
        self.db = db

    def get(self, id: int) -> T:
        return self.db.query(self.model).get(id)

    def get_all(self) -> list[T]:
        return self.db.query(self.model).all()

    def create(self, obj_in: dict) -> T:
        obj = self.model(**obj_in)
        self.db.add(obj)
        self.db.commit()
        self.db.refresh(obj)
        return obj

    def delete(self, id: int):
        obj = self.get(id)
        self.db.delete(obj)
        self.db.commit()
```

---

## 🧱 2. Репозиторий для User (`repositories/user.py`)

```python
from .base import BaseRepository
from models.user import User

class UserRepository(BaseRepository):
    def __init__(self, db):
        super().__init__(User, db)
```

Аналогично для Signal.

---

## 🧱 3. Универсальный сервис (`services/base.py`)

```python
class BaseService:
    def __init__(self, repo):
        self.repo = repo

    def get_all(self):
        return self.repo.get_all()

    def get_one(self, id):
        return self.repo.get(id)

    def create(self, data):
        return self.repo.create(data)
```

---

## 🔁 Итог:

- **Повторная логика** сосредоточена в базовых классах.
    
- Ты можешь использовать универсальные методы везде, где нужно.
    
- Расширение логики (например, фильтрация, проверка прав) происходит в конкретных сервисах.
    

---

## ☑️ Альтернативы:

- **SQLAlchemy Declarative Mixins** – если хочешь общую логику в самих моделях.
    
- **ORM wrappers** как `SQLModel`, `Tortoise`, `PonyORM` иногда делают это из коробки.
    
- В **FastAPI** можно обернуть в Depends + Generic CRUD и получить чистую DI архитектуру.
    

---
