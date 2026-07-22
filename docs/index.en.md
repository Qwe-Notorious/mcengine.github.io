# MCEngine

**MCEngine** is a powerful Fabric mod that integrates a full-fledged JavaScript engine based on GraalVM into Minecraft. 
It allows you to create complex game logic, custom mobs with unique AI, and dynamic events without the need to write and compile Java code.

---

## Features

<div class="cards" markdown>

-   **Advanced AI and Hooks**

    Intercept ticks, attacks, damage taken, and mob deaths. Control their behavior via JS callbacks or disable the vanilla AI entirely.
    
    ---

-   **Custom Mobs and Bosses**

    Create unique creatures with specified health, names, models, and behavior scripts directly from the chat or a startup script.

    ---

-   **Dynamic Model Swapping**

    Replace vanilla textures and mob models with custom ones (globally or for a specific instance) via the Resource Pack overrides system.
    
    ---

-   **Built-in Code Editor**

    Edit scripts directly inside the game. The built-in editor supports a file tree, tabs, and syntax highlighting.

    ---

-   **Player Management**

    Give items, teleport, change game modes, set spawn points, and send messages to chat or the Action Bar.

    ---

-   **High Performance**

    Thanks to GraalVM Polyglot, JavaScript executes directly within the JVM, ensuring fast access to the Minecraft API and security.

</div>

---

## Quick Start
- Installation: Download the latest version of the mod and place the .jar file into the mods folder of your client/server.
- Creating a Project: Launch the game, open the main menu (or pause menu), and press the MCEngine button. Create a new project.
- Writing Code: Open the built-in code editor, create a main.js file, and start writing scripts.
- Reloading: Use the /mcengine reload command to apply changes without restarting the game.

!!! tip "Tip"
    You can execute raw JavaScript code directly from the chat using the command:
    /mcengine run console.log("Hello MCEngine!");

## Code Example

Here is how easily you can change a mob's behavior:

```javascript

// Modifying mob AI
MCEngineServer.modifyAI("minecraft:zombie", {
    onTick: (mob) => {
        var player = mob.findNearestPlayer(15);
        if (player != null) {
            var dx = mob.getX() - player.getX();
            var dz = mob.getZ() - player.getZ();
            var dist = Math.sqrt(dx * dx + dz * dz);
            if (dist > 0) {
                mob.moveTo(mob.getX() + (dx / dist) * 5, mob.getY(), mob.getZ() + (dz / dist) * 5, 1.2);
            }
        }
    },
    
    onDeath: (mob) => {
        mob.spawnParticle("minecraft:large_smoke", 1, 10);
    }    
    
});
console.log("Cowardly zombie !!!")

```

## Navigation

-   [Guide](guide/introduction.en.md)
    
    Learn how to set up the environment and write your first script.

-   [API Reference](api/api-reference-server.en.md)
    
    Full description of all available methods, hooks, and classes.

-   [GitHub](https://github.com/)
    
    Mod source code and bug tracker.
