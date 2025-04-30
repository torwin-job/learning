### **Чистый Python: Калькулятор с Слоистой Архитектурой**  

Рассмотрим простое консольное приложение **калькулятора**, разбитое на слои:  
1. **Презентационный слой (UI)** – ввод/вывод в консоли.  
2. **Бизнес-логика (Service)** – вычисления.  
3. **Слой данных (Repository)** – история операций (можно сохранять в файл).  

---

## **📂 Структура проекта**  
```
calculator/  
│── main.py          # Точка входа  
│── ui/              # Презентационный слой  
│   └── cli.py  
│── services/        # Бизнес-логика  
│   └── calculator.py  
│── repositories/    # История операций  
│   └── history_repo.py  
│── models/          # Модели данных  
│   └── operation.py  
```

---

## **1. Модель (`models/operation.py`)**  
```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Operation:
    operand1: float
    operand2: float
    operator: str
    result: float
    timestamp: datetime = datetime.now()
```

---

## **2. Репозиторий (`repositories/history_repo.py`)**  
```python
from models.operation import Operation
from typing import List

class HistoryRepository:
    def __init__(self):
        self.history: List[Operation] = []

    def add(self, operation: Operation):
        self.history.append(operation)

    def get_all(self) -> List[Operation]:
        return self.history.copy()
```

---

## **3. Сервис (`services/calculator.py`)**  
```python
from models.operation import Operation
from repositories.history_repo import HistoryRepository

class CalculatorService:
    def __init__(self, history_repo: HistoryRepository):
        self.history_repo = history_repo

    def calculate(self, operand1: float, operand2: float, operator: str) -> float:
        match operator:
            case "+":
                result = operand1 + operand2
            case "-":
                result = operand1 - operand2
            case "*":
                result = operand1 * operand2
            case "/":
                if operand2 == 0:
                    raise ValueError("Division by zero")
                result = operand1 / operand2
            case _:
                raise ValueError("Unknown operator")

        operation = Operation(operand1, operand2, operator, result)
        self.history_repo.add(operation)
        return result
```

---

## **4. Презентационный слой (`ui/cli.py`)**  
```python
from services.calculator import CalculatorService
from repositories.history_repo import HistoryRepository

class CLI:
    def __init__(self, calculator: CalculatorService):
        self.calculator = calculator

    def run(self):
        while True:
            print("\n1. Calculate\n2. Show history\n3. Exit")
            choice = input("Choose: ")

            if choice == "1":
                self._calculate()
            elif choice == "2":
                self._show_history()
            elif choice == "3":
                break
            else:
                print("Invalid choice")

    def _calculate(self):
        try:
            operand1 = float(input("Enter first number: "))
            operator = input("Enter operator (+, -, *, /): ")
            operand2 = float(input("Enter second number: "))

            result = self.calculator.calculate(operand1, operand2, operator)
            print(f"Result: {result}")
        except ValueError as e:
            print(f"Error: {e}")

    def _show_history(self):
        history = self.calculator.history_repo.get_all()
        if not history:
            print("History is empty")
            return

        print("\nHistory:")
        for op in history:
            print(f"{op.operand1} {op.operator} {op.operand2} = {op.result} ({op.timestamp})")
```

---

## **5. Точка входа (`main.py`)**  
```python
from repositories.history_repo import HistoryRepository
from services.calculator import CalculatorService
from ui.cli import CLI

def main():
    repo = HistoryRepository()
    calculator = CalculatorService(repo)
    cli = CLI(calculator)
    cli.run()

if __name__ == "__main__":
    main()
```

---

## **🔹 Как это работает?**  
1. **Презентационный слой (`cli.py`)**  
   - Принимает ввод пользователя.  
   - Выводит результаты и ошибки.  

2. **Бизнес-логика (`calculator.py`)**  
   - Выполняет вычисления (`+, -, *, /`).  
   - Обрабатывает ошибки (деление на ноль).  

3. **Слой данных (`history_repo.py`)**  
   - Сохраняет историю операций.  
   - Позволяет просматривать её.  

---

## **🔹 Запуск**  
1. Создайте файлы по структуре выше.  
2. Запустите:  
   ```bash
   python main.py
   ```
3. Пример работы:  
   ```
   4. Calculate
   5. Show history
   6. Exit
   Choose: 1
   Enter first number: 10
   Enter operator (+, -, *, /): *
   Enter second number: 2
   Result: 20.0
   ```

---

## **📌 Улучшения**  
- **Сохранение истории в файл** (JSON, CSV).  
- **Логирование ошибок**.  
- **Юнит-тесты** для `CalculatorService`.  

Этот пример показывает, как разделять логику даже в простых приложениях. 🚀