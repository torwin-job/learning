# Подробный пример паттерна Facade (Фасад) на Python

Рассмотрим реалистичный пример системы домашней автоматизации (умный дом), где фасад скрывает сложность управления различными устройствами.

## Полный код примера с объяснениями

```python
# ==================== 1. Сложная подсистема (устройства умного дома) ====================

class LightSystem:
    """Система управления освещением"""
    def __init__(self):
        self.brightness = 0
    
    def turn_on(self, brightness=50):
        self.brightness = brightness
        print(f"🔆 Свет включен, яркость {self.brightness}%")
    
    def turn_off(self):
        self.brightness = 0
        print("🌑 Свет выключен")
    
    def dim(self, value):
        self.brightness = max(0, min(100, self.brightness + value))
        print(f"🔅 Яркость изменена: {self.brightness}%")

class Thermostat:
    """Система управления температурой"""
    def __init__(self):
        self.temperature = 22
    
    def set_temperature(self, temp):
        self.temperature = temp
        print(f"🌡️ Температура установлена на {self.temperature}°C")
    
    def increase_temp(self, degrees=1):
        self.temperature += degrees
        print(f"🔥 Температура повышена до {self.temperature}°C")
    
    def decrease_temp(self, degrees=1):
        self.temperature -= degrees
        print(f"❄️ Температура понижена до {self.temperature}°C")

class AudioSystem:
    """Аудиосистема"""
    def __init__(self):
        self.volume = 30
        self.current_track = None
    
    def play_music(self, track):
        self.current_track = track
        print(f"🎵 Воспроизведение: {track}")
    
    def stop_music(self):
        self.current_track = None
        print("⏹️ Музыка остановлена")
    
    def set_volume(self, level):
        self.volume = level
        print(f"🔊 Громкость установлена на {self.volume}%")

class SecuritySystem:
    """Система безопасности"""
    def __init__(self):
        self.armed = False
    
    def arm(self):
        self.armed = True
        print("🚨 Охрана включена")
    
    def disarm(self):
        self.armed = False
        print("🟢 Охрана выключена")
    
    def check_status(self):
        return "активна" if self.armed else "неактивна"

# ==================== 2. Фасад для управления умным домом ====================

class SmartHomeFacade:
    """
    Фасад для управления всеми системами умного дома через простой интерфейс.
    Скрывает сложность взаимодействия с отдельными компонентами.
    """
    def __init__(self):
        self.lights = LightSystem()
        self.thermostat = Thermostat()
        self.audio = AudioSystem()
        self.security = SecuritySystem()
    
    def good_morning(self, temperature=22):
        """Утренний режим: включает свет, устанавливает температуру, играет музыку"""
        print("\n=== Активация утреннего режима ===")
        self.lights.turn_on(70)
        self.thermostat.set_temperature(temperature)
        self.audio.play_music("Радио 'Доброе утро'")
        self.security.disarm()
    
    def good_night(self):
        """Ночной режим: выключает свет, понижает температуру, отключает музыку"""
        print("\n=== Активация ночного режима ===")
        self.lights.turn_off()
        self.thermostat.decrease_temp(3)
        self.audio.stop_music()
        self.security.arm()
        print(f"Статус охраны: {self.security.check_status()}")
    
    def party_mode(self):
        """Режим вечеринки: световые эффекты, музыка, комфортная температура"""
        print("\n=== Активация режима вечеринки ===")
        self.lights.turn_on(100)
        self.thermostat.set_temperature(21)
        self.audio.play_music("Dance Mix 2023")
        self.audio.set_volume(80)
        self.security.disarm()
    
    def leave_home(self):
        """Режим 'никого нет дома'"""
        print("\n=== Активация режима 'Ушел' ===")
        self.lights.turn_off()
        self.thermostat.set_temperature(18)
        self.audio.stop_music()
        self.security.arm()
        print(f"Статус охраны: {self.security.check_status()}")

# ==================== 3. Клиентский код ====================

def main():
    # Создаем фасад умного дома
    smart_home = SmartHomeFacade()
    
    # Демонстрация работы разных режимов
    smart_home.good_morning(23)
    
    input("\nНажмите Enter для активации вечернего режима...")
    smart_home.party_mode()
    
    input("\nНажмите Enter для активации ночного режима...")
    smart_home.good_night()
    
    input("\nНажмите Enter для активации режима 'Ушел'...")
    smart_home.leave_home()

if __name__ == "__main__":
    main()
```

## Пошаговое объяснение кода

### 1. Подсистема устройств умного дома

Мы создали 4 класса, представляющих сложные системы:
- `LightSystem` - управление освещением
- `Thermostat` - контроль температуры
- `AudioSystem` - управление аудио
- `SecuritySystem` - система безопасности

Каждый класс имеет свои специфические методы, например:
- `turn_on()`, `turn_off()` для света
- `set_temperature()` для термостата
- `play_music()` для аудиосистемы
- `arm()`, `disarm()` для охраны

### 2. Фасад SmartHomeFacade

Фасад предоставляет простые методы для сложных операций:
- `good_morning()` - включает свет, устанавливает температуру, включает музыку
- `good_night()` - готовит дом ко сну
- `party_mode()` - создает атмосферу для вечеринки
- `leave_home()` - активирует режим отсутствия хозяев

Каждый метод фасада координирует работу нескольких подсистем.

### 3. Преимущества такого подхода

1. **Упрощение клиентского кода**:
   Вместо управления каждым устройством отдельно:
   ```python
   lights.turn_on()
   thermostat.set_temperature(22)
   audio.play_music()
   security.disarm()
   ```
   Клиент просто вызывает:
   ```python
   smart_home.good_morning()
   ```

2. **Снижение связанности**:
   Если изменится логика работы термостата, нужно будет поменять только фасад, а не все места в коде, где используется термостат.

3. **Удобство тестирования**:
   Можно тестировать сложные сценарии через простой интерфейс фасада.

### 4. Пример вывода программы

```
=== Активация утреннего режима ===
🔆 Свет включен, яркость 70%
🌡️ Температура установлена на 23°C
🎵 Воспроизведение: Радио 'Доброе утро'
🟢 Охрана выключена

Нажмите Enter для активации вечернего режима...

=== Активация режима вечеринки ===
🔆 Свет включен, яркость 100%
🌡️ Температура установлена на 21°C
🎵 Воспроизведение: Dance Mix 2023
🔊 Громкость установлена на 80%
🟢 Охрана выключена

Нажмите Enter для активации ночного режима...

=== Активация ночного режима ===
🌑 Свет выключен
❄️ Температура понижена до 18°C
⏹️ Музыка остановлена
🚨 Охрана включена
Статус охраны: активна

Нажмите Enter для активации режима 'Ушел'...

=== Активация режима 'Ушел' ===
🌑 Свет выключен
🌡️ Температура установлена на 18°C
⏹️ Музыка остановлена
🚨 Охрана включена
Статус охраны: активна
```

## Когда использовать паттерн Facade?

1. **Для работы со сложными библиотеками** (например, ORM, графические библиотеки)
2. **В микросервисной архитектуре** (единый шлюз для группы сервисов)
3. **Для упрощения унаследованного кода**
4. **Когда нужно предоставить простой интерфейс к сложной подсистеме**

Фасад особенно полезен в Python для:
- Упрощения работы с API (например, обертка для requests)
- Организации сложных Django-проектов
- Создания удобных интерфейсов для научных вычислений (numpy/pandas)