
---

## 🚲 Простой пример: Транспорт

---

### 💡 Цель:

Есть разные типы транспорта (машина, велосипед). Мы хотим вызывать фабрику, которая создаёт транспорт, но не хотим знать, **какой именно класс транспорта будет использоваться**.

---

### 1. Абстрактный транспорт

```python
from abc import ABC, abstractmethod

# Абстрактный класс "Транспорт"
class Transport(ABC):
    @abstractmethod
    def move(self) -> None:
        """Метод движения"""
        pass
```

---

### 2. Конкретные классы: Машина и Велосипед

```python
# Конкретный класс — Машина
class Car(Transport):
    def move(self) -> None:
        print("Машина едет по дороге.")

# Конкретный класс — Велосипед
class Bicycle(Transport):
    def move(self) -> None:
        print("Велосипед едет по велосипедной дорожке.")
```

---

### 3. Абстрактная фабрика

```python
# Абстрактная фабрика для создания транспорта
class TransportFactory(ABC):
    @abstractmethod
    def create_transport(self) -> Transport:
        """Создаёт объект транспорта"""
        pass
```

---

### 4. Конкретные фабрики

```python
# Фабрика, создающая машины
class CarFactory(TransportFactory):
    def create_transport(self) -> Transport:
        return Car()

# Фабрика, создающая велосипеды
class BicycleFactory(TransportFactory):
    def create_transport(self) -> Transport:
        return Bicycle()
```

---

### 5. Клиентский код

```python
# Клиентская функция, которая не знает, какой именно транспорт используется
def travel(factory: TransportFactory) -> None:
    transport = factory.create_transport()
    transport.move()
```

---

### 6. Пример запуска

```python
if __name__ == "__main__":
    print("Поехали на машине:")
    travel(CarFactory())  # Выведет: Машина едет по дороге.

    print("Поехали на велосипеде:")
    travel(BicycleFactory())  # Выведет: Велосипед едет по велосипедной дорожке.
```

---

## ✅ Что ты получаешь?

📌 **Factory Method** позволяет:

- Упростить создание объектов;
    
- Отделить логику создания от использования;
    
- Упростить расширение (например, можно легко добавить `ScooterFactory`).
    

---

## 📌 Кратко

|Часть|Назначение|
|---|---|
|`Transport`|Интерфейс транспорта|
|`Car`, `Bicycle`|Конкретные реализации|
|`TransportFactory`|Абстрактная фабрика|
|`CarFactory`, `BicycleFactory`|Конкретные фабрики|
|`travel()`|Клиентский код, вызывающий фабрику и не знающий деталей|

---

Хочешь — могу сделать этот пример с вводом от пользователя или с регистрацией всех фабрик через словарь, как DI (в стиле Flask/DRF).