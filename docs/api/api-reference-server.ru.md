# Серверный API (MCEngineServer)

Серверный API предоставляет доступ к основным серверным функциям Minecraft. Все методы доступны в JavaScript через глобальный объект `MCEngineServer`.

!!! warning "Внимание"
    Некоторые методы могут не работать так как есть ошибки на клиентской части мода. А именно: model, texture -
    пока что не работают.

!!! info "Совет"
    Если вы спавните какую-то сущность обращайте свое внимание на позиции в мире x, y, z. В ином случае вы не 
    увидите то что пытаетесь заспавнить. Для тестов или разработки лучше используйте "плоский мир"

---

## Базовые методы

### onInit(callback)

Регистрирует функцию, которая будет вызвана при инициализации серверного скрипта.

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `callback` | Function | Функция, вызываемая при инициализации |

**Пример:**

```javascript
MCEngineServer.onInit(() => {
    console.log("Серверный скрипт инициализирован!");
});
```

---

### log(message)

Выводит сообщение в консоль сервера и в чат игрока (если скрипт вызван из команды).

**Параметры:**

| Параметр  | Тип    | Описание             |
|-----------|--------|----------------------|
| `message` | String | Сообщение для вывода |

**Пример:**

```javascript
MCEngineServer.log("Это сообщение появится в консоли");
```

---

### onServerTick(callback)

Регистрирует функцию, которая будет вызываться каждый серверный тик (20 раз в секунду).

!!! warning "Внимание"
    Не выполняй тяжелые вычисления в этом коллбэке, так как он вызывается очень часто и может вызвать лаги сервера.

**Параметры:**

| Параметр   | Тип      | Описание                       |
|------------|----------|--------------------------------|
| `callback` | Function | Функция, вызываемая каждый тик |

**Пример:**

```javascript
var tickCounter = 0;

MCEngineServer.onServerTick(() => {
    tickCounter++;
    
    // Каждые 20 тиков (1 секунда)
    if (tickCounter % 20 === 0) {
        MCEngineServer.log("Прошла 1 секунда!");
    }
});
```

---

### clearQueueTick()

Очищает все зарегистрированные коллбэки `onServerTick`. Полезно при перезагрузке скриптов.

**Пример:**

```javascript
MCEngineServer.clearQueueTick();
```

---

## Спавн мобов

### spawnEntity(entityId, x, y, z)

Спавнит сущность по её ID в Overworld на указанных координатах.

**Параметры:**

| Параметр   | Тип    | Описание                                     |
|------------|--------|----------------------------------------------|
| `entityId` | String | ID сущности (например, `"minecraft:zombie"`) |
| `x`        | Number | Координата X                                 |
| `y`        | Number | Координата Y                                 |
| `z`        | Number | Координата Z                                 |

**Возвращает:** `boolean` — `true`, если моб успешно заспавнен.

**Пример:**

```javascript
var success = MCEngineServer.spawnEntity("minecraft:zombie", 100, 64, 200);

if (success) {
    MCEngineServer.log("Зомби заспавнен!");
} else {
    MCEngineServer.log("Ошибка спавна!");
}
```

---

### spawnNamedEntity(entityId, x, y, z, customName)

Спавнит моба с кастомным именем, которое будет отображаться над его головой.

**Параметры:**

| Параметр     | Тип    | Описание                                                          |
|--------------|--------|-------------------------------------------------------------------|
| `entityId`   | String | ID сущности                                                       |
| `x, y, z`    | Number | Координаты спавна                                                 |
| `customName` | String | Имя моба (поддерживает цветовые коды, например `"§cКрасный моб"`) |

**Возвращает:** `boolean` — `true`, если моб успешно заспавнен.

**Пример:**

```javascript
MCEngineServer.spawnNamedEntity(
    "minecraft:skeleton", 
    0, 100, 0, 
    "§b§lСКЕЛЕТ-ЛУЧНИК"
);
```

---

### createCustomMob(baseEntityType, x, y, z, config)

Создает кастомного моба с уникальным AI, моделью и поведением. Это самый мощный метод API.

**Параметры:**

| Параметр         | Тип    | Описание                                             |
|------------------|--------|------------------------------------------------------|
| `baseEntityType` | String | ID базовой сущности (например, `"minecraft:zombie"`) |
| `x, y, z`        | Number | Координаты спавна                                    |
| `config`         | Object | Объект конфигурации (см. ниже)                       |

**Структура `config`:**

