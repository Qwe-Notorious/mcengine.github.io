
# Client API (MCEngineClient)

!!! warning "Status: Under Active Development"
    The Client API (`MCEngineClient`) is currently in the active design and implementation phase. Since I am working on MCEngine solo, I am rolling out features incrementally. The methods described on this page are planned and may be adjusted before the final release.

---

## Overview

Unlike the Server API, which I have already implemented for managing the world and mobs, the **Client API** will be responsible for everything that happens exclusively on the player's side (the client). This is necessary to create features that the server cannot or should not handle, in order to avoid unnecessary network load and constant synchronization.

### Main areas I am working on:
1. **Custom Interface (HUD):** The ability to render custom widgets, resource bars, or unique elements over the standard Minecraft interface.
2. **Visual and Audio Effects:** Local spawning of particles and playing sounds exclusively for a specific player.
3. **Input Handling:** Registering custom keybinds and handling mouse events directly from JavaScript.
4. **Client-Side Rendering:** Dynamic real-time modification of model and texture rendering.

---

## Future Updates Roadmap

Below is a list of features I plan to add to `MCEngineClient` in upcoming updates.

### 1. Interface (HUD) Handling
| Planned Method | Description |
| :---------------- | :------- |
| `registerWidget(id, renderCallback)` | Allows registering a custom widget that will be rendered every frame over the game screen. |
| `removeWidget(id)` | Allows removing a registered widget from the screen. |

### 2. Particles and Sounds
| Planned Method | Description |
| :---------------- | :------- |
| `spawnParticle(particleId, x, y, z, count)` | Creates a visual particle effect on the client (will not be visible to other players unless synchronized by the server). |
| `playSound(soundId, volume, pitch)` | Plays a sound locally for the player, ignoring server-side distance limitations. |

### 3. Controls and Input
| Planned Method | Description |
| :---------------- | :------- |
| `registerKeyBind(keyName, keyCode, callback)` | Creates a new key in Minecraft's control settings and binds a JS function to it. |
| `isKeyPressed(keyCode)` | Allows checking if a specific key is currently being held down. |

---

## Conceptual Usage Example

*Note: This code is not yet functional. It demonstrates the intended syntax I am currently working on.*

```javascript
// Register a custom key "Mod Action" (default 'K')
MCEngineClient.registerKeyBind("my_mod_action", "key.keyboard.k", () => {
    
    // 1. Play a sound only for this player
    MCEngineClient.playSound("minecraft:entity.experience_orb.pickup", 1.0, 1.0);
    
    // 2. Create a particle effect around the player
    var player = MCEngineClient.getPlayer(); // Get the local player
    MCEngineClient.spawnParticle(
        "minecraft:flame", 
        player.getX(), 
        player.getY() + 1, 
        player.getZ(), 
        20 // Number of particles
    );
    
    // 3. Display a message in the Action Bar
    MCEngineClient.sendActionBar("§eMagical ability activated!");
});
```

---

## How You Can Influence Development

Since I am developing this project solo, your feedback and ideas are critically important to me. If you need a specific feature in the Client API that is not on this list:

1. Go to the [GitHub Issues](https://github.com/) section.
2. Create a new ticket with the label **`client-api-request`** (or another appropriate label for different APIs).
3. Describe the specific task or problem you want to solve with this feature.

I regularly review these requests and add the most requested and interesting ideas to my priority development roadmap.

---

## Related Sections

- [**Server API**](api-reference-server.en.md) — Fully functional API for managing server-side logic.
- [**Mob Wrapper (MobWrapper)**](mob-wrapper.en.md) — Methods for interacting with entities.
- [**Player Wrapper (PlayerWrapper)**](player-wrapper.en.md) — Methods for interacting with players.