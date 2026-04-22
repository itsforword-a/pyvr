# VRFX - Фреймворк для создания VR-приложений на Python

VRFX - это библиотека Python для создания приложений виртуальной реальности с поддержкой WebXR (браузерная VR) и SteamVR (нативные шлемы). Библиотека автоматически определяет окружение и запускает соответствующий бэкенд без ручной настройки.

![Stars](https://img.shields.io/github/stars/username/pyvrfx)

## Возможности

- Два режима работы: WebXR для браузерной VR и SteamVR для нативных шлемов
- Автоматическое определение окружения и запуск сервера
- Отслеживание положения контроллеров и нажатий кнопок в реальном времени
- Простое управление сценой: кубы, сферы и пользовательские модели
- Событийный цикл обновления
- Не требует ручного запуска сервера

## Установка

### Базовая установка (только WebXR)

```bash
pip install aiohttp
git clone https://github.com/yourusername/pyvrfx.git
cd pyvrfx
pip install -e .
```

### Полная установка (с поддержкой SteamVR)

```bash
pip install -e .[steamvr]
```

### Требования

- Python 3.8 или выше
- aiohttp >= 3.8.0
- Для SteamVR: openxr, pyopenxr, numpy
- Для WebXR: современный браузер с поддержкой WebXR (Meta Quest, Firefox Reality или Chrome с включенным WebXR)

## Быстрый старт

### Приложение для WebXR

```python
from pyvrfx import VRFX

app = VRFX(mode="web", port=8012)

cube = app.scene.add_cube("my_cube", x=0, y=1, z=-2, color=0xff3366)

@app.on_update
def update(dt):
    cube.x += 0.01

if __name__ == "__main__":
    app.run()
```

### Приложение для SteamVR

```python
from pyvrfx import VRFX

app = VRFX(mode="steamvr")

cube = app.scene.add_cube("target", x=0, y=1, z=-2, color=0xff4444)

@app.on_update
def update(dt):
    if app.controllers.get('right'):
        right = app.controllers['right']
        if right.trigger > 0.5:
            cube.color = 0x00ff00
            app.scene.update_object("target", color=cube.color)

if __name__ == "__main__":
    app.run()
```

### Автоматический режим

```python
from pyvrfx import VRFX

app = VRFX(mode="auto")  # Автоматически определяет SteamVR или использует WebXR

app.scene.add_cube("test", x=0, y=1, z=-2)
app.run()
```

## Справочник API

### Класс VRFX

Главный класс приложения.

**Конструктор:**
```python
VRFX(mode: str = "auto", port: int = 8012)
```

**Параметры:**
- `mode`: Одно из значений "auto", "web" или "steamvr". В режиме auto библиотека проверяет наличие SteamVR.
- `port`: Номер порта для WebXR сервера (игнорируется в режиме SteamVR)

**Методы:**
- `on_update(callback)`: Декоратор или метод для регистрации функций обновления
- `run()`: Запускает VR-приложение

### Класс Scene

Управляет 3D-объектами в виртуальной среде.

**Методы:**
- `add_cube(name, x, y, z, color, size)`: Добавляет куб на сцену
- `add_sphere(name, x, y, z, color, radius)`: Добавляет сферу на сцену
- `add_model(name, model_path, x, y, z, scale)`: Добавляет 3D-модель
- `update_object(name, **kwargs)`: Обновляет свойства объекта
- `remove_object(name)`: Удаляет объект со сцены
- `set_light(intensity, color, x, y, z)`: Настраивает направленный свет

### Свойства SceneObject

Все объекты сцены имеют следующие свойства:
- `x, y, z`: Координаты положения
- `color`: Цвет в шестнадцатеричном формате (целое число)
- `scale`: Масштаб объекта

**Методы:**
- `move(dx, dy, dz)`: Перемещает объект на указанные значения
- `set_position(x, y, z)`: Устанавливает абсолютное положение

### Класс Controller

Представляет VR-контроллер.

**Свойства:**
- `side`: Строка "left" или "right"
- `x, y, z`: Положение контроллера в мировом пространстве
- `trigger`: Значение от 0.0 до 1.0 (нажатие курка)
- `grip`: Значение от 0.0 до 1.0 (нажатие хвата)
- `menu`: Логическое значение (кнопка меню)

**Методы:**
- `distance_to(x, y, z)`: Возвращает евклидово расстояние до точки
- `is_pointing_at(x, y, z, max_distance)`: Проверяет, находится ли контроллер рядом с точкой

## Примеры

### Интерактивные объекты с управлением от контроллеров

```python
from pyvrfx import VRFX
import math

app = VRFX()

cube = app.scene.add_cube("interactive", x=0, y=1, z=-2, color=0xff6600)
sphere = app.scene.add_sphere("target", x=1.5, y=0.5, z=-1.5, color=0x44aaff)

@app.on_update
def update(dt):
    if app.controllers.get('right'):
        right = app.controllers['right']
        
        if right.is_pointing_at(cube.x, cube.y, cube.z, 0.5):
            cube.color = 0xff0000
            cube.y += 0.1
        else:
            cube.color = 0xff6600
        
        if right.trigger > 0.8:
            sphere.color = 0xffaa44
        else:
            sphere.color = 0x44aaff
        
        app.scene.update_object("interactive", color=cube.color, y=cube.y)
        app.scene.update_object("target", color=sphere.color)

app.run()
```

### Анимированная сцена с вращающимися объектами

```python
from pyvrfx import VRFX
import math
import time

app = VRFX()

cubes = []
for i in range(5):
    cube = app.scene.add_cube(f"cube_{i}", x=i*1.5-3, y=1, z=-3, color=0xff6600 + i*1000)
    cubes.append(cube)

@app.on_update
def update(dt):
    t = time.time()
    for i, cube in enumerate(cubes):
        cube.y = 0.8 + math.sin(t * 2 + i) * 0.5
        cube.x += math.sin(t * 3 + i) * 0.02
        app.scene.update_object(cube.name, y=cube.y, x=cube.x)

app.run()
```

### Простая игра: поймай объект контроллером

```python
from pyvrfx import VRFX
import random

app = VRFX()

score = 0
target = app.scene.add_cube("target", x=0, y=1, z=-2, color=0xff3366)

def move_target():
    target.x = random.uniform(-2, 2)
    target.y = random.uniform(0.5, 1.8)
    target.z = random.uniform(-3, -1)
    app.scene.update_object("target", x=target.x, y=target.y, z=target.z)

@app.on_update
def check_hit(dt):
    global score
    
    if app.controllers.get('right'):
        right = app.controllers['right']
        distance = right.distance_to(target.x, target.y, target.z)
        
        if distance < 0.4:
            score += 1
            move_target()
            print(f"Счёт: {score}")
            
            target.color = 0x00ff00
            app.scene.update_object("target", color=target.color)
        else:
            target.color = 0xff3366
            app.scene.update_object("target", color=target.color)

move_target()
app.run()
```

## Режимы запуска

### WebXR режим

Подходит для разработки и тестирования. Работает в любом браузере с поддержкой WebXR.

Особенности:
- Не требует установки драйверов VR
- Можно тестировать на обычном компьютере
- Для полноценного VR нужен Meta Quest или другой WebXR-шлем

### SteamVR режим

Для профессиональных VR-приложений с максимальной производительностью.

Особенности:
- Требует установленного SteamVR
- Работает с любыми SteamVR-совместимыми шлемами
- Более низкая задержка
- Полный доступ к возможностям OpenXR

### Автоматический режим

Библиотека сама определяет, запущен ли SteamVR, и выбирает подходящий бэкенд.

## Структура проекта для библиотеки

```
pyvrfx/
├── __init__.py          # Основной модуль
├── core.py              # Ядро библиотеки
├── vr_backend.py        # Бэкенды WebXR и SteamVR
├── scene.py             # Управление сценой
├── controllers.py       # Работа с контроллерами
├── launcher.py          # Автоматический запуск
└── web/
    └── index.html       # HTML-страница для WebXR
```

При обнаружении ошибок или для предложений по улучшению, пожалуйста, создавайте issue на GitHub.
```
