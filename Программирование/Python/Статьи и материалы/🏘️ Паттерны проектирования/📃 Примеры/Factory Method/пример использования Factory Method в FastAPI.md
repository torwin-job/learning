Отлично! Давай сделаем **реальный пример использования Factory Method в FastAPI**.

---

## 🎯 **Задача**: Отправка уведомлений (email, sms, telegram)

---

## 📦 Структура

```
project/
│
├── main.py
├── notifications/
│   ├── base.py
│   ├── email.py
│   ├── sms.py
│   ├── telegram.py
│   └── factory.py
```

---

## 🔹 1. `notifications/base.py` — базовый интерфейс

```python
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def send(self, message: str) -> str:
        """Метод отправки уведомлений"""
        pass
```

---

## 🔹 2. `notifications/email.py`

```python
from .base import Notifier

class EmailNotifier(Notifier):
    def send(self, message: str) -> str:
        # Симуляция отправки email
        return f"Email отправлен: {message}"
```

---

## 🔹 3. `notifications/sms.py`

```python
from .base import Notifier

class SMSNotifier(Notifier):
    def send(self, message: str) -> str:
        return f"SMS отправлено: {message}"
```

---

## 🔹 4. `notifications/telegram.py`

```python
from .base import Notifier

class TelegramNotifier(Notifier):
    def send(self, message: str) -> str:
        return f"Telegram сообщение отправлено: {message}"
```

---

## 🔹 5. `notifications/factory.py` — фабрика

```python
from .base import Notifier
from .email import EmailNotifier
from .sms import SMSNotifier
from .telegram import TelegramNotifier

def get_notifier(channel: str) -> Notifier:
    """Фабричный метод: возвращает нужный класс-уведомитель"""
    if channel == "email":
        return EmailNotifier()
    elif channel == "sms":
        return SMSNotifier()
    elif channel == "telegram":
        return TelegramNotifier()
    else:
        raise ValueError(f"Неизвестный канал: {channel}")
```

---

## 🔹 6. `main.py` — FastAPI приложение

```python
from fastapi import FastAPI, Query, HTTPException
from notifications.factory import get_notifier

app = FastAPI()

@app.get("/notify")
def notify(channel: str = Query(..., description="email | sms | telegram"),
           message: str = Query(...)):
    try:
        notifier = get_notifier(channel)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))

    result = notifier.send(message)
    return {"status": "success", "detail": result}
```

---

## 📦 Пример запроса:

```
GET /notify?channel=sms&message=Привет
```

📥 Ответ:

```json
{
  "status": "success",
  "detail": "SMS отправлено: Привет"
}
```

---

## ✅ Что здесь происходит?

|Компонент|Назначение|
|---|---|
|`Notifier`|Абстрактный интерфейс|
|`EmailNotifier` и др.|Реализация отправки в разные каналы|
|`get_notifier()`|**Factory Method**, выбирает нужную реализацию|
|`FastAPI`|Вызывает фабрику и работает через интерфейс|

---

## 📌 Плюсы такого подхода

- ➕ Легко добавлять новые каналы уведомлений без изменения логики в `main.py`.
    
- ✅ Уведомители изолированы, и каждый делает свою задачу.
    
- 🧪 Можно мокать любые `Notifier` в тестах.
    

---

Хочешь — могу расширить пример: добавить асинхронную отправку, логи, DI или отправку с подтверждением (OTP, кнопка в Telegram и т.д.).