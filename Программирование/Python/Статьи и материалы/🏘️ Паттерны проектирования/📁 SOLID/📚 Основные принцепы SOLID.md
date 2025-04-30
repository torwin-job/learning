# SOLID принципы объектно-ориентированного программирования

## S — Single Responsibility Principle (Принцип единственной ответственности)

**Определение**: Класс должен иметь только одну причину для изменения, то есть выполнять только одну конкретную задачу.

**Пример плохого кода**:
```python
class Employee:
    def __init__(self, name, position):
        self.name = name
        self.position = position
    
    def get_details(self):
        return f"{self.name} - {self.position}"
    
    def save_to_database(self):
        # Код для сохранения в базу данных
        pass
    
    def generate_report(self):
        # Код для создания отчета
        pass
```

**Пример хорошего кода**:
```python
class Employee:
    def __init__(self, name, position):
        self.name = name
        self.position = position
    
    def get_details(self):
        return f"{self.name} - {self.position}"

class EmployeeRepository:
    def save(self, employee):
        # Код для сохранения в базу данных
        pass

class ReportGenerator:
    def generate_employee_report(self, employee):
        # Код для создания отчета
        pass
```

## O — Open/Closed Principle (Принцип открытости/закрытости)

**Определение**: Программные сущности должны быть открыты для расширения, но закрыты для модификации.

**Пример плохого кода**:
```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

class AreaCalculator:
    def calculate_area(self, shape):
        if isinstance(shape, Rectangle):
            return shape.width * shape.height
        elif isinstance(shape, Circle):
            return 3.14 * shape.radius ** 2
        # При добавлении новой фигуры придется изменять этот метод
```

**Пример хорошего кода**:
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return 3.14 * self.radius ** 2

class AreaCalculator:
    def calculate_area(self, shape):
        return shape.area()
```

## L — Liskov Substitution Principle (Принцип подстановки Барбары Лисков)

**Определение**: Объекты базового класса должны быть заменяемы объектами его подклассов без нарушения работы программы.

**Пример плохого кода**:
```python
class Bird:
    def fly(self):
        pass

class Duck(Bird):
    def fly(self):
        print("Duck flying")

class Ostrich(Bird):
    def fly(self):
        raise Exception("Страусы не летают!")
```

**Пример хорошего кода**:
```python
class Bird:
    pass

class FlyingBird(Bird):
    def fly(self):
        pass

class Duck(FlyingBird):
    def fly(self):
        print("Duck flying")

class Ostrich(Bird):
    # Не имеет метода fly, поэтому не нарушает принцип
    pass
```

## I — Interface Segregation Principle (Принцип разделения интерфейса)

**Определение**: Клиенты не должны зависеть от методов, которые они не используют. Много специализированных интерфейсов лучше, чем один универсальный.

**Пример плохого кода**:
```python
class Worker:
    def work(self):
        pass
    
    def eat(self):
        pass
    
    def sleep(self):
        pass

class Robot(Worker):
    def work(self):
        print("Robot working")
    
    def eat(self):
        raise Exception("Роботы не едят!")
    
    def sleep(self):
        raise Exception("Роботы не спят!")
```

**Пример хорошего кода**:
```python
class Workable:
    def work(self):
        pass

class Eatable:
    def eat(self):
        pass

class Sleepable:
    def sleep(self):
        pass

class Human(Workable, Eatable, Sleepable):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating")
    
    def sleep(self):
        print("Human sleeping")

class Robot(Workable):
    def work(self):
        print("Robot working")
```

## D — Dependency Inversion Principle (Принцип инверсии зависимостей)

**Определение**: 
1. Высокоуровневые модули не должны зависеть от низкоуровневых. Оба должны зависеть от абстракций.
2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

**Пример плохого кода**:
```python
class MySQLDatabase:
    def save(self, data):
        print(f"Saving {data} to MySQL")

class UserService:
    def __init__(self):
        self.database = MySQLDatabase()  # Жесткая зависимость
    
    def save_user(self, user):
        self.database.save(user)
```

**Пример хорошего кода**:
```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):
    def save(self, data):
        print(f"Saving {data} to MySQL")

class MongoDatabase(Database):
    def save(self, data):
        print(f"Saving {data} to MongoDB")

class UserService:
    def __init__(self, database: Database):
        self.database = database  # Зависимость от абстракции
    
    def save_user(self, user):
        self.database.save(user)

# Использование
mysql_db = MySQLDatabase()
mongo_db = MongoDatabase()

# Можно легко заменить реализацию базы данных
user_service = UserService(mysql_db)
user_service.save_user("John")

user_service = UserService(mongo_db)
user_service.save_user("John")
```

## Преимущества применения SOLID

1. **Гибкость**: Код легче адаптировать к изменяющимся требованиям
2. **Расширяемость**: Добавление новых функций без изменения существующего кода
3. **Тестируемость**: Отдельные компоненты проще тестировать
4. **Понятность**: Код более читаемый и поддерживаемый
5. **Снижение связанности**: Модули менее зависимы друг от друга


