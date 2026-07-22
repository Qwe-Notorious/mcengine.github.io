
# Mob Wrapper (MobWrapper)

`MobWrapper` is a safe wrapper over Minecraft entities (`Entity`, `LivingEntity`, `MobEntity`), designed for use in JavaScript. It is adapted to the Minecraft 1.21.11 API and provides convenient access to mob properties and methods without the need to work with low-level Java code.

!!! note "Note"
    A `MobWrapper` instance is created automatically when a mob is passed to JS callbacks (e.g., in `onTick`, `onDeath`) or when using search and spawn methods.

---

## Basic Information

### getType()
Returns the untranslated entity type name.

**Returns:** `String` (e.g., `"Zombie"`).

### getFullType()
Returns the full entity type identifier from the registry.

**Returns:** `String` (e.g., `"minecraft:zombie"`).

### getUuid()
Returns the unique entity identifier.

**Returns:** `String` (UUID in string format).

### getName()
Returns the current display name of the entity.

**Returns:** `String`.

### isAlive()
Checks if the entity is alive.

**Returns:** `boolean`.

---

## Position and Movement

### getX(), getY(), getZ()
Return the current coordinates of the entity.

**Returns:** `Number` (double).

### getYaw(), getPitch()
Return the current rotation angle (yaw and pitch) of the entity.

**Returns:** `Number` (float).

### setPosition(x, y, z)
Instantly moves the entity to the specified coordinates without animation.

**Parameters:**

| Parameter | Type   | Description        |
| :-------- | :----- | :----------------- |
| `x, y, z` | Number | Target coordinates |

### teleport(x, y, z)
Teleports the entity. 

!!! info "Implementation Details"
    For players (`ServerPlayerEntity`), the safe `requestTeleport` method is used. For other entities, the `teleport` method bound to `ServerWorld` is used, which prevents errors in Minecraft 1.21+.

**Parameters:**

| Parameter | Type   | Description        |
| :-------- | :----- | :----------------- |
| `x, y, z` | Number | Target coordinates |

### moveTo(x, y, z, speed)
Forces the mob to walk to the specified coordinates independently, using Minecraft's built-in navigator.

!!! warning "Warning"
    Works only for entities inheriting from `MobEntity`. For players or other entities, it will return `false`.

**Parameters:**

| Parameter | Type   | Description                             |
| :-------- | :----- | :-------------------------------------- |
| `x, y, z` | Number | Target coordinates                      |
| `speed`   | Number | Movement speed (usually from 0.1 to 1.0)|

**Returns:** `boolean` (whether movement was successfully initiated).

### stopMoving()
Immediately stops the mob's movement.

---

## Health and Damage

### getHealth()
Returns the current health of the entity.

**Returns:** `Number` (float). Returns `0` if the entity is not a `LivingEntity`.

### setHealth(health)
Sets the entity's health.

**Parameters:**

| Parameter | Type   | Description         |
| :-------- | :----- | :------------------ |
| `health`  | Number | New health value    |

### getMaxHealth()
Returns the maximum health of the entity.

**Returns:** `Number` (float).

### kill()
Instantly kills the entity.

!!! info "Implementation Details"
    In Minecraft 1.21+, the `kill()` method requires passing `ServerWorld`. The wrapper does this automatically.

---

## Name and Appearance

### setName(name)
Sets a custom name for the entity (supports Minecraft color codes, e.g., `"§cRed"`).

**Parameters:**

| Parameter | Type   | Description |
| :-------- | :----- | :---------- |
| `name`    | String | New name    |

### setNameVisible(visible)
Makes the custom name visible or hidden above the entity's head.

**Parameters:**

| Parameter | Type    | Description                          |
| :-------- | :------ | :----------------------------------- |
| `visible` | Boolean | `true` to display, `false` to hide   |

### setInvisible(invisible)
Makes the entity invisible.

**Parameters:**

| Parameter   | Type    | Description           |
| :---------- | :------ | :-------------------- |
| `invisible` | Boolean | `true` for invisibility |

### setSilent(silent)
Disables all sounds made by the entity.

**Parameters:**

| Parameter | Type    | Description                |
| :-------- | :------ | :------------------------- |
| `silent`  | Boolean | `true` to disable sounds   |

### setNoGravity(noGravity)
Disables gravity for the entity (it will float).

**Parameters:**

| Parameter   | Type    | Description                |
| :---------- | :------ | :------------------------- |
| `noGravity` | Boolean | `true` to disable gravity  |

---

## Targets and Attacks

### setTarget(target)
Forces the mob to choose a new target to attack.

**Parameters:**

| Parameter | Type       | Description            |
| :-------- | :--------- | :--------------------- |
| `target`  | MobWrapper | Target entity wrapper  |

### getTarget()
Returns the mob's current attack target.

**Returns:** `MobWrapper` (target) or `null` if there is no target.

### attackTarget(target)
Forces the mob to immediately execute the attack animation and logic on the specified target.

!!! info "Implementation Details"
    In Minecraft 1.21+, the `tryAttack` method requires passing `ServerWorld`. The wrapper does this automatically.

