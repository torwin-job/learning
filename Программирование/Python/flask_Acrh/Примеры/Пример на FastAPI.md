### **Слоистая архитектура в FastAPI (Python)**  

Рассмотрим пример REST API для управления задачами (Todo-лист) с четким разделением на:  
1. **Презентационный слой** (FastAPI роутеры)  
2. ***Бизнес-логику** (сервисы)*  
3. **Слой данных** (**репозиторий** + *модель*)  

---

## **📂 Структура проекта**  
```
todo_api/  
│── main.py                 # Точка входа  
│── api/                    # Презентационный слой (роутеры)  
│   └── tasks.py  
│── services/               # Бизнес-логика  
│   └── task_service.py  
│── repositories/           # Работа с данными  
│   └── task_repository.py  
│── models/                 # Модели данных (DTO, Entity)  
│   └── task.py  
│── database/               # Подключение к БД  
│   └── db.py  
```

---

## **1. Модель (`models/task.py`)**  
```python
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

# Модель для API (DTO)
class TaskCreate(BaseModel):
    title: str
    description: Optional[str] = None

class TaskResponse(TaskCreate):
    id: int
    created_at: datetime
    is_completed: bool

# Внутренняя модель (Entity)
class Task:
    def __init__(self, id: int, title: str, description: str = None, is_completed: bool = False):
        self.id = id
        self.title = title
        self.description = description
        self.is_completed = is_completed
        self.created_at = datetime.now()
```

---

## **2. Репозиторий (`repositories/task_repository.py`)**  
```python
from models.task import Task
from typing import List, Optional

class TaskRepository:
    def __init__(self):
        self.tasks: List[Task] = []
        self.current_id = 1

    def get_all(self) -> List[Task]:
        return self.tasks

    def get_by_id(self, task_id: int) -> Optional[Task]:
        return next((t for t in self.tasks if t.id == task_id), None)

    def create(self, task_data: dict) -> Task:
        new_task = Task(
            id=self.current_id,
            title=task_data["title"],
            description=task_data.get("description")
        )
        self.tasks.append(new_task)
        self.current_id += 1
        return new_task

    def mark_completed(self, task_id: int) -> Optional[Task]:
        task = self.get_by_id(task_id)
        if task:
            task.is_completed = True
        return task
```

---

## **3. Сервис (`services/task_service.py`)**  
```python
from models.task import TaskCreate, TaskResponse
from repositories.task_repository import TaskRepository
from typing import List

class TaskService:
    def __init__(self, repository: TaskRepository):
        self.repository = repository

    def get_all_tasks(self) -> List[TaskResponse]:
        tasks = self.repository.get_all()
        return [self._to_response(task) for task in tasks]

    def get_task(self, task_id: int) -> Optional[TaskResponse]:
        task = self.repository.get_by_id(task_id)
        return self._to_response(task) if task else None

    def create_task(self, task_data: TaskCreate) -> TaskResponse:
        task = self.repository.create(task_data.dict())
        return self._to_response(task)

    def complete_task(self, task_id: int) -> Optional[TaskResponse]:
        task = self.repository.mark_completed(task_id)
        return self._to_response(task) if task else None

    def _to_response(self, task) -> TaskResponse:
        return TaskResponse(
            id=task.id,
            title=task.title,
            description=task.description,
            is_completed=task.is_completed,
            created_at=task.created_at
        )
```

---

## **4. Роутер (`api/tasks.py`)**  
```python
from fastapi import APIRouter, HTTPException
from models.task import TaskCreate, TaskResponse
from services.task_service import TaskService
from repositories.task_repository import TaskRepository
from typing import List

router = APIRouter(prefix="/tasks", tags=["tasks"])
repo = TaskRepository()
service = TaskService(repo)

@router.get("/", response_model=List[TaskResponse])
def get_all_tasks():
    return service.get_all_tasks()

@router.get("/{task_id}", response_model=TaskResponse)
def get_task(task_id: int):
    task = service.get_task(task_id)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task

@router.post("/", response_model=TaskResponse, status_code=201)
def create_task(task_data: TaskCreate):
    return service.create_task(task_data)

@router.patch("/{task_id}/complete", response_model=TaskResponse)
def complete_task(task_id: int):
    task = service.complete_task(task_id)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task
```

---

## **5. Точка входа (`main.py`)**  
```python
from fastapi import FastAPI
from api.tasks import router as tasks_router

app = FastAPI()
app.include_router(tasks_router)

@app.get("/")
def home():
    return {"message": "Todo API"}
```

---

## **🔹 Как это работает?**  
1. **`GET /tasks`** → Роутер → Сервис → Репозиторий → Возвращает список задач.  
2. **`POST /tasks`** → Принимает JSON, валидирует через `TaskCreate` → Сохраняет в "БД".  
3. **`PATCH /tasks/{id}/complete`** → Помечает задачу выполненной.  

---

## **🔹 Запуск и тестирование**  
1. Установите зависимости:  
   ```bash
   pip install fastapi uvicorn
   ```
2. Запустите сервер:  
   ```bash
   uvicorn main:app --reload
   ```
3. Откройте `http://127.0.0.1:8000/docs` для доступа к Swagger UI.  

---

## **📌 Улучшения**  
- **Реальная БД**: Подключите SQLAlchemy или Django ORM вместо списка в памяти.  
- **Dependency Injection**: Используйте `Depends` в FastAPI для внедрения зависимостей.  
- **Валидация**: Добавьте больше проверок в сервисном слое.  

Этот пример показывает, как разделять ответственность в FastAPI-приложении. 🚀