
# Server API (MCEngineServer)

The Server API provides access to core Minecraft server functions. All methods are available in JavaScript via the global `MCEngineServer` object.

!!! warning "Warning"
    Some methods may not work due to bugs in the client-side part of the mod. Specifically: `model` and `texture` do not work at the moment.

!!! info "Tip"
    If you are spawning an entity, pay close attention to the `x, y, z` world coordinates. Otherwise, you will not see what you are trying to spawn. For testing or development, it is better to use a "Superflat" world.

---

## Basic Methods

### onInit(callback)

Registers a function that will be called upon server script initialization.

**Parameters:**

| Parameter | Type     | Description                          |
|-----------|----------|--------------------------------------|
| `callback`| Function | The function called upon initialization |

**Example:**

```javascript
MCEngineServer.onInit(() => {
    console.log("Server script initialized!");
});
```

---

### log(message)

Outputs a message to the server console and to the player's chat (if the script is triggered by a command).

**Parameters:**

| Parameter | Type   | Description           |
|-----------|--------|-----------------------|
| `message` | String | The message to output |

**Example:**

```javascript
console.log("This message will appear in the console");
```

---

### onServerTick(callback)

Registers a function that will be called every server tick (20 times per second).

!!! warning "Warning"
    Do not perform heavy computations in this callback, as it is called very frequently and may cause server lag.

**Parameters:**

| Parameter  | Type     | Description                        |
|------------|----------|------------------------------------|
| `callback` | Function | The function called every tick     |

**Example:**

```javascript
var tickCounter = 0;

MCEngineServer.onServerTick(() => {
    tickCounter++;
    
    // Every 20 ticks (1 second)
    if (tickCounter % 20 === 0) {
        console.log("1 second has passed!");
    }
});
```

---

### clearQueueTick()

Clears all registered `onServerTick` callbacks. Useful when reloading scripts.

**Example:**

```javascript
MCEngineServer.clearQueueTick();
```

---

## Mob Spawning

### spawnEntity(entityId, x, y, z)

Spawns an entity by its ID in the Overworld at the specified coordinates.

**Parameters:**

| Parameter  | Type   | Description                                      |
|------------|--------|--------------------------------------------------|
| `entityId` | String | Entity ID (e.g., `"minecraft:zombie"`)           |
| `x`        | Number | X coordinate                                     |
| `y`        | Number | Y coordinate                                     |
| `z`        | Number | Z coordinate                                     |

**Returns:** `boolean` — `true` if the mob was successfully spawned.

**Example:**

```javascript
var success = MCEngineServer.spawnEntity("minecraft:zombie", 100, 64, 200);

if (success) {
    console.log("Zombie spawned!");
} else {
    console.log("Spawn error!");
}
```

---

### spawnNamedEntity(entityId, x, y, z, customName)

Spawns a mob with a custom name that will be displayed above its head.

**Parameters:**

| Parameter    | Type   | Description                                                                 |
|--------------|--------|-----------------------------------------------------------------------------|
| `entityId`   | String | Entity ID                                                                   |
| `x, y, z`    | Number | Spawn coordinates                                                           |
| `customName` | String | Mob name (supports color codes, e.g., `"§cRed Mob"`)                        |

**Returns:** `boolean` — `true` if the mob was successfully spawned.

**Example:**

```javascript
MCEngineServer.spawnNamedEntity(
    "minecraft:skeleton", 
    0, 100, 0, 
    "§b§lSKELETON ARCHER"
);
```

---

### createCustomMob(baseEntityType, x, y, z, config)

Creates a custom mob with unique AI, model, and behavior. This is the most powerful API method.

**Parameters:**

| Parameter        | Type   | Description                                          |
|------------------|--------|------------------------------------------------------|
| `baseEntityType` | String | Base entity ID (e.g., `"minecraft:zombie"`)          |
| `x, y, z`        | Number | Spawn coordinates                                    |
| `config`         | Object | Configuration object (see below)                     |

**`config` Structure:**

| Field        | Type     | Description                                     |
|--------------|----------|-------------------------------------------------|
| `name`       | String   | Mob name                                        |
| `health`     | Number   | Mob health                                      |
| `scale`      | Number   | Model scale (1.0 = normal size)                 |
| `model`      | String   | Custom model ID                                 |
| `texture`    | String   | Path to custom texture                          |
| `invisible`  | Boolean  | Make the mob invisible                          |
| `silent`     | Boolean  | Disable mob sounds                              |
| `onTick`     | Function | Callback called every tick                      |
| `onAttack`   | Function | Callback on attack                              |
| `onDeath`    | Function | Callback on death                               |
| `onInteract` | Function | Callback on interaction                         |

