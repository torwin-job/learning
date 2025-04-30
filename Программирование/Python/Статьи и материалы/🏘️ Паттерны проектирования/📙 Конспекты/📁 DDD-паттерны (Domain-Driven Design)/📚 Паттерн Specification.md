## 🔷 Что такое паттерн "Спецификация"?

**Specification (Спецификация)** — поведенческий шаблон проектирования, который позволяет инкапсулировать бизнес-логику проверки объектов на соответствие определённым условиям. Паттерн делает эти условия переиспользуемыми, комбинируемыми и легко тестируемыми.

---

## ✅ Когда использовать

- У вас есть сложные условия фильтрации/валидации объектов.
    
- Условия часто изменяются.
    
- Условия должны быть **переиспользуемыми и комбинируемыми** (например, с логикой `AND`, `OR`, `NOT`).
    

---

## 📦 Структура паттерна

1. **Specification (интерфейс/абстракция)** — определяет метод `is_satisfied_by(obj)`.
    
2. **ConcreteSpecification** — конкретная реализация.
    
3. **CombinatorSpecification** — логические комбинации: `AndSpecification`, `OrSpecification`, `NotSpecification`.
    

---

## 🔨 Пример на Python (фильтрация пользователей)

### 📁 Модель пользователя:

```python
class User:
    def __init__(self, name: str, age: int, country: str):
        self.name = name
        self.age = age
        self.country = country
```

---

### 📁 Базовая спецификация:

```python
from abc import ABC, abstractmethod

class Specification(ABC):
    @abstractmethod
    def is_satisfied_by(self, candidate) -> bool:
        pass

    # Комбинирующие методы
    def __and__(self, other):
        return AndSpecification(self, other)

    def __or__(self, other):
        return OrSpecification(self, other)

    def __invert__(self):
        return NotSpecification(self)
```

---

### 📁 Конкретные спецификации:

```python
class AgeGreaterThan(Specification):
    def __init__(self, age):
        self.age = age

    def is_satisfied_by(self, candidate):
        return candidate.age > self.age


class CountryIs(Specification):
    def __init__(self, country):
        self.country = country

    def is_satisfied_by(self, candidate):
        return candidate.country.lower() == self.country.lower()
```

---

### 📁 Комбинаторы:

```python
class AndSpecification(Specification):
    def __init__(self, *specs):
        self.specs = specs

    def is_satisfied_by(self, candidate):
        return all(spec.is_satisfied_by(candidate) for spec in self.specs)


class OrSpecification(Specification):
    def __init__(self, *specs):
        self.specs = specs

    def is_satisfied_by(self, candidate):
        return any(spec.is_satisfied_by(candidate) for spec in self.specs)


class NotSpecification(Specification):
    def __init__(self, spec):
        self.spec = spec

    def is_satisfied_by(self, candidate):
        return not self.spec.is_satisfied_by(candidate)
```

---

## 🧪 Использование

```python
users = [
    User("Alice", 30, "USA"),
    User("Bob", 22, "Canada"),
    User("Charlie", 28, "USA"),
    User("David", 19, "Mexico"),
]

# Создаём спецификацию: старше 25 и из США
spec = AgeGreaterThan(25) & CountryIs("USA")

# Фильтруем
filtered_users = list(filter(spec.is_satisfied_by, users))

for user in filtered_users:
    print(user.name)
```

**Вывод:**

```
Alice
Charlie
```

---

## ➕ Дополнительно: Преимущества

- ✔ Повторное использование логики.
    
- ✔ Чистая архитектура: бизнес-логика отделена от модели.
    
- ✔ Поддержка логических комбинаций (`&`, `|`, `~`).
    

---

## 🧠 Продвинутый пример: Валидация заказа

```python
class Order:
    def __init__(self, amount, is_paid, customer_type):
        self.amount = amount
        self.is_paid = is_paid
        self.customer_type = customer_type

class PaidSpecification(Specification):
    def is_satisfied_by(self, order):
        return order.is_paid

class VIPCustomerSpecification(Specification):
    def is_satisfied_by(self, order):
        return order.customer_type == "VIP"

class LargeAmountSpecification(Specification):
    def is_satisfied_by(self, order):
        return order.amount >= 1000

# Проверка: заказ должен быть оплачен и (VIP или большой)
spec = PaidSpecification() & (VIPCustomerSpecification() | LargeAmountSpecification())
```

---

## 🧹 Итого

|Плюсы|Минусы|
|---|---|
|✅ Читабельность условий|🐌 Может быть избыточным для простых проверок|
|✅ Гибкость и расширяемость|🔧 Нужно писать много кода для каждого условия|
|✅ Легко комбинируется|—|

---

Если хочешь, могу сделать мини-фреймворк по этому паттерну или адаптировать под твой проект (Django, FastAPI, CLI и т.д.).