| Поле         | Тип      | Описание                              |
|--------------|----------|---------------------------------------|
| `name`       | String   | Имя моба                              |
| `health`     | Number   | Здоровье моба                         |
| `scale`      | Number   | Масштаб модели (1.0 = обычный размер) |
| `model`      | String   | ID кастомной модели                   |
| `texture`    | String   | Путь к кастомной текстуре             |
| `invisible`  | Boolean  | Сделать моба невидимым                |
| `silent`     | Boolean  | Отключить звуки моба                  |
| `onTick`     | Function | Коллбэк, вызываемый каждый тик        |
| `onAttack`   | Function | Коллбэк при атаке                     |
| `onDeath`    | Function | Коллбэк при смерти                    |
| `onInteract` | Function | Коллбэк при взаимодействии            |

**Возвращает:** `MobWrapper` — обертка над созданным мобом.

**Пример создания босса:**

```javascript
var boss = MCEngineServer.createCustomMob("minecraft:zombie", 0, 100, 0, {
    name: "§c§lДРЕВНИЙ СТРАЖ",
    health: 500.0,
    scale: 1.5,
    model: "mcengine:custom_zombie",
    
    onTick: (mob) => {
        // Поджигаем себя при низком HP
        if (mob.getHealth() < 100) {
            mob.setOnFire(5);
        }
        
        // Ищем ближайшего игрока
        var player = mob.findNearestPlayer(20);
        if (player != null) {
            mob.moveTo(player.getX(), player.getY(), player.getZ(), 0.5);
        }
    },
    
    onDeath: (mob) => {
        MCEngineServer.log("Страж повержен!");
    }
});
```

---

## Модификация AI

### modifyAI(target, config)

Модифицирует AI мобов. Можно применить глобально (ко всем мобам определенного типа) или к конкретному экземпляру.

**Параметры:**

| Параметр | Тип                   | Описание                                                                   |
|----------|-----------------------|----------------------------------------------------------------------------|
| `target` | String или MobWrapper | ID типа моба (например, `"minecraft:zombie"`) или обертка конкретного моба |
| `config` | Object                | Объект конфигурации AI                                                     |

**Структура `config`:**

| Поле               | Тип      | Описание                         |
|--------------------|----------|----------------------------------|
| `onTick`           | Function | Коллбэк каждый тик               |
| `onAttack`         | Function | Коллбэк при атаке                |
| `onDeath`          | Function | Коллбэк при смерти               |
| `onTarget`         | Function | Коллбэк при выборе цели          |
| `onHurt`           | Function | Коллбэк при получении урона      |
| `onInteract`       | Function | Коллбэк при взаимодействии       |
| `disableVanillaAI` | Boolean  | Полностью отключить ванильный AI |
| `disableMovement`  | Boolean  | Запретить движение               |
| `disableAttacking` | Boolean  | Запретить атаки                  |

**Пример глобальной модификации:**

```javascript
// Делаем всех зомби мирными
MCEngineServer.modifyAI("minecraft:zombie", {
    disableAttacking: true,
    
    onTick: (mob) => {
        // Зомби бродит случайным образом
        if (Math.random() < 0.01) {
            var x = mob.getX() + (Math.random() * 10 - 5);
            var z = mob.getZ() + (Math.random() * 10 - 5);
            mob.moveTo(x, mob.getY(), z, 0.3);
        }
    }
});
```

**Пример модификации конкретного моба:**

```javascript
var zombie = MCEngineServer.spawnEntity("minecraft:zombie", 0, 100, 0);
var mobWrapper = new MobWrapper(zombie);

MCEngineServer.modifyAI(mobWrapper, {
    onDeath: (mob) => {
        MCEngineServer.log("Конкретный зомби умер!");
    }
});
```

---

## Замена моделей

### setModel(target, config)

Заменяет модель моба на кастомную. Работает через систему Resource Pack overrides.

**Параметры:**

| Параметр | Тип                   | Описание                                  |
|----------|-----------------------|-------------------------------------------|
| `target` | String или MobWrapper | ID типа моба или обертка конкретного моба |
| `config` | Object                | Объект конфигурации модели                |

**Структура `config`:**

| Поле          | Тип     | Описание                  |
|---------------|---------|---------------------------|
| `model`       | String  | ID кастомной модели       |
| `texture`     | String  | Путь к кастомной текстуре |
| `scale`       | Number  | Масштаб модели            |
| `hideVanilla` | Boolean | Скрыть ванильную модель   |

**Пример:**

```javascript
// Заменяем модель всех зомби
MCEngineServer.setModel("minecraft:zombie", {
    model: "mcengine:custom_zombie",
    texture: "mcengine:textures/entity/custom_zombie.png",
    scale: 1.2,
    hideVanilla: false
});
```

---

## Работа с игроками

### getPlayer(name)

Получает игрока по его никнейму.

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `name` | String | Никнейм игрока |

**Возвращает:** `PlayerWrapper` или `null`, если игрок не найден.

