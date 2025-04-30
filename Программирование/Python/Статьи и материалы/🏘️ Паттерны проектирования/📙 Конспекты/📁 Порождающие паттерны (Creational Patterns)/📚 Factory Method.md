Вот подробный конспект по паттерну **Factory Method (Фабричный метод)** на Python 3 — с объяснением, примерами и типичным применением.

---

## 🔧 Что такое Factory Method?

**Factory Method (Фабричный метод)** — это порождающий паттерн проектирования, который определяет интерфейс создания объекта, но позволяет подклассам изменять тип создаваемого объекта.

Он **делегирует создание объектов подклассам**, не указывая конкретный класс создаваемого объекта.

[[Простой пример Factory Method]]
[[Конкретный пример Factory Method]]
[[пример использования Factory Method в FastAPI]]

## 📦 Зачем нужен?

- Избавляет от привязки к конкретным классам.
    
- Позволяет использовать **наследование** для управления созданием объектов.
    
- Упрощает расширение кода при добавлении новых типов объектов.
    

---

## 📐 Структура паттерна:

```text
             ┌────────────────┐
             │   Creator      │◄────────────────────┐
             │ (абстрактный)  │                     │
             └────────────────┘                     │
                    ▲                               │
                    │                               │
      ┌─────────────────────────┐                   │
      │ ConcreteCreator         │                   │
      │ (реализация фабрики)    │                   │
      └─────────────────────────┘                   │
                    │                               │
                    ▼                               ▼
            ┌────────────────┐           ┌────────────────┐
            │ Product        │◄──────────│ ConcreteProduct│
            │ (интерфейс)    │           └────────────────┘
            └────────────────┘
```

---

## ✅ Пример (на Python 3)

### Абстрактный продукт:

```python
from abc import ABC, abstractmethod

class Product(ABC):
    @abstractmethod
    def operation(self) -> str:
        pass
```

### Конкретные продукты:

```python
class ConcreteProductA(Product):
    def operation(self) -> str:
        return "Результат: Продукт A"

class ConcreteProductB(Product):
    def operation(self) -> str:
        return "Результат: Продукт B"
```

### Абстрактный создатель:

```python
class Creator(ABC):
    @abstractmethod
    def factory_method(self) -> Product:
        pass

    def some_operation(self) -> str:
        # Используем фабричный метод
        product = self.factory_method()
        return f"Creator: работаю с {product.operation()}"
```

### Конкретные создатели:

```python
class ConcreteCreatorA(Creator):
    def factory_method(self) -> Product:
        return ConcreteProductA()

class ConcreteCreatorB(Creator):
    def factory_method(self) -> Product:
        return ConcreteProductB()
```

---

## 💡 Использование

```python
def client_code(creator: Creator) -> None:
    print(creator.some_operation())

client_code(ConcreteCreatorA())
client_code(ConcreteCreatorB())
```

**Вывод:**

```
Creator: работаю с Результат: Продукт A
Creator: работаю с Результат: Продукт B
```

---

## 🧠 Когда применять?

- Когда у тебя есть **множество похожих объектов**, но ты не хочешь жёстко привязываться к их конкретным классам.
    
- Когда объект создаётся **динамически**, а не жёстко вшит в код.
    
- Когда нужно предоставить **расширяемость** без изменения существующего кода.
    

---

## 🆚 Отличие от Abstract Factory

|Паттерн|Описание|
|---|---|
|Factory Method|Создаёт **один** продукт|
|Abstract Factory|Создаёт **группу** связанных продуктов|

---

## 🧪 Тестирование фабричных методов

```python
def test_creator():
    creator = ConcreteCreatorA()
    assert creator.some_operation() == "Creator: работаю с Результат: Продукт A"

    creator = ConcreteCreatorB()
    assert creator.some_operation() == "Creator: работаю с Результат: Продукт B"

test_creator()
```

---

## 📌 Кратко

- Паттерн: **Factory Method**
    
- Тип: **Порождающий**
    
- Проблема: нужно создавать объект, не зная его точного класса
    
- Решение: делегировать создание объекта через **фабричный метод**
    

---

Если хочешь, могу дополнительно показать пример из реальной жизни — например, для создания обработчиков сообщений, контроллеров, или API-интерфейсов.