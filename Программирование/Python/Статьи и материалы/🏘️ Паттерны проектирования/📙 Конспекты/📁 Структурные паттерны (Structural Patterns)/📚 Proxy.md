Вот пример паттерна **Proxy (Заместитель)** на Python.  
[[Пример Proxy на Flask]]
### **Сценарий:**  
Допустим, у нас есть сервис для загрузки видео (`VideoService`), но мы хотим:  
1. **Кешировать** уже загруженные видео, чтобы не грузить их повторно.  
2. **Контролировать доступ** (например, проверять, есть ли у пользователя права).  

Для этого создадим **Proxy**, который будет перехватывать вызовы к реальному сервису.  

---

### **Код:**  

#### **1. Интерфейс (или абстрактный класс) для сервиса и прокси**  
```python
from abc import ABC, abstractmethod

class VideoService(ABC):
    @abstractmethod
    def download_video(self, video_id: str) -> str:
        pass
```

#### **2. Реальный сервис (тяжёлый объект)**  
```python
class RealVideoService(VideoService):
    def download_video(self, video_id: str) -> str:
        print(f"Загрузка видео с ID {video_id} с сервера... (Это долгая операция)")
        return f"Данные видео {video_id}"
```

#### **3. Прокси (добавляет логику кеширования и контроля доступа)**  
```python
class VideoServiceProxy(VideoService):
    def __init__(self, real_service: VideoService):
        self._real_service = real_service
        self._cache = {}  # Кеш загруженных видео

    def download_video(self, video_id: str) -> str:
        # Проверка доступа (можно добавить любую логику)
        if not self._check_access():
            raise PermissionError("Доступ запрещён!")

        # Проверка кеша
        if video_id in self._cache:
            print(f"Видео {video_id} взято из кеша!")
            return self._cache[video_id]

        # Если нет в кеше — загружаем и сохраняем
        video_data = self._real_service.download_video(video_id)
        self._cache[video_id] = video_data
        return video_data

    def _check_access(self) -> bool:
        # Здесь может быть сложная логика (например, проверка подписки)
        return True  # Для примера всегда разрешаем
```

#### **4. Использование**  
```python
# Клиентский код
real_service = RealVideoService()
proxy = VideoServiceProxy(real_service)

# Первый вызов — загрузка с сервера
print(proxy.download_video("123"))  # Загрузка с сервера

# Повторный вызов — берётся из кеша
print(proxy.download_video("123"))  # Взято из кеша!

# Попытка без доступа (если бы _check_access() вернул False)
# print(proxy.download_video("456"))  # PermissionError
```

---

### **Вывод:**  
- **Proxy** перехватывает вызовы к `RealVideoService` и добавляет:  
  - **Кеширование** (экономит время и ресурсы).  
  - **Контроль доступа** (безопасность).  
- Клиент работает с `Proxy`, не зная о деталях реального сервиса.  

Это лишь один из вариантов использования **Proxy**. Другие примеры:  
- **Ленивая загрузка** (например, для больших объектов).  
- **Логирование** вызовов методов.  
- **Защита API** (например, проверка токенов).  

Если нужно что-то уточнить — спрашивай! 😊