**Returns:** `MobWrapper` — A wrapper around the created mob.

**Boss Creation Example:**

```javascript
var boss = MCEngineServer.createCustomMob("minecraft:zombie", 0, 100, 0, {
    name: "§c§lANCIENT GUARDIAN",
    health: 500.0,
    scale: 1.5,
    model: "mcengine:custom_zombie",
    
    onTick: (mob) => {
        // Ignite ourselves at low HP
        if (mob.getHealth() < 100) {
            mob.setOnFire(5);
        }
        
        // Find the nearest player
        var player = mob.findNearestPlayer(20);
        if (player != null) {
            mob.moveTo(player.getX(), player.getY(), player.getZ(), 0.5);
        }
    },
    
    onDeath: (mob) => {
        console.log("Guardian defeated!");
    }
});
```

---

## AI Modification

### modifyAI(target, config)

Modifies mob AI. Can be applied globally (to all mobs of a specific type) or to a specific instance.

**Parameters:**

| Parameter | Type                | Description                                                                |
|-----------|---------------------|----------------------------------------------------------------------------|
| `target`  | String or MobWrapper| Mob type ID (e.g., `"minecraft:zombie"`) or a specific mob wrapper         |
| `config`  | Object              | AI configuration object                                                    |

**`config` Structure:**

| Field              | Type    | Description                                |
|--------------------|---------|--------------------------------------------|
| `onTick`           | Function| Callback every tick                        |
| `onAttack`         | Function| Callback on attack                         |
| `onDeath`          | Function| Callback on death                          |
| `onTarget`         | Function| Callback when choosing a target            |
| `onHurt`           | Function| Callback when taking damage                |
| `onInteract`       | Function| Callback on interaction                    |
| `disableVanillaAI` | Boolean | Completely disable vanilla AI              |
| `disableMovement`  | Boolean | Prevent movement                           |
| `disableAttacking` | Boolean | Prevent attacks                            |

**Global Modification Example:**

```javascript
// Make all zombies peaceful
MCEngineServer.modifyAI("minecraft:zombie", {
    disableAttacking: true,
    
    onTick: (mob) => {
        // Zombie wanders randomly
        if (Math.random() < 0.01) {
            var x = mob.getX() + (Math.random() * 10 - 5);
            var z = mob.getZ() + (Math.random() * 10 - 5);
            mob.moveTo(x, mob.getY(), z, 0.3);
        }
    }
});
```

**Specific Mob Modification Example:**

```javascript
var zombie = MCEngineServer.spawnEntity("minecraft:zombie", 0, 100, 0);
var mobWrapper = new MobWrapper(zombie);

MCEngineServer.modifyAI(mobWrapper, {
    onDeath: (mob) => {
        console.log("Specific zombie died!");
    }
});
```

---

## Model Replacement

### setModel(target, config)

Replaces a mob's model with a custom one. Works via the Resource Pack overrides system.

**Parameters:**

| Parameter | Type                | Description                                 |
|-----------|---------------------|---------------------------------------------|
| `target`  | String or MobWrapper| Mob type ID or a specific mob wrapper       |
| `config`  | Object              | Model configuration object                  |

**`config` Structure:**

| Field         | Type    | Description                 |
|---------------|---------|-----------------------------|
| `model`       | String  | Custom model ID             |
| `texture`     | String  | Path to custom texture      |
| `scale`       | Number  | Model scale                 |
| `hideVanilla` | Boolean | Hide the vanilla model      |

**Example:**

```javascript
// Replace all zombie models
MCEngineServer.setModel("minecraft:zombie", {
    model: "mcengine:custom_zombie",
    texture: "mcengine:textures/entity/custom_zombie.png",
    scale: 1.2,
    hideVanilla: false
});
```

---

## Working with Players

### getPlayer(name)

Gets a player by their nickname.

**Parameters:**

| Parameter | Type   | Description      |
|-----------|--------|------------------|
| `name`    | String | Player nickname  |

**Returns:** `PlayerWrapper` or `null` if the player is not found.

