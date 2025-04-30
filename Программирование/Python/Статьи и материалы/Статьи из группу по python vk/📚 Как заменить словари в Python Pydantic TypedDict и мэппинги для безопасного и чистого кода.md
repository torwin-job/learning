### **1. Введение**  
Автор рассказывает, что словари (`dict`) в Python — удобная структура данных, но в больших проектах их неконтролируемое использование приводит к ошибкам и сложностям в поддержке.  
Вместо них предлагаются три подхода:  
- **Pydantic** – для строгой валидации и сериализации данных.  
- **TypedDict** – для аннотации структуры словарей в легаси-коде.  
- **Мэппинги (`Mapping`, `MutableMapping`)** – для явного указания роли словарей как хранилищ ключ-значение.  

---  

### **2. Pydantic: мощная альтернатива классам данных**  
Pydantic – библиотека для проверки данных, автоматического преобразования типов и работы с моделями.  

#### **Пример модели `GitHubRepo`:**  
```python
from pydantic import BaseModel

class GitHubRepo(BaseModel):
    """GitHub repository."""
    owner: str
    name: str
    description: str

    class Config:
        frozen = True  # делает объект неизменяемым

    def full_name(self) -> str:
        """Возвращает полное имя репозитория."""
        return f"{self.owner}/{self.name}"
```  
**Преимущества:**  
- Автоматическая валидация полей.  
- Сериализация/десериализация (удобно для JSON, Redis и др.).  
- Возможность добавлять методы.  

#### **Сервис jsontopydantic.com**  
Автоматически генерирует модели Pydantic из JSON-ответов API.  
Пример: ответ от Todoist API → готовая модель.  

---  

### **3. TypedDict: аннотация словарей в легаси-коде**  
Если код уже использует много словарей, их можно типизировать через `TypedDict`.  

#### **Пример `GitHubRepo` как TypedDict:**  
```python
from typing import TypedDict

class GitHubRepo(TypedDict):
    """GitHub repository."""
    owner: str
    name: str
    description: str

repo: GitHubRepo = {
    "owner": "imankulov",
    "name": "empty",
    "description": "An empty repository",
}
```  
**Плюсы:**  
- IDE (например, PyCharm) подсказывают типы и находят ошибки.  
- Код остается совместимым со старыми версиями Python.  

#### **Скриншоты из PyCharm:**  
1. **Автодополнение** – IDE знает тип значения и предлагает подсказки.  
2. **Проверка типов** – если ключ пропущен, PyCharm выдаст предупреждение.  

---  

### **4. Мэппинги (`Mapping`, `MutableMapping`)**  
Если словарь используется как хранилище ключ-значение, лучше аннотировать его как `Mapping` (неизменяемый) или `MutableMapping` (изменяемый).  

#### **Пример с цветами:**  
```python
from typing import Mapping

colors: Mapping[str, str] = {
    "red": "#FF0000",
    "pink": "#FFC0CB",
    "purple": "#800080",
}

def add_yellow(colors: Mapping[str, str]):
    colors["yellow"] = "#FFFF00"  # Ошибка: Mapping не поддерживает изменение

if __name__ == "__main__":
    add_yellow(colors)  # mypy обнаружит ошибку
```  
**Почему это важно?**  
- Четкое разделение на изменяемые/неизменяемые структуры.  
- Mypy находит ошибки до запуска кода.  

---  

### **5. Вывод**  
- **Pydantic** – лучший выбор для новых проектов (валидация, сериализация, методы).  
- **TypedDict** – для постепенного рефакторинга легаси-кода.  
- **Мэппинги** – чтобы явно указать роль словаря (хранилище ключ-значение).  

**Главный совет:**  
> *"Следите за словарями. Не позволяйте им захватить ваше приложение. Чем раньше вы внедрите типизированные структуры, тем проще будет поддерживать код."*  

---  

### **Дополнение (новые примеры)**  

#### **1. Pydantic с кастомной валидацией**  
```python
from pydantic import BaseModel, validator

class Item(BaseModel):
    price: float
    discount: float

    @validator("discount")
    def check_discount(cls, v, values):
        if "price" in values and v > values["price"]:
            raise ValueError("Discount cannot exceed price!")
```  

#### **2. TypedDict с Optional-полями**  
```python
from typing import TypedDict, Optional

class User(TypedDict):
    name: str
    age: Optional[int]  # Поле может быть None

user: User = {"name": "Alice", "age": None}  # Корректно
```  

#### **3. Мэппинг с `MutableMapping`**  
```python
from typing import MutableMapping

cache: MutableMapping[str, int] = {}
cache["count"] = 10  # OK, т.к. изменяемый
```  

**Итог:** Выбор инструмента зависит от задачи, но везде цель одна – сделать код надежнее и понятнее.