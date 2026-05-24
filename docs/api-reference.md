# API Reference

Complete reference for all Distance API functions.

## Table of Contents

- [Core Functions](#core-functions)
- [Timing & Async](#timing--async)
- [Player State](#player-state)
- [Rotation & Look](#rotation--look)
- [World & Environment](#world--environment)
- [Input](#input)
- [Entities](#entities)
- [Blocks](#blocks)
- [Actions](#actions)
- [Inventory & Items](#inventory--items)
- [Rendering 2D](#rendering-2d)
- [Rendering 3D](#rendering-3d)
- [Projection](#projection)
- [Colors & Utilities](#colors--utilities)
- [Math Helpers](#math-helpers)
- [Clipboard](#clipboard)
- [Persistent Storage](#persistent-storage)
- [Script Communication](#script-communication)
- [Web Requests](#web-requests)
- [GUI Registration](#gui-registration)
- [Particles & Sound](#particles--sound)

---

## Core Functions

### `Distance.log(message)`
Prints to the console (terminal / F3 overlay).

```javascript
Distance.log("Player X: " + Distance.getPlayerX());
```

### `Distance.chat(message)`
Sends a message to the player's chat. Supports `&` color codes.

```javascript
Distance.chat("&aGreen &cRed &eYellow");
```

**Color codes:**

| Code | Color      | Code | Color       |
|------|------------|------|-------------|
| `&0` | Black      | `&8` | Dark Gray   |
| `&1` | Dark Blue  | `&9` | Blue        |
| `&2` | Dark Green | `&a` | Green       |
| `&3` | Dark Aqua  | `&b` | Aqua        |
| `&4` | Dark Red   | `&c` | Red         |
| `&5` | Dark Purple| `&d` | Light Purple|
| `&6` | Gold       | `&e` | Yellow      |
| `&7` | Gray       | `&f` | White       |

Format codes: `&l` Bold · `&n` Underline · `&r` Reset

### `Distance.error(message)`
Sends a red `[Script Error]` prefixed message to chat.

### `Distance.disable()`
Disables the current script.

### `Distance.reset()`
Reloads the current script from disk.

### `Distance.scriptName`
The filename of the running script (read-only property).

```javascript
Distance.chat("I am: " + Distance.scriptName);
```

---

## Timing & Async

### `Distance.setTimeout(task, delayMs)` → `int`
Runs `task` once after `delayMs` milliseconds. Returns a task ID you can pass to `clearTimeout`.

```javascript
var id = Distance.setTimeout(function() {
    Distance.chat("3 seconds later!");
}, 3000);
```

### `Distance.setInterval(task, intervalMs)` → `int`
Runs `task` repeatedly every `intervalMs` milliseconds. Returns a task ID.

```javascript
var pingId = Distance.setInterval(function() {
    Distance.chat("Ping: " + Distance.getServerPing() + "ms");
}, 5000);
```

### `Distance.clearTimeout(id)` / `Distance.clearInterval(id)` / `Distance.clearTimer(id)`
Cancels a pending or repeating task by its ID. All three are equivalent aliases.

```javascript
Distance.clearInterval(pingId);
```

### `Distance.millis()` → `long`
Current system time in milliseconds. Useful for cooldowns.

```javascript
var lastAttack = 0;
Distance.on("attack", function(target) {
    var now = Distance.millis();
    if (now - lastAttack < 500) return; // 500ms cooldown
    lastAttack = now;
    Distance.chat("Hit " + target.name);
});
```

### `Distance.nanos()` → `long`
High-precision elapsed time in nanoseconds. Use for benchmarking.

```javascript
var start = Distance.nanos();
// ... do work ...
Distance.log("Elapsed: " + ((Distance.nanos() - start) / 1e6).toFixed(2) + "ms");
```

---

## Player State

### Position & Motion

| Function | Returns | Description |
|---|---|---|
| `getPlayerX()` | double | Player X coordinate |
| `getPlayerY()` | double | Player Y coordinate |
| `getPlayerZ()` | double | Player Z coordinate |
| `getMotionX()` | double | X velocity |
| `getMotionY()` | double | Y velocity |
| `getMotionZ()` | double | Z velocity |
| `getSpeed()` | double | Horizontal speed in blocks/tick |

```javascript
var speed = Distance.getSpeed() * 20; // convert to blocks/sec
Distance.chat("Speed: " + speed.toFixed(2) + " b/s");
```

### `Distance.setMotion(x, y, z)`
Sets the player's velocity directly.

```javascript
// Boost upward
Distance.setMotion(Distance.getMotionX(), 0.5, Distance.getMotionZ());
```

### Boolean State Checks

| Function | Description |
|---|---|
| `isPlayerNull()` | True if player doesn't exist (menus, loading) |
| `isOnGround()` / `onGround()` | True if standing on a block |
| `isSneaking()` | True if sneaking |
| `isSprinting()` | True if sprinting |
| `isFlying()` | True if creative flying |
| `isCreative()` | True if in creative mode |
| `isAlive()` | True if player entity is alive |
| `isInvisible()` | True if player is invisible |
| `isBurning()` | True if player is on fire |
| `isInWater()` | True if in water |
| `isInLava()` | True if in lava |
| `isInLiquid()` | True if in water OR lava |
| `isWorldNull()` | True if no world is loaded |

### Stats

| Function | Returns | Description |
|---|---|---|
| `getHealth()` | float | Current health |
| `getMaxHealth()` | float | Maximum health |
| `getAbsorption()` | float | Absorption hearts |
| `getFoodLevel()` | int | Hunger level (0–20) |
| `getSaturation()` | float | Food saturation |
| `getArmor()` | int | Total armor value |
| `getXpLevel()` | int | Experience level |
| `getXpProgress()` | float | XP progress to next level (0.0–1.0) |
| `getFallDistance()` | float | Current fall distance |
| `getEyeHeight()` | float | Eye height offset |
| `getFOV()` | float | Current FOV setting |
| `getGameMode()` | int | Game mode ID (0=survival, 1=creative, 2=adventure, 3=spectator) |
| `getName()` | String | Player username |
| `getUUID()` | String | Player UUID |
| `getYaw()` | float | Horizontal rotation (-180 to 180) |
| `getPitch()` | float | Vertical rotation (-90 to 90) |

### Potion Effects

#### `Distance.hasPotionEffect(name)` → `boolean`
```javascript
if (Distance.hasPotionEffect("speed")) {
    Distance.chat("I'm zooming!");
}
```

#### `Distance.getPotionAmplifier(name)` → `int`
Returns the amplifier (0-based), or -1 if the effect is not active.

```javascript
var level = Distance.getPotionAmplifier("strength");
if (level >= 0) {
    Distance.chat("Strength " + (level + 1));
}
```

---

## Rotation & Look

### `Distance.setRotation(yaw, pitch)`
Sets the player's rotation directly (sent to server).

### `Distance.setSilentRotation(yaw, pitch)`
Sets rotation without sending the packet to the server.

### `Distance.lookAt(x, y, z)`
Rotates the player to look at a world position.

### `Distance.lookAtEntity(entity)`
Rotates the player to look at an entity's eye level.

### `Distance.normalizeYaw(yaw)` → `float`
Clamps a yaw value to the -180 to 180 range.

### `Distance.angleDiff(a, b)` → `float`
Returns the shortest angular difference between two yaw values, correctly handling wrap-around at ±180.

```javascript
var diff = Distance.angleDiff(Distance.getYaw(), targetYaw);
if (Math.abs(diff) < 5) {
    Distance.chat("Looking at target!");
}
```

---

## World & Environment

### Server & Performance

| Function | Returns | Description |
|---|---|---|
| `getFPS()` | int | Current frames per second |
| `currentServerIP()` | String | Current server IP, or "Singleplayer" |
| `currentServerName()` | String | Current server display name |
| `getServerPing()` | int | Ping in ms, or -1 in singleplayer |
| `getSystemTime()` | long | System time in ms (alias for `millis()`) |

### World Checks

| Function | Returns | Description |
|---|---|---|
| `isWorldNull()` | boolean | True if no world loaded |
| `isDay()` | boolean | True if world is daytime |
| `isRaining()` | boolean | True if raining |
| `isThundering()` | boolean | True if thunderstorm |
| `getWorldTime()` | long | World time ticks (0–23999) |
| `getDimension()` | int | Dimension ID (0=overworld, -1=nether, 1=end) |
| `getMoonPhase()` | int | Moon phase (0–7) |

### `Distance.getBiome(x, y, z)` → `String`
Returns the biome name at a position.

```javascript
var biome = Distance.getBiome(Distance.getPlayerX(), Distance.getPlayerY(), Distance.getPlayerZ());
Distance.chat("You're in: " + biome);
```

### `Distance.getLightLevel(x, y, z)` → `int`
Returns the block light level (0–15) at a position.

---

## Input

### Keyboard

#### `Distance.isKeyDown(keyCode)` / `Distance.isKeyDown(keyName)` → `boolean`

```javascript
if (Distance.isKeyDown("LSHIFT")) { /* sneaking */ }
if (Distance.isKeyDown(17)) { /* W key */ }
```

#### `Distance.getKeyCode(name)` → `int`
Converts a key name to its LWJGL key code.

#### `Distance.getKeyName(keyCode)` → `String`
Converts a key code to its name.

**Common Key Codes:**

| Key | Code | Key | Code |
|-----|------|-----|------|
| W | 17 | Space | 57 |
| A | 30 | Left Shift | 42 |
| S | 31 | Left Ctrl | 29 |
| D | 32 | Left Alt | 56 |
| 1–9 | 2–10 | Up Arrow | 200 |
| L | 38 | Down Arrow | 208 |
| P | 25 | Left Arrow | 203 |
| | | Right Arrow | 205 |

### Mouse

#### `Distance.isMouseDown(button)` → `boolean`
`0` = left, `1` = right, `2` = middle.

#### `Distance.getMouseX()` / `Distance.getMouseY()` → `int`
Mouse position in scaled GUI coordinates.

---

## Entities

All entity query functions return maps (or lists of maps) with the following fields:

| Field | Type | Description |
|---|---|---|
| `name` | String | Display name |
| `id` | int | Entity ID |
| `uuid` | String | Entity UUID |
| `type` | String | Entity type string (e.g. `"Zombie"`, `"Player"`) |
| `x`, `y`, `z` | double | World position |
| `yaw`, `pitch` | float | Rotation |
| `isPlayer` | boolean | True if it's a player |
| `isInvisible` | boolean | |
| `isBurning` | boolean | |
| `isSneaking` | boolean | |
| `isSprinting` | boolean | |
| `health` | float | (EntityLivingBase only) |
| `maxHealth` | float | (EntityLivingBase only) |
| `armor` | int | (EntityLivingBase only) |
| `distance` | float | Distance from local player |

### Query Functions

| Function | Returns | Description |
|---|---|---|
| `getEntities()` | List | All loaded entities |
| `getPlayers()` | List | All players except yourself |
| `getNearbyEntities(radius)` | List | All entities within radius blocks |
| `getNearbyPlayers(radius)` | List | All players within radius blocks |
| `getEntityById(id)` | Map or null | Entity by its entity ID |
| `getClosestPlayer()` | Map or null | Nearest other player |
| `getTargetEntity()` | Entity or null | Entity the crosshair is on |

### `Distance.getEquipment(entityId)` → `Map`
Returns the armor/weapon loadout of an entity.

```javascript
var eq = Distance.getEquipment(target.id);
if (eq != null) {
    Distance.chat("Holding: " + (eq.mainhand ? eq.mainhand.name : "nothing"));
    Distance.chat("Helmet: " + (eq.helmet ? eq.helmet.name : "none"));
}
```

Returns a map with keys: `mainhand`, `boots`, `leggings`, `chestplate`, `helmet`.
Each value is an item map (same format as `getHeldItem()`) or null.

### `Distance.getTargetedBlock()` → `Map or null`
Returns info about the block the crosshair is pointing at.

```javascript
var block = Distance.getTargetedBlock();
if (block != null) {
    Distance.chat("Looking at " + block.id + " face: " + block.side);
}
```

Returns: `{ x, y, z, side }` — side is one of `"up"`, `"down"`, `"north"`, `"south"`, `"east"`, `"west"`.

### `Distance.getTargetedPlaceBlock()` → `Map or null`
Returns the position where a block would be placed (the face-offset block), or null.

---

## Blocks

### `Distance.getBlockAt(x, y, z)` → `Map or null`

```javascript
var block = Distance.getBlockAt(Distance.getPlayerX(), Distance.getPlayerY() - 1, Distance.getPlayerZ());
if (block != null) {
    Distance.chat("Standing on: " + block.name + " (" + block.id + ")");
}
```

Returns: `{ id, name, meta, isAir }`

### `Distance.scanBlocks(x, y, z, radius)` → `List`
Returns all non-air blocks within `radius` blocks of the center point. Each entry has `{ x, y, z, id }`.

> ⚠️ Large radii can be expensive. Keep radius under 8 for real-time use.

```javascript
var blocks = Distance.scanBlocks(Distance.getPlayerX(), Distance.getPlayerY(), Distance.getPlayerZ(), 5);
Distance.chat("Nearby blocks: " + blocks.length);
```

---

## Actions

| Function | Description |
|---|---|
| `attack(entity)` | Attack an entity and swing the arm |
| `swing()` | Swing the player's hand |
| `jump()` | Make the player jump |
| `rightClick()` | Right-click the currently targeted block |
| `sendMessage(text)` | Send a chat message as the player |
| `executeCommand(cmd)` | Execute a command (adds `/` prefix if missing) |
| `notification(title, msg, seconds)` | Show a Distance popup notification |
| `setSprint(state)` | Set sprinting state |
| `setSneak(state)` | Set sneaking state |
| `closeScreen()` | Close the current GUI screen |

```javascript
// Right-click automation
Distance.on("tick", function() {
    if (Distance.isKeyDown("R")) {
        Distance.rightClick();
    }
});
```

---

## Inventory & Items

### Item Object Fields

All item-returning functions share this format:

| Field | Type | Description |
|---|---|---|
| `name` | String | Display name |
| `id` | String | Registry ID (e.g. `"minecraft:diamond_sword"`) |
| `count` | int | Stack size |
| `damage` | int | Current damage value |
| `maxDamage` | int | Max durability |
| `isDamaged` | boolean | True if item has taken damage |
| `hasEnchants` | boolean | True if item is enchanted |

### Reading

| Function | Returns | Description |
|---|---|---|
| `getHeldItem()` | Map or null | Currently held item |
| `getSelectedSlot()` | int | Active hotbar slot index (0–8) |
| `getInventory()` | List | All 36 inventory slots |
| `getInventory("hotbar")` | List | Hotbar slots 0–8 |
| `getInventory("inventory")` | List | Main inventory slots 9–35 |
| `getContainerItems()` | List | All slots in the open container |
| `getChestItems()` | List | Chest-only slots (no player inventory) |
| `getSlot(index)` | Map or null | Single slot from the open container |
| `isInGui()` | boolean | True if a GUI screen is open |
| `getGuiType()` | String | Class name of the current GUI |

### Searching

#### `Distance.findItemInHotbar(query)` → `int`
Returns the first hotbar slot (0–8) whose name or registry ID contains `query` (case-insensitive), or -1.

```javascript
var swordSlot = Distance.findItemInHotbar("sword");
if (swordSlot >= 0) Distance.setSlot(swordSlot);
```

#### `Distance.findItemInInventory(query)` → `int`
Same as above but searches all 36 inventory slots.

### Manipulating

| Function | Description |
|---|---|
| `setSlot(slot)` | Switch to hotbar slot 0–8 |
| `moveItem(from, to)` | Move item between container slots |
| `dropItem(slot)` | Drop item from a slot |
| `clickSlot(slot, button, mode)` | Raw container click (button: 0=left/1=right; mode: 0=normal/4=drop) |

---

## Rendering 2D

All 2D draw calls must happen inside a `render2d` event handler.

**Negative coordinates** are measured from the right/bottom edge of the screen:
```javascript
// Draw 10px from the right edge
Distance.drawText("Hi", -50, 10, 0xFFFFFF);
```

### Screen Info

| Function | Returns | Description |
|---|---|---|
| `getScreenWidth()` | int | GUI screen width |
| `getScreenHeight()` | int | GUI screen height |
| `textWidth(text)` | int | Pixel width of a text string |
| `textHeight()` | int | Font height in pixels |

### Text

| Function | Description |
|---|---|
| `drawText(text, x, y, color)` | Plain text |
| `drawTextShadow(text, x, y, color)` | Text with drop shadow |
| `drawTextCentered(text, x, y, color)` | Horizontally centered around x |
| `drawTextCenteredShadow(text, x, y, color)` | Centered with shadow |
| `drawTextWithBorder(text, x, y, color, borderColor)` | Text with a solid colored outline |

Colors are `0xAARRGGBB`. If alpha is `0x00`, full opacity is assumed.

### Shapes

#### `Distance.drawRect(x, y, w, h, color)`
Filled rectangle (no rounding).

#### `Distance.drawRoundedRect(x, y, w, h, radius, color)`
Filled rounded rectangle.

#### `Distance.drawBorderedRect(x, y, w, h, radius, fillColor, borderColor)`
#### `Distance.drawBorderedRect(x, y, w, h, radius, fillColor, borderColor, borderWidth)`
Rounded rectangle with a visible border.

#### `Distance.drawLine(x1, y1, x2, y2, color, width)`
2D line between two screen points.

#### `Distance.drawGradientRect(x, y, w, h, colorTop, colorBottom)`
Rectangle with a vertical gradient from `colorTop` to `colorBottom`.

#### `Distance.drawCircle(x, y, radius, color)`
Filled circle.

#### `Distance.drawRing(x, y, radius, thickness, color)`
Hollow circle (outline only).

#### `Distance.drawArc(x, y, radius, startDeg, endDeg, color, lineWidth)`
Arc from `startDeg` to `endDeg` (clockwise, in degrees).

```javascript
// Draw a 75% progress arc
Distance.drawArc(100, 100, 20, 0, 270, 0xFF00FF00, 3);
```

### Progress Bars

#### `Distance.drawProgressBar(x, y, w, h, progress, bgColor, fgColor)`
#### `Distance.drawProgressBar(x, y, w, h, progress, radius, bgColor, fgColor)`
Draws a filled progress bar. `progress` is 0.0–1.0.

```javascript
Distance.on("render2d", function() {
    var hp = Distance.getHealth() / Distance.getMaxHealth();
    Distance.drawProgressBar(10, 10, 100, 6, hp, 3, 0x80000000, 0xFFFF5555);
});
```

### Scissor / Clipping

#### `Distance.scissorStart(x, y, w, h)`
Clips all subsequent rendering to the given rectangle.

#### `Distance.scissorEnd()`
Removes the clipping region.

### Images

#### `Distance.drawImage(imageId, x, y, width, height)`
Renders an image previously loaded with `downloadImage`.

#### `Distance.drawResource(path, x, y, width, height)`
Renders a Minecraft texture resource. Use `"namespace:path"` or just `"path"` (defaults to minecraft namespace).

```javascript
Distance.drawResource("textures/items/diamond.png", 10, 10, 16, 16);
```

---

## Rendering 3D

All 3D draw calls must happen inside a `render3d` event handler.

#### `Distance.drawOutline(entity, color, lineWidth)`
Wireframe box around an entity (interpolated for smooth movement).

#### `Distance.drawOutline(x, y, z, color, lineWidth)`
Wireframe 1×1×1 box at a block position.

#### `Distance.drawFilledBox(x, y, z, color)`
Filled 1×1×1 box at a position.

#### `Distance.drawFilledBox(x, y, z, w, h, d, color)` / `Distance.drawBox3D(...)`
Filled box with custom dimensions.

#### `Distance.drawLine3D(x1, y1, z1, x2, y2, z2, color)`
Line between two world-space points.

#### `Distance.drawSphere(x, y, z, radius, slices, stacks, color)`
Filled sphere. `slices`/`stacks` control smoothness (16/16 is typical).

#### `Distance.drawPyramid(x, y, z, width, height, color)`
Filled pyramid shape.

```javascript
Distance.on("render3d", function() {
    var target = Distance.getTargetEntity();
    if (target != null) {
        Distance.drawOutline(target, 0xFFFF0000, 2.0);
    }
});
```

---

## Projection

### `Distance.worldToScreen(x, y, z)` → `Map or null`
Converts 3D world coordinates to 2D screen space. Returns `null` if projection fails.

```javascript
// { x: float, y: float, z: float (depth), visible: boolean }
var pos = Distance.worldToScreen(entity.x, entity.y + 2, entity.z);
if (pos != null && pos.visible) {
    Distance.drawTextShadow(entity.name, pos.x, pos.y, 0xFFFFFF);
}
```

`visible` is `true` when the point is in front of the camera. Check this before rendering to avoid drawing behind you.

### `Distance.getPartialTicks()` → `float`
The render partial ticks value (0.0–1.0) for smooth interpolation between game ticks.

---

## Colors & Utilities

### Building Colors

#### `Distance.rgb(r, g, b)` → `int`
Creates a fully opaque ARGB color from 0–255 components.

#### `Distance.argb(a, r, g, b)` → `int`
Creates an ARGB color with explicit alpha.

```javascript
var red   = Distance.rgb(255, 0, 0);
var semiR = Distance.argb(128, 255, 0, 0); // 50% transparent red
```

#### `Distance.parseColor(hex)` → `int`
Parses a hex string like `"FF0000"` or `"#FF0000"` to ARGB.

```javascript
var color = Distance.parseColor("#3498DB");
```

#### `Distance.resolveColor(colorString)` → `int`
Resolves any Distance color string — plain hex, `"Chroma"`, or `"Multi"` — to the current ARGB value. Useful when passing user-configured colors from settings.

### Interpolation

#### `Distance.lerpColor(colorA, colorB, t)` → `int`
Linearly interpolates between two ARGB colors. `t` is 0.0–1.0.

```javascript
var blend = Distance.lerpColor(0xFFFF0000, 0xFF0000FF, 0.5); // purple
```

### Chroma / Rainbow

#### `Distance.getChromaColor()` → `int`
#### `Distance.getChromaColor(speed)` → `int`
Returns a continuously cycling rainbow color. `speed` defaults to `1.0`; higher values cycle faster.

```javascript
Distance.on("render2d", function() {
    Distance.drawTextShadow("Rainbow!", 10, 10, Distance.getChromaColor());
});
```

---

## Math Helpers

| Function | Description |
|---|---|
| `lerp(a, b, t)` | Linear interpolation (float or double overloads) |
| `clamp(v, min, max)` | Clamp a value to a range (float or double) |
| `distance2D(x1, z1, x2, z2)` | 2D (horizontal) distance |
| `distance3D(x1, y1, z1, x2, y2, z2)` | 3D distance |
| `getDistance(x1, y1, z1, x2, y2, z2)` | Alias for `distance3D` |

```javascript
var smooth = Distance.lerp(prevValue, newValue, 0.1); // smooth approach
var clamped = Distance.clamp(speed, 0, 20);
```

---

## Clipboard

### `Distance.copyToClipboard(text)`
Writes a string to the system clipboard.

```javascript
Distance.copyToClipboard(Distance.getPlayerX() + ", " + Distance.getPlayerY() + ", " + Distance.getPlayerZ());
Distance.chat("&aCoordinates copied!");
```

### `Distance.getClipboard()` → `String`
Reads the current system clipboard contents.

```javascript
var text = Distance.getClipboard();
Distance.chat("Clipboard: " + text);
```

---

## Persistent Storage

Data is saved to `config/DistanceMod/scripts/<scriptName>.dat` and persists across game sessions.

### `Distance.saveData(key, value)`
Saves a string, number, or boolean under `key`.

```javascript
Distance.saveData("highScore", 9001);
Distance.saveData("playerName", "Steve");
Distance.saveData("enabled", true);
```

### `Distance.loadData(key, defaultValue)` → `String`
Loads a string value. Returns `defaultValue` if the key doesn't exist.

### `Distance.loadDataDouble(key, default)` → `double`
### `Distance.loadDataBool(key, default)` → `boolean`
Typed loaders that handle parsing automatically.

```javascript
var highScore = Distance.loadDataDouble("highScore", 0);
var name = Distance.loadData("playerName", "Unknown");
var wasEnabled = Distance.loadDataBool("enabled", false);
```

### `Distance.deleteData(key)`
Removes a saved key.

**Example — persist settings across sessions:**
```javascript
var myVolume = Distance.loadDataDouble("volume", 1.0);

Distance.addSlider("Volume", myVolume, 0.0, 1.0, function(value) {
    myVolume = value;
    Distance.saveData("volume", value);
});
```

---

## Script Communication

Scripts can communicate with each other by broadcasting named events.

### `Distance.emit(event, data)`
Broadcasts a named event (with any data) to **all** loaded scripts, including the sender.

```javascript
// In ScriptA.dscript
Distance.emit("playerDied", { x: Distance.getPlayerX(), z: Distance.getPlayerZ() });
```

```javascript
// In ScriptB.dscript
Distance.on("playerDied", function(data) {
    Distance.chat("Player died at " + data.x + ", " + data.z);
});
```

### `Distance.getScriptNames()` → `List<String>`
Returns a list of the filenames of all currently enabled/loaded scripts.

```javascript
var scripts = Distance.getScriptNames();
Distance.chat("Loaded scripts: " + scripts.join(", "));
```

---

## Web Requests

All requests run on a background thread. Callbacks are automatically dispatched back to the main thread, so it's safe to call any Distance API inside them.

### `Distance.httpGet(url, callback)`
### `Distance.httpGet(url, headers, callback)`

```javascript
Distance.httpGet("https://api.example.com/data", function(response) {
    var data = JSON.parse(response);
    Distance.chat("Result: " + data.value);
});

// With custom headers
Distance.httpGet("https://api.example.com/private", { "Authorization": "Bearer token123" }, function(response) {
    Distance.log(response);
});
```

### `Distance.httpPost(url, data, callback)`
### `Distance.httpPost(url, data, headers, callback)`

```javascript
var payload = JSON.stringify({ score: 9001, name: Distance.getName() });
Distance.httpPost("https://api.example.com/scores", payload, function(response) {
    Distance.chat("&aScore submitted!");
});
```

### `Distance.downloadImage(url, imageId, callback)`
Downloads an image and registers it under `imageId` for use with `drawImage`.

```javascript
Distance.downloadImage("https://example.com/icon.png", "my_icon", function() {
    Distance.log("Image ready");
});

Distance.on("render2d", function() {
    Distance.drawImage("my_icon", 10, 10, 32, 32);
});
```

---

## GUI Registration

Register UI elements in the Distance GUI settings panel. Call these at script load time (not inside event handlers).

### `Distance.createTab(name, icon)`
Creates a custom tab. Must be called before adding any settings.

**Available icon names:** `announcement`, `audio`, `effects`, `folder`, `general_settings`, `heart`, `image`, `irc`, `placeholder`, `premium`, `script`, `search`, `star`, `sun`, `targeter`, `user`, `visuals`, `world`

### `Distance.addToggle(label, defaultValue, callback)`
ON/OFF switch.

### `Distance.addSlider(label, defaultValue, min, max, callback)`
Numeric slider.

### `Distance.addColor(label, defaultColorString, callback)`
Color picker. The callback receives a color string (hex or special like `"Chroma"`).

```javascript
var myColor = 0xFFFFFF;

Distance.addColor("Text Color", "FFFFFF", function(value) {
    myColor = Distance.resolveColor(value);
});
```

### `Distance.addSelector(label, options[], defaultValue, callback)`
Mode selector. `options` is a string array.

```javascript
var mode = "Normal";

Distance.addSelector("Mode", ["Normal", "Aggressive", "Passive"], "Normal", function(value) {
    mode = value;
});
```

### `Distance.addTextInput(label, defaultValue, callback)`
Free-text input field.

```javascript
var serverIP = "play.example.com";

Distance.addTextInput("Server IP", serverIP, function(value) {
    serverIP = value;
});
```

### `Distance.addButton(label, action)`
Clickable button that runs a function.

```javascript
Distance.addButton("Reset Stats", function() {
    hitCount = 0;
    Distance.chat("&eStats reset!");
});
```

---

## Particles & Sound

### `Distance.spawnParticle(type, x, y, z, vx, vy, vz)`

```javascript
Distance.spawnParticle("FLAME", Distance.getPlayerX(), Distance.getPlayerY() + 1, Distance.getPlayerZ(), 0, 0.1, 0);
```

**Common particle types:** `FLAME`, `LAVA`, `SMOKE_NORMAL`, `SMOKE_LARGE`, `EXPLOSION_NORMAL`, `CRIT`, `CRIT_MAGIC`, `SPELL`, `SPELL_MOB`, `PORTAL`, `HEART`, `NOTE`, `REDSTONE`, `WATER_BUBBLE`, `WATER_SPLASH`, `FIREWORKS_SPARK`, `ENCHANTMENT_TABLE`, `VILLAGER_HAPPY`, `VILLAGER_ANGRY`

### `Distance.playSound(sound, volume, pitch)`
- `volume`: 0.0–1.0
- `pitch`: 0.5–2.0

```javascript
Distance.playSound("random.levelup", 1.0, 1.0);
Distance.playSound("random.orb", 0.5, 1.5);
```

**Common sound names:** `random.orb`, `random.levelup`, `random.explode`, `random.successful_hit`, `random.pop`, `note.bass`, `note.pling`, `fire.ignite`, `mob.endermen.portal`
