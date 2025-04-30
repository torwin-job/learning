## 🔹 Что такое Entity?

**Entity (Сущность)** — это **объект бизнес-домена**, обладающий:

- уникальным идентификатором (обычно `id`)
    
- **устойчивой идентичностью** во времени, независимо от изменений свойств
    
- бизнес-значимым поведением
    

> В отличие от Value Object, Entity важна **сама по себе**, а не только по значению.

---

## 🔹 Примеры из жизни

|Пример|Entity?|Почему?|
|---|---|---|
|Пользователь|✅|Уникальный `id`, может менять имя, email|
|Товар|✅|Может меняться цена, описание и т.д.|
|Деньги 💵|❌|Это Value Object — важна сумма, а не объект|
|Координаты|❌|Важны значения, а не их "личность"|

---

## 🔹 Отличие от других паттернов

|Тип|Имеет ID|Равенство по ID|Пример|
|---|---|---|---|
|**Entity**|✅|По ID|Пользователь, Заказ|
|**ValueObject**|❌|По значениям|Деньги, Адрес, Координаты|
|**Aggregate**|✅|Управляет группой Entity/ValueObject|Корзина покупок|

---

## 🔹 Основные свойства Entity

- Уникальный идентификатор (`id`)
    
- Методы, описывающие **поведение**, а не только данные
    
- Может включать Value Object'ы
    
- Является частью доменной модели
    
- Используется внутри Aggregate
    

---

## 🔹 Entity в DDD (Domain-Driven Design)

DDD рекомендует:

- Entity — **ядро доменной логики**
    
- Разрабатывается вместе с **бизнес-экспертами**
    
- Не зависит от инфраструктуры (ORM, БД)
    

---

## 🔹 Пример на Python (базовый)

```python
import uuid

class User:
    def __init__(self, name: str, email: str, id: uuid.UUID = None):
        self.id = id or uuid.uuid4()
        self.name = name
        self.email = email

    def change_email(self, new_email: str):
        # Бизнес-правило: нельзя менять email на корпоративный
        if new_email.endswith('@company.com'):
            raise ValueError("Недопустимый email")
        self.email = new_email

    def __eq__(self, other):
        return isinstance(other, User) and self.id == other.id

    def __hash__(self):
        return hash(self.id)
```

---

## 🔹 Пример с SQLAlchemy ORM

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    email = Column(String)

    def change_email(self, new_email: str):
        if new_email.endswith('@company.com'):
            raise ValueError("Нельзя такой email")
        self.email = new_email
```

---

## 🔹 Интеграция с Value Object

```python
class Email:
    def __init__(self, value: str):
        if '@' not in value:
            raise ValueError("Некорректный email")
        self.value = value

    def __eq__(self, other):
        return isinstance(other, Email) and self.value == other.value

class User:
    def __init__(self, id, name, email: Email):
        self.id = id
        self.name = name
        self.email = email
```

---

## 🔹 Пример в доменной архитектуре

```
project/
├── domain/
│   ├── entities/
│   │   └── user.py       # здесь Entity
│   ├── value_objects/
│   │   └── email.py      # здесь Email (VO)
│   └── services/         # бизнес-логика
│
├── infrastructure/
│   ├── orm/
│   │   └── mappings.py   # SQLAlchemy модель
│   └── repositories/
```

---

## 🔹 Преимущества

✅ Явно выраженная бизнес-сущность  
✅ Устойчивость к изменениям структуры БД  
✅ Тестируемость  
✅ Простота миграции между хранилищами (если использовать UoW + Repository)

---

## 🔹 Недостатки

⚠️ Требует дисциплины в проектировании  
⚠️ Может дублировать логику с ORM  
⚠️ Не всегда нужно — для CRUD можно обойтись DTO/ORM

---

## 🔹 Важные практики

- Не смешивать ORM и поведение в одной модели (по возможности)
    
- Изолировать Entity в `domain/`
    
- Делать Entity **"чистой" от инфраструктуры**
    

---

## 🔹 Подводим итог

|Свойство|Entity|
|---|---|
|Идентификатор|✅ Да (`id`)|
|Важность состояния|❌ (состояние может меняться)|
|Равенство|По `id`, а не по содержимому|
|Поведение|Да, содержит бизнес-методы|
|Используется в|Domain, Service, Aggregate|

---

## 🔹 Хочешь углубиться?

Могу также показать:

- отличие от **DTO, Aggregate, Value Object**
    
- Entity без ORM (для unit-тестов)
    
- Entity в стиле `dataclass + @property`
    

Напиши, если интересует архитектура под DDD, и я соберу пример проекта с Entity + ValueObject + Repository + UoW.

Готов продолжать 💻