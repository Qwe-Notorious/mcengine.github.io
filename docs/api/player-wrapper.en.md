
# Player Wrapper (PlayerWrapper)

`PlayerWrapper` is a safe wrapper over `ServerPlayerEntity`, designed for convenient player management from JavaScript. The class is adapted to the Minecraft 1.21.11 API and hides complex low-level calls, providing a simple and intuitive interface.

!!! note "Note"
    A `PlayerWrapper` instance is returned by the `MCEngineServer.getPlayer()`, `MCEngineServer.getAllPlayers()` methods, or when mobs search using `mob.findNearestPlayer()`.

---

## Basic Information

### getName()
Returns the player's nickname.

**Returns:** `String`.

### getUuid()
Returns the player's unique identifier.

**Returns:** `String` (UUID in string format).

### getX(), getY(), getZ()
Return the player's current coordinates.

**Returns:** `Number` (double).

### getHealth(), setHealth(health)
Gets or sets the player's current health.

**`setHealth` Parameters:**

| Parameter | Type   | Description         |
| :-------- | :----- | :------------------ |
| `health`  | Number | New health value    |

### getMaxHealth()
Returns the player's maximum health.

**Returns:** `Number` (float).

### getFoodLevel(), setFoodLevel(level)
Gets or sets the player's food (hunger) level (from 0 to 20).

**`setFoodLevel` Parameters:**

| Parameter | Type   | Description             |
| :-------- | :----- | :---------------------- |
| `level`   | Number | Food level (0-20)       |

### getXpLevel(), setXpLevel(level)
Gets or sets the player's current experience level.

**`setXpLevel` Parameters:**

| Parameter | Type   | Description      |
| :-------- | :----- | :--------------- |
| `level`   | Number | New XP level     |

---

## Messages

### sendMessage(message)
Sends a text message to the player's chat.

**Parameters:**

| Parameter | Type   | Description     |
| :-------- | :----- | :-------------- |
| `message` | String | Message text    |

### sendActionBar(message)
Sends a message to the Action Bar (above the hotbar, disappears after a few seconds).

**Parameters:**

| Parameter | Type   | Description     |
| :-------- | :----- | :-------------- |
| `message` | String | Message text    |

### sendColoredMessage(message)
Sends a message to the chat supporting Minecraft color codes (e.g., `"§aSuccess!"`).

**Parameters:**

| Parameter | Type   | Description                     |
| :-------- | :----- | :------------------------------ |
| `message` | String | Text with color codes (`§`)     |

---

## Inventory

### giveItem(itemId, count)
Gives the specified amount of items to the player.

!!! info "Implementation Details"
    In Minecraft 1.21.11, the `insertStack` method is used instead of the deprecated `giveItemStack` for correct item insertion.

**Parameters:**

| Parameter | Type   | Description                                 |
| :-------- | :----- | :------------------------------------------ |
| `itemId`  | String | Item ID (e.g., `"minecraft:diamond"`)       |
| `count`   | Number | Amount of items                             |

**Returns:** `boolean` (whether the items were successfully added).

### hasItem(itemId, minCount)
Checks if the player has the minimum amount of the specified item (sums stacks).

**Parameters:**

| Parameter  | Type   | Description                     |
| :--------- | :----- | :------------------------------ |
| `itemId`   | String | Item ID                         |
| `minCount` | Number | Minimum required amount         |

**Returns:** `boolean`.

### removeItem(itemId, count)
Takes the specified amount of items from the player's inventory (starting from the first slots).

**Parameters:**

| Parameter | Type   | Description          |
| :-------- | :----- | :------------------- |
| `itemId`  | String | Item ID              |
| `count`   | Number | Amount to remove     |

**Returns:** `boolean` (whether the entire requested amount was successfully removed).

### getItemCount(itemId)
Returns the total amount of the specified item across all inventory slots.

**Parameters:**

| Parameter | Type   | Description |
| :-------- | :----- | :---------- |
| `itemId`  | String | Item ID     |

**Returns:** `Number` (int).

### clearInventory()
Completely clears the player's inventory (including armor and offhand).

---

## Movement and Teleportation

### teleport(x, y, z)
Teleports the player to the specified coordinates.

!!! info "Implementation Details"
    In Minecraft 1.21.11, the safe `requestTeleport` method is used for players, which correctly handles chunk loading and prevents movement errors.

**Parameters:**

| Parameter | Type   | Description        |
| :-------- | :----- | :----------------- |
| `x, y, z` | Number | Target coordinates |

### setPosition(x, y, z)
Instantly moves the player to the specified coordinates without teleportation animation.

**Parameters:**

