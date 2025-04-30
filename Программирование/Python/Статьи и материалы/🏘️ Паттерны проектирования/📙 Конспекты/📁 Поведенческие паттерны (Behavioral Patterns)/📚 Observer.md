## Определение
Паттерн наблюдатель - поведенческий паттерн проектирования, который создает механизм подписки, позволяющий одним объектам следить и реагировать на события, происходящие в других объектах.

## Структура
- **Издатель (Subject)** - объект, который содержит состояние и уведомляет наблюдателей
- **Наблюдатель (Observer)** - интерфейс с методом обновления, который вызывается издателем
- **Конкретный наблюдатель (Concrete Observer)** - реализует интерфейс наблюдателя

## Пример реализации на Python

```python
from abc import ABC, abstractmethod
from typing import List

# Интерфейс наблюдателя
class Observer(ABC):
    @abstractmethod
    def update(self, temperature: float, humidity: float, pressure: float) -> None:
        pass

# Издатель (Субъект)
class WeatherData:
    def __init__(self):
        self._observers: List[Observer] = []
        self._temperature = 0
        self._humidity = 0
        self._pressure = 0
    
    # Регистрация наблюдателя
    def register_observer(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)
    
    # Удаление наблюдателя
    def remove_observer(self, observer: Observer) -> None:
        self._observers.remove(observer)
    
    # Уведомление всех наблюдателей
    def notify_observers(self) -> None:
        for observer in self._observers:
            observer.update(self._temperature, self._humidity, self._pressure)
    
    # Изменение данных запускает оповещение
    def set_measurements(self, temperature: float, humidity: float, pressure: float) -> None:
        self._temperature = temperature
        self._humidity = humidity
        self._pressure = pressure
        self.notify_observers()

# Конкретный наблюдатель - текущие условия
class CurrentConditionsDisplay(Observer):
    def __init__(self, weather_data: WeatherData):
        self._temperature = 0
        self._humidity = 0
        weather_data.register_observer(self)
    
    def update(self, temperature: float, humidity: float, pressure: float) -> None:
        self._temperature = temperature
        self._humidity = humidity
        self.display()
    
    def display(self) -> None:
        print(f"Текущие условия: {self._temperature}°C и {self._humidity}% влажности")

# Конкретный наблюдатель - статистика
class StatisticsDisplay(Observer):
    def __init__(self, weather_data: WeatherData):
        self._temperatures = []
        weather_data.register_observer(self)
    
    def update(self, temperature: float, humidity: float, pressure: float) -> None:
        self._temperatures.append(temperature)
        self.display()
    
    def display(self) -> None:
        avg = sum(self._temperatures) / len(self._temperatures)
        print(f"Средняя/Макс/Мин температура: {avg:.1f}/{max(self._temperatures)}/{min(self._temperatures)}")

# Использование
weather_data = WeatherData()
current_display = CurrentConditionsDisplay(weather_data)
statistics_display = StatisticsDisplay(weather_data)

# Изменение данных приводит к уведомлению всех наблюдателей
weather_data.set_measurements(27, 65, 1010)
weather_data.set_measurements(28, 70, 1012)
```

## Преимущества
- Слабая связанность между издателем и наблюдателями
- Возможность добавлять новых наблюдателей без изменения издателя
- Издатель не зависит от конкретных классов наблюдателей

## Недостатки
- Наблюдатели уведомляются в случайном порядке
- Утечки памяти при неаккуратном управлении подписками
- Неожиданные обновления при сложной зависимости между наблюдателями

## Применение
- Графические интерфейсы (обработка событий)
- Системы подписок и уведомлений
- Шаблон MVC (Модель является Издателем, Представления - Наблюдателями)
