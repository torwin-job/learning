## 🔧 **Пример с комментариями** — Логгеры (Консольный и Файловый)

---

### 1. Абстрактный продукт — `Logger`

```python
from abc import ABC, abstractmethod

# Абстрактный класс логгера: определяет интерфейс всех логгеров
class Logger(ABC):

    @abstractmethod
    def log(self, message: str) -> None:
        """Метод логирования — должен быть реализован в конкретном логгере"""
        pass
```

---

### 2. Конкретные продукты — `ConsoleLogger` и `FileLogger`

```python
# Логгер, выводящий сообщение в консоль
class ConsoleLogger(Logger):
    def log(self, message: str) -> None:
        print(f"[Console] {message}")


# Логгер, записывающий сообщение в файл
class FileLogger(Logger):
    def __init__(self, filename: str = "log.txt"):
        self.filename = filename

    def log(self, message: str) -> None:
        with open(self.filename, "a") as f:
            f.write(f"[File] {message}\n")
```

---

### 3. Абстрактный создатель — `LoggerCreator`

```python
# Абстрактный создатель, определяет фабричный метод create_logger
class LoggerCreator(ABC):

    @abstractmethod
    def create_logger(self) -> Logger:
        """Фабричный метод: должен вернуть объект, реализующий интерфейс Logger"""
        pass

    def write_log(self, msg: str) -> None:
        """
        Общая логика для всех создателей.
        Получает логгер через фабричный метод и вызывает у него log().
        """
        logger = self.create_logger()
        logger.log(msg)
```

---

### 4. Конкретные создатели — `ConsoleLoggerCreator`, `FileLoggerCreator`

```python
# Конкретный создатель для ConsoleLogger
class ConsoleLoggerCreator(LoggerCreator):
    def create_logger(self) -> Logger:
        # Возвращает конкретный объект ConsoleLogger
        return ConsoleLogger()


# Конкретный создатель для FileLogger
class FileLoggerCreator(LoggerCreator):
    def __init__(self, filename: str = "log.txt"):
        self.filename = filename

    def create_logger(self) -> Logger:
        # Возвращает объект FileLogger с нужным файлом
        return FileLogger(self.filename)
```

---

### 5. Клиентский код

```python
def client_code(creator: LoggerCreator) -> None:
    """
    Функция, которая использует логгер, но не знает, какой конкретно.
    Это демонстрирует слабую связанность между клиентом и реализациями.
    """
    print("Клиент: работаю с логгером, не зная его типа.")
    creator.write_log("Это тестовое сообщение")
```

---

### 6. Пример использования (точка входа)

```python
if __name__ == "__main__":
    # Используем консольный логгер
    console_creator = ConsoleLoggerCreator()
    client_code(console_creator)

    # Используем файловый логгер
    file_creator = FileLoggerCreator("events.log")
    client_code(file_creator)
```

---

## 🧪 Результат

✅ В консоли будет:

```
Клиент: работаю с логгером, не зная его типа.
[Console] Это тестовое сообщение
Клиент: работаю с логгером, не зная его типа.
```

✅ В файле `events.log` появится:

```
[File] Это тестовое сообщение
```

---

## 🔄 Что здесь происходит?

|Компонент|Объяснение|
|---|---|
|`Logger`|Абстрактный интерфейс логгера|
|`ConsoleLogger`, `FileLogger`|Реальные классы логгирования|
|`LoggerCreator`|Фабрика, знает **как** создавать логгер, но не **какой**|
|`ConsoleLoggerCreator`, `FileLoggerCreator`|Конкретные фабрики, создающие нужный логгер|
|`client_code()`|Использует `.write_log()`, не зная реализацию логгера|

---

Если хочешь — могу сделать этот пример более реальным: например, с логгированием по уровням (INFO, ERROR), добавлением асинхронности или использованием паттерна Singleton для логгера.