# Distance Scripting Framework

A powerful JavaScript scripting framework for Minecraft that lets you create custom HUDs, movement tools, combat utilities, world scanners, and much more.

### Click here to get started: [Getting Started](getting-started.md)

## Features

- **Custom HUDs & Overlays** — Build personalized on-screen displays with text, shapes, images, and progress bars
- **Event System** — Hook into tick, render, attack, chat, input, inventory, and world events
- **2D & 3D Rendering** — Full suite of drawing primitives for both screen-space and world-space
- **Entity & Block Queries** — Scan nearby entities/players, get equipment, look up block types
- **GUI Settings** — Toggles, sliders, color pickers, mode selectors, text inputs, and buttons
- **Persistent Storage** — Save and load data across game sessions
- **Script Communication** — Broadcast events between multiple loaded scripts
- **Web Requests** — Async HTTP GET/POST with custom headers, image downloading
- **Clipboard Access** — Read from and write to the system clipboard
- **Timer API** — `setTimeout` / `setInterval` / `clearInterval` just like in a browser
- **Color Utilities** — Build colors, interpolate, parse hex, chroma/rainbow colors
- **Math Helpers** — `lerp`, `clamp`, `distance2D/3D`

## Quick Start

### Installation

1. Place your scripts in: `config/DistanceMod/scripts/`
2. Scripts must use the `.dscript` extension
3. Enable scripts through the Distance GUI in-game (Scripts tab)

### Script Metadata

Add metadata comments at the top of your script:

```javascript
//@author YourName
//@version 1.0
//@description A short description shown in the script list
```

### Hello World

```javascript
//@author Me
//@version 1.0
//@description My first script

Distance.log("Script loaded!");
Distance.chat("&aHello from Distance!");

Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    Distance.drawTextShadow("Hello World!", 10, 10, Distance.getChromaColor());
});
```

## Documentation

| Doc | Description |
|-----|-------------|
| [Getting Started](getting-started.md) | Installation, first script, basic concepts |
| [API Reference](api-reference.md) | Every function documented with examples |
| [Event System](events.md) | All events and best practices |
| [Rendering Guide](rendering.md) | 2D/3D rendering in depth |
| [GUI Settings](gui-settings.md) | Building config panels |
| [Advanced Features](advanced-features.md) | Storage, web, colors, communication |
| [Best Practices](best-practices.md) | Performance tips and patterns |
| [Examples](examples.md) | Ready-to-use example scripts |
| [Available Icons](available-icons.md) | All icons for your tab |

## Quick Reference

### Control
```javascript
Distance.log(msg)          // Console output
Distance.chat(msg)         // Chat (supports &color codes)
Distance.error(msg)        // Red error in chat
Distance.disable()         // Stop this script
Distance.reset()           // Reload this script
Distance.scriptName        // This script's filename
```

### Events
```javascript
Distance.on("tick", fn)            // 20x/sec game logic
Distance.on("render2d", fn)        // HUD rendering
Distance.on("render3d", fn)        // World-space rendering
Distance.on("attack", fn(target))  // Player attacked entity
Distance.on("chatReceive", fn(msg))
Distance.on("key", fn(keyCode))
Distance.on("click", fn(button))
Distance.on("itemPickup", fn(item))
Distance.on("itemDrop", fn(item))
Distance.on("chestOpen", fn(items))
Distance.on("chestClose", fn)
Distance.on("inventoryOpen", fn)
Distance.on("inventoryClose", fn)
Distance.on("worldChange", fn)
```

### Player
```javascript
Distance.getPlayerX/Y/Z()          // Position
Distance.getMotionX/Y/Z()          // Velocity
Distance.getSpeed()                 // Horizontal speed (blocks/tick)
Distance.getYaw() / getPitch()      // Rotation
Distance.getHealth() / getMaxHealth()
Distance.getAbsorption()
Distance.getFoodLevel() / getSaturation()
Distance.getArmor() / getXpLevel() / getXpProgress()
Distance.getFOV() / getGameMode()
Distance.getName() / getUUID()
Distance.isOnGround() / isFlying() / isCreative()
Distance.isSneaking() / isSprinting()
Distance.isInWater() / isInLava() / isInLiquid()
Distance.isBurning() / isAlive() / isInvisible()
Distance.hasPotionEffect("speed")
Distance.getPotionAmplifier("strength")
```

