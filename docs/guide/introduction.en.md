# Introduction to MCEngine

**MCEngine** is a Fabric mod that adds a full-fledged JavaScript engine based on GraalVM Polyglot to Minecraft. It allows you to create complex game logic, custom mobs with unique AI, and dynamic events without having to write and compile Java code.

---

## How does it work?

MCEngine uses **GraalVM** to execute JavaScript code directly inside the JVM (Java Virtual Machine). This ensures:

- **High performance** — JS code runs as fast as Java
- **Secure API access** — scripts have access only to allowed methods through special "Bridges"
- **Two—way integration** - Java can call JS functions, and JS can call Java methods

### System architecture
````
    ┌───────────────────────────────────────────────────────────┐
    │                    Minecraft (Fabric)                     │
    ├───────────────────────────────────────────────────────────┤
    │                                                           │
    │   ┌──────────────┐            ┌───────────────────────┐   │
    │   │  Java Code   │◄──────────►│    GraalVM Context    │   │
    │   │  (MCEngine)  │            │    (JavaScript VM)    │   │
    │   └──────────────┘            └───────────────────────┘   │
    │           ▲                               ▲               │
    │           │                               │               │
    │           │                               ▼               │
    │           │                   ┌───────────────────────┐   │
    │           │                   │      JS Bridges       │   │
    │           │                   │  - MCEngineServer     │   │
    │           │                   │  - MCEngineClient     │   │
    │           │                   │  - console            │   │
    │           │                   └───────────────────────┘   │
    │           │                               ▲               │
    │           │                               │               │
    │           ▼                               │               │
    │   ┌──────────────┐                        │               │
    │   │  Minecraft   │◄───────────────────────┘               │
    │   │     API      │                                        │
    │   └──────────────┘                                        │
    │                                                           │
    └───────────────────────────────────────────────────────────┘
````

```text
    MCEngine_Project/
    └── MyProject/
        ├── project.json # Project metadata
        ├── code/
        ,── client/ # Client scripts
        │   │   └── main.js # Entry Point (client)
        │   └── server/       # Server scripts
        │       └── main.js # Entry point (server)
        ├── models/           # Custom models (.bbmodel)
        └── textures/         # Custom textures (.png)
```


### Client and server sides

MCEngine divides logic into two independent sides:

**Server side (`code/server/`):**
- Controls mobs, spawn, AI
- Handles game events (damage, death, ticks)
- Runs on a dedicated server and in single player mode
- API access: 'MCEngineServer`

**Client side (`code/client/`):**
- Responsible for HUD, particles, sounds
- Processes player input
- Works only on the client
- API access: 'MCEngineClient`

!!! warning "Important"
Client and server scripts are executed in different contexts. The server code does not have access to the client API and vice versa.

---

## Basic concepts

### 1. Entry point (main.js)

Each project must have a `main' file.js` in the folders `client/` and `server/'. This is the first file that is executed when the project is uploaded.

````javascript
// code/server/main.js
console.log("Server script loaded!");
MCEngineServer.onInit(() => {
    console.log("Server initialized");
});
````


### 2. Hooks and callbacks

MCEngine provides a system of hooks, functions that are called upon certain events.

````javascript
// Registering a callback for each server tick
MCEngineServer.onServerTick(() => {
// This code is executed 20 times per second
});

// Registering a callback for the death of a mob
MCEngineServer.modifyAI("minecraft:zombie", {
    onDeath: (mob) => {
        console.log("Zombie is dead!");
    }
});
````

### 3. Wrappers

To safely work with Minecraft objects from JavaScript, MCEngine uses wrapper classes.:
MobWrapper — wrapper over Entity/LivingEntity/MobEntity
PlayerWrapper — wrapper over ServerPlayerEntity
These wrappers provide a simplified API for working with entities.:

````javascript
var player = MCEngineServer.getPlayer("Steve");
player.SendMessage("Hello, " + player.getName() + "!");
player.giveItem("minecraft:diamond", 10);
````

###4. Custom mobs

One of the most powerful features is to create custom mobs with unique behavior.:

```javascript
var boss = MCEngineServer.createCustomMob("minecraft:zombie", 0, 100, 0, {
name: "
ccl boss", health: 500.0,
    scale: 1.5,
    
    onTick: (mob) => {
        // Logic of behavior every tick
    },
    
    onDeath: (mob) => {
        console.log("The boss is defeated!");
    }
});
```

## Teams

MCEngine adds some useful commands:

| Command                | Description                                |
|:-----------------------|:-------------------------------------------|
| `/mcengine reload`     | Reloads all scripts of the current project |
| `/mcengine run <code>` | Executes arbitrary JavaScript code         |

!!! tip "Tip"
    The `/mcengine run` command is useful for testing individual functions without having to create files:

````javascript
/mcengine run console.log("Test message")
````

## What's next?

Now that you understand the basics of the MCEngine architecture, go to the following sections:

* **[Your first script](first-script.md )** — Let's write a simple script that spawns mobs and reacts to events.
* **[API Reference](api-reference-server.en.md )** — Full description of all available methods and classes.

!!! info "Need help?"
    If you have any questions or problems, create a topic in the project repository:
    [Create an issue on GitHub](https://github.com )