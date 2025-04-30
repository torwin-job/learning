## 🔹 Что такое Domain Service?

**Domain Service** — это объект, инкапсулирующий **бизнес-логику, не принадлежащую ни одной конкретной сущности (Entity)**.

Он **работает с Entity и Value Object'ами**, обеспечивая реализацию бизнес-правил, **где нет "хозяина" этой логики**.

---

## 🔹 Когда использовать?

Используй `Domain Service`, когда:

- Логика не может быть естественно привязана к одной Entity
    
- Требуется **координация нескольких сущностей**
    
- Поведение **нельзя поместить внутрь одной сущности**
    

---

## 🔹 Отличие от других компонентов

|Компонент|Содержит состояние|Бизнес-логика|Пример|
|---|---|---|---|
|**Entity**|✅ Да (id, поля)|✅ Да|`User.change_email()`|
|**Value Object**|❌ Нет|✅ Да|`Money.add()`|
|**Domain Service**|❌ Нет|✅ Да|`TransferMoneyService.transfer(from, to)`|
|**Application Service**|❌|🚫 (координация слоёв)|`handle_create_user()`|

---

## 🔹 Пример из жизни

**Банковская система**:

- `Account` — Entity
    
- `Money` — Value Object
    
- `TransferService` — Domain Service, потому что перевод затрагивает 2 аккаунта
    

---

## 🔹 Структура Domain Service

```python
class DomainService:
    def метод(self, сущность, параметры):
        # логика, затрагивающая одну или более сущностей
```

---

## 🔹 Простой пример на Python

### Сущности:

```python
class Account:
    def __init__(self, id, balance):
        self.id = id
        self.balance = balance

    def withdraw(self, amount):
        if self.balance < amount:
            raise ValueError("Недостаточно средств")
        self.balance -= amount

    def deposit(self, amount):
        self.balance += amount
```

### Доменный сервис:

```python
class MoneyTransferService:
    def transfer(self, from_account: Account, to_account: Account, amount: float):
        from_account.withdraw(amount)
        to_account.deposit(amount)
```

---

## 🔹 DomainService vs ApplicationService

|Особенность|Domain Service|Application Service|
|---|---|---|
|Отвечает за|Чистая бизнес-логика|Координация слоёв приложения|
|Зависит от Entity/VO|✅ Да|❌ Нет|
|Использует репозитории|❌ (обычно не напрямую)|✅ Да|
|Взаимодействует с UoW|❌ Нет|✅ Да|
|Пример|`TransferService.transfer()`|`CreateUserHandler.handle()`|

---

## 🔹 Где размещать Domain Services?

В DDD-структуре обычно:

```
project/
├── domain/
│   ├── services/
│   │   └── money_transfer.py
│   ├── entities/
│   ├── value_objects/
```

---

## 🔹 Расширенный пример с UoW

```python
class MoneyTransferService:
    def __init__(self, uow):
        self.uow = uow

    def transfer(self, from_id, to_id, amount):
        with self.uow as u:
            from_account = u.accounts.get(from_id)
            to_account = u.accounts.get(to_id)

            from_account.withdraw(amount)
            to_account.deposit(amount)

            u.commit()
```

---

## 🔹 Преимущества

✅ Централизация бизнес-правил  
✅ Отделение логики от инфраструктуры  
✅ Повторное использование  
✅ Высокая тестируемость  
✅ Хорошо сочетается с Unit of Work и Repository

---

## 🔹 Недостатки

⚠️ Перенос логики "в сторону" может усложнить поиск  
⚠️ Часто путается с Application Service  
⚠️ Может дублировать Entity-логику, если проект спроектирован неаккуратно

---

## 🔹 Практики и советы

- Domain Service **должен быть без состояния** (stateless)
    
- Не должен напрямую вызывать инфраструктуру (например, БД)
    
- Принимает Entity и VO как аргументы, возвращает результат
    
- Чёткое имя по бизнес-логике: `TransferMoneyService`, `BookingService`, `BillingService`
    

---

## 🔹 Итог

|Аспект|Значение|
|---|---|
|Цель|Инкапсуляция бизнес-правил, не принадлежащих Entity|
|Использует|Entity, Value Object|
|Связан с|Domain Layer|
|Формат|Класс или функция без состояния|
|Инфраструктура?|❌ Не использует напрямую|

---

## 🔹 Бонус: Тестирование Domain Service

```python
def test_transfer_success():
    from_acc = Account(id=1, balance=100)
    to_acc = Account(id=2, balance=50)

    service = MoneyTransferService()
    service.transfer(from_acc, to_acc, 30)

    assert from_acc.balance == 70
    assert to_acc.balance == 80
```

---

Если хочешь — могу:

- Привести примеры для твоего реального проекта (напр. учёт личного состава, финансы, сигналы и т.п.)
    
- Показать **структуру проекта с Entity + VO + DomainService**
    
- Интеграцию с FastAPI, Flask или CLI
    

Напиши, если интересен следующий паттерн или нужна практика.