**Пример:**

```javascript
var player = MCEngineServer.getPlayer("Steve");

if (player != null) {
    player.sendMessage("Привет, " + player.getName() + "!");
}
```

---

### getAllPlayers()

Получает список всех игроков онлайн.

**Возвращает:** `Array<PlayerWrapper>` — массив оберток игроков.

**Пример:**

```javascript
var players = MCEngineServer.getAllPlayers();

for (var i = 0; i < players.length; i++) {
    players[i].sendMessage("Всем привет!");
}
```

---

### giveItem(playerName, itemId, count)

Выдает предмет игроку.

**Параметры:**

| Параметр     | Тип    | Описание                                      |
|--------------|--------|-----------------------------------------------|
| `playerName` | String | Никнейм игрока                                |
| `itemId`     | String | ID предмета (например, `"minecraft:diamond"`) |
| `count`      | Number | Количество предметов                          |

**Возвращает:** `boolean` — `true`, если предмет успешно выдан.

**Пример:**

```javascript
MCEngineServer.giveItem("Steve", "minecraft:diamond", 10);
```

---

### hasItem(playerName, itemId, minCount)

Проверяет, есть ли у игрока определенное количество предметов.

**Параметры:**

| Параметр     | Тип    | Описание               |
|--------------|--------|------------------------|
| `playerName` | String | Никнейм игрока         |
| `itemId`     | String | ID предмета            |
| `minCount`   | Number | Минимальное количество |

**Возвращает:** `boolean` — `true`, если у игрока есть нужное количество предметов.

**Пример:**

```javascript
if (MCEngineServer.hasItem("Steve", "minecraft:diamond", 5)) {
    MCEngineServer.log("У Стива есть 5 алмазов!");
}
```

---

### removeItem(playerName, itemId, count)

Забирает предметы у игрока.

**Параметры:**

| Параметр     | Тип    | Описание                |
|--------------|--------|-------------------------|
| `playerName` | String | Никнейм игрока          |
| `itemId`     | String | ID предмета             |
| `count`      | Number | Количество для удаления |

**Возвращает:** `boolean` — `true`, если предметы успешно удалены.

**Пример:**

```javascript
MCEngineServer.removeItem("Steve", "minecraft:diamond", 5);
```

---

## Управление враждебностью

### makeHostile(entityType1, entityType2)

Делает два типа мобов враждебными друг к другу через систему команд `/team`.

**Параметры:**

| Параметр      | Тип    | Описание             |
|---------------|--------|----------------------|
| `entityType1` | String | ID первого типа моба |
| `entityType2` | String | ID второго типа моба |

**Пример:**

```javascript
// Зомби и скелеты теперь атакуют друг друга
MCEngineServer.makeHostile("minecraft:zombie", "minecraft:skeleton");
```

---

## Очистка

### clearAllModifications()

Очищает все AI-модификации, замены моделей и коллбэки. Полезно при перезагрузке скриптов.

**Пример:**

```javascript
MCEngineServer.clearAllModifications();
```

---

## Полный пример

Вот полный пример скрипта, который создает кастомного босса с уникальным поведением:

```javascript
MCEngineServer.onInit(() => {
    MCEngineServer.log("Скрипт босса загружен!");
});

// Спавн босса через 5 секунд после запуска
var tickCounter = 0;

MCEngineServer.onServerTick(() => {
    tickCounter++;
    
    if (tickCounter === 100) { // 5 секунд (100 тиков)
        var boss = MCEngineServer.createCustomMob("minecraft:zombie", 0, 100, 0, {
            name: "§c§lДРЕВНИЙ СТРАЖ",
            health: 500.0,
            scale: 1.5,
            
            onTick: (mob) => {
                // Ищем ближайшего игрока
                var player = mob.findNearestPlayer(30);
                
                if (player != null) {
                    // Двигаемся к игроку
                    mob.moveTo(player.getX(), player.getY(), player.getZ(), 0.4);
                    
                    // Если близко — атакуем
                    if (mob.distanceToPlayer(player) < 2) {
                        mob.attackTarget(player);
                    }
                }
                
                // Поджигаем себя при низком HP
                if (mob.getHealth() < 100) {
                    mob.setOnFire(3);
                }
            },
            
            onDeath: (mob) => {
                MCEngineServer.log("Страж повержен!");
                
                // Награждаем ближайшего игрока
                var player = mob.findNearestPlayer(50);
                if (player != null) {
                    MCEngineServer.giveItem(player.getName(), "minecraft:diamond", 10);
                    player.sendMessage("§aТы победил Стража! +10 алмазов");
                }
            }
        });
        
        MCEngineServer.log("Босс заспавнен!");
    }
});
```