**Example:**

```javascript
var player = MCEngineServer.getPlayer("Steve");

if (player != null) {
    player.sendMessage("Hello, " + player.getName() + "!");
}
```

---

### getAllPlayers()

Gets a list of all online players.

**Returns:** `Array<PlayerWrapper>` — Array of player wrappers.

**Example:**

```javascript
var players = MCEngineServer.getAllPlayers();

for (var i = 0; i < players.length; i++) {
    players[i].sendMessage("Hello everyone!");
}
```

---

### giveItem(playerName, itemId, count)

Gives an item to a player.

**Parameters:**

| Parameter    | Type   | Description                                     |
|--------------|--------|-------------------------------------------------|
| `playerName` | String | Player nickname                                 |
| `itemId`     | String | Item ID (e.g., `"minecraft:diamond"`)           |
| `count`      | Number | Amount of items                                 |

**Returns:** `boolean` — `true` if the item was successfully given.

**Example:**

```javascript
MCEngineServer.giveItem("Steve", "minecraft:diamond", 10);
```

---

### hasItem(playerName, itemId, minCount)

Checks if a player has a certain amount of an item.

**Parameters:**

| Parameter    | Type   | Description          |
|--------------|--------|----------------------|
| `playerName` | String | Player nickname      |
| `itemId`     | String | Item ID              |
| `minCount`   | Number | Minimum amount       |

**Returns:** `boolean` — `true` if the player has the required amount of items.

**Example:**

```javascript
if (MCEngineServer.hasItem("Steve", "minecraft:diamond", 5)) {
    console.log("Steve has 5 diamonds!");
}
```

---

### removeItem(playerName, itemId, count)

Takes items away from a player.

**Parameters:**

| Parameter    | Type   | Description                 |
|--------------|--------|-----------------------------|
| `playerName` | String | Player nickname             |
| `itemId`     | String | Item ID                     |
| `count`      | Number | Amount to remove            |

**Returns:** `boolean` — `true` if the items were successfully removed.

**Example:**

```javascript
MCEngineServer.removeItem("Steve", "minecraft:diamond", 5);
```

---

## Managing Hostility

### makeHostile(entityType1, entityType2)

Makes two types of mobs hostile to each other via the `/team` command system.

**Parameters:**

| Parameter     | Type   | Description              |
|---------------|--------|--------------------------|
| `entityType1` | String | ID of the first mob type |
| `entityType2` | String | ID of the second mob type|

**Example:**

```javascript
// Zombies and skeletons now attack each other
MCEngineServer.makeHostile("minecraft:zombie", "minecraft:skeleton");
```

---

## Cleanup

### clearAllModifications()

Clears all AI modifications, model replacements, and callbacks. Useful when reloading scripts.

**Example:**

```javascript
MCEngineServer.clearAllModifications();
```

---

## Full Example

Here is a complete example of a script that creates a custom boss with unique behavior:

```javascript
MCEngineServer.onInit(() => {
    console.log("Boss script loaded!");
});

// Spawn boss 5 seconds after launch
var tickCounter = 0;

MCEngineServer.onServerTick(() => {
    tickCounter++;
    
    if (tickCounter === 100) { // 5 seconds (100 ticks)
        var boss = MCEngineServer.createCustomMob("minecraft:zombie", 0, 100, 0, {
            name: "§c§lANCIENT GUARDIAN",
            health: 500.0,
            scale: 1.5,
            
            onTick: (mob) => {
                // Find the nearest player
                var player = mob.findNearestPlayer(30);
                
                if (player != null) {
                    // Move towards the player
                    mob.moveTo(player.getX(), player.getY(), player.getZ(), 0.4);
                    
                    // If close — attack
                    if (mob.distanceToPlayer(player) < 2) {
                        mob.attackTarget(player);
                    }
                }
                
                // Ignite ourselves at low HP
                if (mob.getHealth() < 100) {
                    mob.setOnFire(3);
                }
            },
            
            onDeath: (mob) => {
                console.log("Guardian defeated!");
                
                // Reward the nearest player
                var player = mob.findNearestPlayer(50);
                if (player != null) {
                    MCEngineServer.giveItem(player.getName(), "minecraft:diamond", 10);
                    player.sendMessage("§aYou defeated the Guardian! +10 diamonds");
                }
            }
        });
        
        console.log("Boss spawned!");
    }
});
```