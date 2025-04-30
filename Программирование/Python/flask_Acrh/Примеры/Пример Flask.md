### **Конкретные примеры слоистой архитектуры на Python**  

Слоистая архитектура в Python часто применяется в веб-приложениях (Django, Flask), CLI-утилитах и даже в десктопных приложениях. Рассмотрим несколько примеров.  

---

## **📌 Пример 1: Веб-приложение на Flask (REST API)**  

### **Структура проекта**  
```
my_flask_app/  
│── app.py                # Точка входа  
│── controllers/          # Презентационный слой (API endpoints)  
│   └── user_controller.py  
│── services/             # Бизнес-логика  
│   └── user_service.py  
│── repositories/         # Работа с данными  
│   └── user_repository.py  
│── models/               # Модели данных (DTO, Entity)  
│   └── user.py  
│── database/             # Настройка БД  
│   └── db.py  
```  

### **1. Модель (`models/user.py`)**  
```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    username: str
    email: str
```  

### **2. Репозиторий (`repositories/user_repository.py`)**  
```python
from models.user import User

class UserRepository:
    def __init__(self):
        self.users = []  # В реальности тут будет подключение к БД (SQLAlchemy, Django ORM)

    def get_user_by_id(self, user_id: int) -> User | None:
        return next((u for u in self.users if u.id == user_id), None)

    def add_user(self, user: User):
        self.users.append(user)
```  

### **3. Сервис (`services/user_service.py`)**  
```python
from models.user import User
from repositories.user_repository import UserRepository

class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    def register_user(self, username: str, email: str) -> User:
        new_user = User(id=len(self.repository.users) + 1, username=username, email=email)
        self.repository.add_user(new_user)
        return new_user

    def get_user(self, user_id: int) -> User | None:
        return self.repository.get_user_by_id(user_id)
```  

### **4. Контроллер (`controllers/user_controller.py`)**  
```python
from flask import jsonify, request
from services.user_service import UserService
from repositories.user_repository import UserRepository

user_service = UserService(UserRepository())

def init_user_routes(app):
    @app.route("/users/<int:user_id>", methods=["GET"])
    def get_user(user_id):
        user = user_service.get_user(user_id)
        return jsonify(user.__dict__) if user else ("User not found", 404)

    @app.route("/users", methods=["POST"])
    def create_user():
        data = request.get_json()
        user = user_service.register_user(data["username"], data["email"])
        return jsonify(user.__dict__), 201
```  

### **5. Точка входа (`app.py`)**  
```python
from flask import Flask
from controllers.user_controller import init_user_routes

app = Flask(__name__)
init_user_routes(app)

if __name__ == "__main__":
    app.run(debug=True)
```  

**🔹 Как это работает:**  
1. **Презентационный слой** (`user_controller.py`) – принимает HTTP-запросы.  
2. **Бизнес-логика** (`user_service.py`) – регистрация, валидация.  
3. **Слой данных** (`user_repository.py`) – сохранение/получение данных.  

---

## **📌 Пример 2: Консольное приложение (Банковский аккаунт)**  

### **Структура**  
```
bank_app/  
│── main.py               # Точка входа  
│── domain/               # Бизнес-логика  
│   └── account.py  
│── storage/              # Работа с данными  
│   └── account_repo.py  
│── ui/                   # Интерфейс  
│   └── cli_interface.py  
```  

### **1. Модель (`domain/account.py`)**  
```python
from dataclasses import dataclass

@dataclass
class BankAccount:
    id: str
    owner: str
    balance: float = 0.0

    def deposit(self, amount: float):
        if amount <= 0:
            raise ValueError("Amount must be positive")
        self.balance += amount

    def withdraw(self, amount: float):
        if amount <= 0 or amount > self.balance:
            raise ValueError("Invalid amount")
        self.balance -= amount
```  

### **2. Репозиторий (`storage/account_repo.py`)**  
```python
from domain.account import BankAccount

class AccountRepository:
    def __init__(self):
        self.accounts = {}

    def save(self, account: BankAccount):
        self.accounts[account.id] = account

    def find_by_id(self, account_id: str) -> BankAccount | None:
        return self.accounts.get(account_id)
```  

### **3. UI-слой (`ui/cli_interface.py`)**  
```python
from domain.account import BankAccount
from storage.account_repo import AccountRepository

class CLIInterface:
    def __init__(self, repo: AccountRepository):
        self.repo = repo

    def run(self):
        while True:
            print("\n1. Create account\n2. Deposit\n3. Withdraw\n4. Exit")
            choice = input("Choose: ")
            if choice == "1":
                self._create_account()
            elif choice == "2":
                self._deposit()
            elif choice == "3":
                self._withdraw()
            elif choice == "4":
                break

    def _create_account(self):
        account_id = input("Enter account ID: ")
        owner = input("Enter owner name: ")
        account = BankAccount(id=account_id, owner=owner)
        self.repo.save(account)
        print("Account created!")

    def _deposit(self):
        account_id = input("Enter account ID: ")
        amount = float(input("Enter amount: "))
        account = self.repo.find_by_id(account_id)
        if account:
            account.deposit(amount)
            print(f"New balance: {account.balance}")
        else:
            print("Account not found!")
```  

### **4. Запуск (`main.py`)**  
```python
from storage.account_repo import AccountRepository
from ui.cli_interface import CLIInterface

repo = AccountRepository()
cli = CLIInterface(repo)
cli.run()
```  

**🔹 Как это работает:**  
1. **UI-слой** (`cli_interface.py`) – взаимодействие с пользователем.  
2. **Бизнес-логика** (`account.py`) – правила операций с балансом.  
3. **Хранение данных** (`account_repo.py`) – сохранение в памяти (можно заменить на SQLite).  

---

## **📌 Вывод**  
✅ **Flask-пример** – классическая слоистая архитектура для API.  
✅ **CLI-пример** – разделение логики, данных и интерфейса.  

Можно адаптировать под Django, FastAPI или даже игры (например, отделить логику игры от рендеринга).  

Если нужен более строгий контроль зависимостей, можно добавить **Dependency Injection** (например, через `dependency-injector`).