**Parameters:**

| Parameter | Type       | Description            |
| :-------- | :--------- | :--------------------- |
| `target`  | MobWrapper | Target entity wrapper  |

**Returns:** `boolean` (whether the attack was successful).

---

## Entity Search

### findNearestPlayer(radius)
Searches for the nearest player within a given radius.

**Parameters:**

| Parameter | Type   | Description             |
| :-------- | :----- | :---------------------- |
| `radius`  | Number | Search radius in blocks |

**Returns:** `PlayerWrapper` (nearest player) or `null` if no players are within the radius.

### findNearestMob(entityType, radius)
Searches for the nearest living mob of the specified type within a given radius.

**Parameters:**

| Parameter    | Type   | Description                                 |
| :----------- | :----- | :------------------------------------------ |
| `entityType` | String | Mob type (e.g., `"minecraft:zombie"`)       |
| `radius`     | Number | Search radius in blocks                     |

**Returns:** `MobWrapper` or `null`.

### findAllMobs(entityType, radius)
Finds all living mobs of the specified type within a given radius.

**Parameters:**

| Parameter    | Type   | Description                         |
| :----------- | :----- | :---------------------------------- |
| `entityType` | String | Mob type                            |
| `radius`     | Number | Search radius in blocks             |

**Returns:** `Array` (array of `MobWrapper` objects).

### distanceTo(other)
Calculates the distance to another entity.

**Parameters:**

| Parameter | Type       | Description      |
| :-------- | :--------- | :--------------- |
| `other`   | MobWrapper | Target entity    |

**Returns:** `Number` (distance in blocks). Returns `-1` if `other` is `null`.

### distanceToPlayer(player)
Calculates the distance to a player.

**Parameters:**

| Parameter | Type          | Description   |
| :-------- | :------------ | :------------ |
| `player`  | PlayerWrapper | Target player |

**Returns:** `Number` (distance in blocks). Returns `-1` if `player` is `null`.

---

## Special Abilities

### addEffect(effectId, duration, amplifier)
Applies a potion effect to the entity.

!!! info "Implementation Details"
    In Minecraft 1.21+, adding effects requires `RegistryEntry`. The wrapper automatically converts the string ID to the required format.

**Parameters:**

| Parameter   | Type   | Description                                          |
| :---------- | :----- | :--------------------------------------------------- |
| `effectId`  | String | Effect ID (e.g., `"minecraft:speed"`)                |
| `duration`  | Number | Duration in ticks (20 ticks = 1 second)              |
| `amplifier` | Number | Effect level (0 = level I, 1 = level II)             |

### setOnFire(seconds)
Sets the entity on fire for the specified number of seconds.

!!! info "Implementation Details"
    In Minecraft 1.21.11, the method accepts a `float` value.

**Parameters:**

| Parameter | Type   | Description               |
| :-------- | :----- | :------------------------ |
| `seconds` | Number | Number of seconds to burn |

### playSound(soundId, volume, pitch)
Plays a sound at the entity's current position.

**Parameters:**

| Parameter | Type   | Description                                             |
| :-------- | :----- | :------------------------------------------------------ |
| `soundId` | String | Sound ID (e.g., `"minecraft:entity.zombie.ambient"`)    |
| `volume`  | Number | Volume (usually 1.0)                                    |
| `pitch`   | Number | Pitch (usually 1.0)                                     |

---

## Complex Example: Smart Guardian

Below is an example of how `MobWrapper` methods are used together to create intelligent behavior:

```javascript
MCEngineServer.onInit(() => {
    var boss = MCEngineServer.createCustomMob("minecraft:zombie", 0, 100, 0, {
        name: "§c§lDUNGEON GUARDIAN",
        health: 300.0,
        
        onTick: (mob) => {
            // 1. Find the nearest player within a 30-block radius
            var player = mob.findNearestPlayer(30);
            
            if (player != null) {
                var dist = mob.distanceToPlayer(player);
                
                // 2. If the player is far, move towards them
                if (dist > 3.0) {
                    mob.moveTo(player.getX(), player.getY(), player.getZ(), 0.4);
                } 
                // 3. If the player is close, attack them
                else {
                    mob.attackTarget(player);
                }
                
                // 4. If health drops below 30%, set ourselves on fire and speed up
                if (mob.getHealth() < 90.0) { // 30% of 300
                    mob.addEffect("minecraft:speed", 40, 1); // Speed II for 2 seconds
                    mob.setOnFire(3);
                }
            } else {
                // If there are no players, stop moving
                mob.stopMoving();
            }
        },
        
        onDeath: (mob) => {
            // Play a sound on death
            mob.playSound("minecraft:entity.wither.death", 1.0, 0.8);
            
            // Find all zombies within a 10-block radius and kill them (death explosion effect)
            var minions = mob.findAllMobs("minecraft:zombie", 10);
            for (var i = 0; i < minions.length; i++) {
                minions[i].kill();
            }
            
            console.log("Guardian defeated, its minions destroyed!");
        }
    });
});
```