# Getting Started

Welcome to Distance scripting! This guide will get you from zero to a working script.

## Prerequisites

- Minecraft with Distance Mod installed
- Basic JavaScript knowledge (helpful but not required)

## File Setup

Scripts live in:
```
config/DistanceMod/scripts/
```

They must end in `.dscript`. Anything else is ignored.

## Script Metadata

You can tag your script with metadata that appears in the Distance script list:

```javascript
//@author YourName
//@version 1.2
//@description A short description of what this does
```

These are optional but recommended for any script you share.

## Your First Script

Create `MyFirstScript.dscript`:

```javascript
//@author Me
//@version 1.0
//@description Hello world example

Distance.log("Script loaded!");
Distance.chat("&aHello from Distance!");

Distance.on("tick", function() {
    // Runs 20 times per second
});

Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    Distance.drawTextShadow("Hello World!", 10, 10, 0xFF55FF55);
});
```

## Loading Your Script

1. Place the `.dscript` file in `config/DistanceMod/scripts/`
2. Open the Distance GUI in-game
3. Navigate to the **Scripts** tab
4. Toggle your script ON
5. To reload after editing, toggle it OFF then ON again

## Core Concepts

### The Distance Object

Every script gets a global `Distance` object with all the API methods. The `mc` variable (the raw Minecraft client) is also available.

### Script Name

```javascript
Distance.chat("I am: " + Distance.scriptName);
```

### Event-Driven Design

Almost everything in Distance works through events. Register handlers at load time:

```javascript
Distance.on("tick", function() { /* 20x/sec */ });
Distance.on("render2d", function() { /* HUD rendering */ });
Distance.on("attack", function(target) { /* player attacked something */ });
```

### Safety Checks

Always check that the player and world exist before accessing them:

```javascript
Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    // safe to use player APIs here
});
```

## Simple Examples

### Example 1: Coordinates HUD

```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    var x = Distance.getPlayerX().toFixed(1);
    var y = Distance.getPlayerY().toFixed(1);
    var z = Distance.getPlayerZ().toFixed(1);
    Distance.drawTextShadow("XYZ: " + x + " " + y + " " + z, 10, 10, 0xFFFFFF);
});
```

### Example 2: Speed Meter

```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    var speed = Distance.getSpeed() * 20; // blocks per second
    var color = speed > 6 ? 0xFF55FF55 : 0xFFFFFFFF;
    Distance.drawTextShadow("Speed: " + speed.toFixed(2) + " b/s", 10, 10, color);
});
```

### Example 3: Auto Jump

```javascript
Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    if (Distance.isOnGround() && Distance.isKeyDown("W")) {
        Distance.setMotion(Distance.getMotionX(), 0.42, Distance.getMotionZ());
    }
});
```

### Example 4: Persistent Toggle

```javascript
//@description Toggle example with persisted state

var enabled = Distance.loadDataBool("enabled", false);

Distance.createTab("My Script", "star");
Distance.addToggle("Enable", enabled, function(value) {
    enabled = value;
    Distance.saveData("enabled", value);
    Distance.chat(value ? "&aEnabled!" : "&cDisabled!");
});

Distance.on("render2d", function() {
    if (!enabled || Distance.isPlayerNull()) return;
    Distance.drawTextShadow("Active", 10, 10, 0xFF55FF55);
});
```

## Color Codes

Use `&` in `Distance.chat()` messages:

| Code | Effect |
|------|--------|
| `&a` | Green |
| `&c` | Red |
| `&e` | Yellow |
| `&b` | Aqua |
| `&f` | White |
| `&l` | Bold |
| `&r` | Reset |

Full table in [API Reference](api-reference.md#core-functions).

## Debugging

```javascript
Distance.log("Value: " + myVar);        // goes to console
Distance.chat("&e[Debug] " + myVar);    // goes to in-game chat
```

For on-screen debug display:

```javascript
var debug = true;

Distance.on("render2d", function() {
    if (!debug || Distance.isPlayerNull()) return;
    Distance.drawTextShadow("HP: " + Distance.getHealth(), 10, 10, 0xFFFFFF);
    Distance.drawTextShadow("OnGround: " + Distance.isOnGround(), 10, 22, 0xFFFFFF);
    Distance.drawTextShadow("Speed: " + Distance.getSpeed().toFixed(3), 10, 34, 0xFFFFFF);
});
```

## Common Issues

**Script doesn't load** — Check the console for a syntax error. Verify the file ends in `.dscript` and is in the right folder.

**Events don't fire** — Make sure the event name is spelled correctly and the script is toggled ON.

**Crashes with null errors** — Always guard with `Distance.isPlayerNull()` / `Distance.isWorldNull()` before accessing player/world data.

**Settings don't persist** — Use `Distance.saveData()` / `Distance.loadData()` to save values across sessions.

## Next Steps

- [API Reference](api-reference.md) — Every function, with examples
- [Event System](events.md) — All events and when to use them
- [Rendering Guide](rendering.md) — 2D and 3D drawing
- [GUI Settings](gui-settings.md) — Config panels
- [Advanced Features](advanced-features.md) — Storage, web requests, script communication
- [Examples](examples.md) — Complete ready-to-use scripts
- [Best Practices](best-practices.md) — Performance tips