| Parameter | Type   | Description        |
| :-------- | :----- | :----------------- |
| `x, y, z` | Number | Target coordinates |

---

## Abilities and State

### setGameMode(mode)
Sets the game mode for the player.

**Parameters:**

| Parameter | Type   | Description                                                                       |
| :-------- | :----- | :-------------------------------------------------------------------------------- |
| `mode`    | Number | `0` = Survival, `1` = Creative, `2` = Adventure, `3` = Spectator                  |

### getGameMode()
Returns the player's current game mode.

**Returns:** `String` (e.g., `"survival"`, `"creative"`).

### giveXp(amount)
Adds the specified amount of experience points to the player.

!!! info "Implementation Details"
    In Minecraft 1.21.11, the `addExperience` method is used instead of the deprecated `giveXp`.

**Parameters:**

| Parameter | Type   | Description               |
| :-------- | :----- | :------------------------ |
| `amount`  | Number | Amount of experience points|

### heal(amount)
Restores the player's health by the specified amount.

**Parameters:**

| Parameter | Type   | Description                  |
| :-------- | :----- | :--------------------------- |
| `amount`  | Number | Amount of health to restore  |

### isAlive()
Checks if the player is alive.

**Returns:** `boolean`.

---

## Spawn Point

### setSpawnPoint(x, y, z)
Sets the player's personal respawn point.

!!! info "Implementation Details"
    In Minecraft 1.21.11, a new API is used: a `SpawnPoint` object is created via `WorldProperties.SpawnPoint.create()` and wrapped in a `Respawn` record.

**Parameters:**

| Parameter | Type   | Description        |
| :-------- | :----- | :----------------- |
| `x, y, z` | Number | Spawn coordinates  |

### getSpawnPoint()
Returns the coordinates of the player's current respawn point.

**Returns:** `String` in the format `"x,y,z"` or `null` if no point is set.

### clearSpawnPoint()
Resets the player's personal respawn point (respawn will occur at the world spawn).

---

## Additional Features

### killWithReason(reason)
Kills the player with a specific death reason (displayed in chat, e.g., "Player burned").

!!! info "Implementation Details"
    Instead of calling `kill()` directly, the method uses `player.damage()` with a damage source (`DamageSource`) and a value of `Float.MAX_VALUE`, which guarantees death and correctly logs the reason.

**Parameters:**

| Parameter | Type   | Description (allowed values)                                                                                 |
| :-------- | :----- | :----------------------------------------------------------------------------------------------------------- |
| `reason`  | String | `"generic"`, `"fire"`, `"burn"`, `"lava"`, `"drown"`, `"water"`, `"fall"`, `"magic"`, `"starve"`, `"hunger"`, `"void"`, `"world"` |

### distanceTo(other)
Calculates the distance to another player.

**Parameters:**

| Parameter | Type          | Description   |
| :-------- | :------------ | :------------ |
| `other`   | PlayerWrapper | Target player |

**Returns:** `Number` (distance in blocks). Returns `-1` if `other` is `null`.

### distanceToMob(mob)
Calculates the distance to a mob.

**Parameters:**

| Parameter | Type       | Description  |
| :-------- | :--------- | :----------- |
| `mob`     | MobWrapper | Target mob   |

**Returns:** `Number` (distance in blocks). Returns `-1` if `mob` is `null`.

---

## Complex Example: Quest Completion System

Below is an example of how `PlayerWrapper` methods are used together to create quest reward logic:

```javascript
function completeQuest(playerName) {
    var player = MCEngineServer.getPlayer(playerName);
    
    if (player == null) {
        console.log("Player " + playerName + " not found (possibly offline).");
        return;
    }

    // 1. Check if the player met the condition (e.g., has 10 diamonds)
    if (player.hasItem("minecraft:diamond", 10)) {
        // 2. Take the diamonds
        player.removeItem("minecraft:diamond", 10);
        
        // 3. Give reward: XP, item, and message
        player.giveXp(100);
        player.giveItem("minecraft:netherite_sword", 1);
        player.sendColoredMessage("§a§l[QUEST] Congratulations! You received a Netherite Sword and 100 XP!");
        player.sendActionBar("§6Quest completed!");
        
        // 4. Teleport the player to the save point
        player.teleport(0, 100, 0);
        
        // 5. Set this point as their personal spawn
        player.setSpawnPoint(0, 100, 0);
        
        console.log("Player " + playerName + " successfully completed the quest.");
    } else {
        // If conditions are not met
        var currentDiamonds = player.getItemCount("minecraft:diamond");
        player.sendMessage("§cYou don't have enough diamonds. You have: " + currentDiamonds + " out of 10.");
    }
}

// Example call from a command or another script:
// completeQuest("Steve");
```