### World
```javascript
Distance.isDay() / isRaining() / isThundering()
Distance.getDimension()            // 0=OW, -1=Nether, 1=End
Distance.getWorldTime()            // 0–23999
Distance.getMoonPhase()            // 0–7
Distance.getBiome(x, y, z)
Distance.getLightLevel(x, y, z)
Distance.getFPS()
Distance.currentServerIP() / currentServerName()
Distance.getServerPing()
```

### Entities & Blocks
```javascript
Distance.getEntities()
Distance.getPlayers()
Distance.getNearbyEntities(radius)
Distance.getNearbyPlayers(radius)
Distance.getClosestPlayer()
Distance.getEntityById(id)
Distance.getEquipment(entityId)
Distance.getTargetEntity()
Distance.getTargetedBlock()        // { x, y, z, side }
Distance.getBlockAt(x, y, z)      // { id, name, meta, isAir }
Distance.scanBlocks(x, y, z, r)
```

### Rendering 2D
```javascript
Distance.drawText(text, x, y, color)
Distance.drawTextShadow(text, x, y, color)
Distance.drawTextCentered(text, x, y, color)
Distance.drawTextWithBorder(text, x, y, color, border)
Distance.drawRect(x, y, w, h, color)
Distance.drawRoundedRect(x, y, w, h, r, color)
Distance.drawBorderedRect(x, y, w, h, r, fill, border)
Distance.drawLine(x1, y1, x2, y2, color, width)
Distance.drawGradientRect(x, y, w, h, colorTop, colorBottom)
Distance.drawCircle(x, y, radius, color)
Distance.drawRing(x, y, radius, thickness, color)
Distance.drawArc(x, y, radius, startDeg, endDeg, color, lw)
Distance.drawProgressBar(x, y, w, h, progress, bg, fg)
Distance.scissorStart(x, y, w, h) / scissorEnd()
Distance.drawImage(id, x, y, w, h)
Distance.drawResource(path, x, y, w, h)
```

### Rendering 3D
```javascript
Distance.drawOutline(entity, color, lw)
Distance.drawOutline(x, y, z, color, lw)
Distance.drawFilledBox(x, y, z, color)
Distance.drawFilledBox(x, y, z, w, h, d, color)
Distance.drawLine3D(x1,y1,z1, x2,y2,z2, color)
Distance.drawSphere(x, y, z, r, slices, stacks, color)
Distance.drawPyramid(x, y, z, w, h, color)
Distance.worldToScreen(x, y, z)    // → { x, y, z, visible }
```

### Colors
```javascript
Distance.rgb(r, g, b)
Distance.argb(a, r, g, b)
Distance.parseColor("#RRGGBB")
Distance.resolveColor(colorString)  // handles Chroma/Multi too
Distance.lerpColor(colorA, colorB, t)
Distance.getChromaColor()
Distance.getChromaColor(speed)
```

### Storage & Clipboard
```javascript
Distance.saveData(key, value)
Distance.loadData(key, default)
Distance.loadDataDouble(key, default)
Distance.loadDataBool(key, default)
Distance.deleteData(key)
Distance.copyToClipboard(text)
Distance.getClipboard()
```

### Timing
```javascript
Distance.setTimeout(fn, ms)        // → task ID
Distance.setInterval(fn, ms)       // → task ID
Distance.clearTimeout(id)
Distance.clearInterval(id)
Distance.millis()                   // current time ms
Distance.nanos()                    // high-res nanoseconds
```

### Script Communication
```javascript
Distance.emit(event, data)          // broadcast to all scripts
Distance.getScriptNames()           // list of enabled scripts
```

---

**Powered by Nashorn JavaScript Engine**
