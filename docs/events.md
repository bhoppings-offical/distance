# Event System

Distance provides an event system that allows your scripts to react to game events.

## Registering Events

Use `Distance.on(eventName, callback)` to register event handlers:

```javascript
Distance.on("tick", function(event) {
    // Your code here
});
```

## Available Events

### `tick`

Fires every game tick (20 times per second).

**Parameters:** `event` (null)

**Use for:**
- Movement logic
- Player state checks
- Timers and cooldowns
- Input handling

```javascript
Distance.on("tick", function(event) {
    if (Distance.isPlayerNull()) return;
    
    // Runs 20 times per second
    Distance.log("Tick!");
});
```

**Example - Speed Boost:**
```javascript
Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    
    if (Distance.isKeyDown(17)) { // W key
        var mx = Distance.getMotionX() * 1.5;
        var mz = Distance.getMotionZ() * 1.5;
        Distance.setMotion(mx, Distance.getMotionY(), mz);
    }
});
```

**Example - Auto Jump:**
```javascript
Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    
    if (Distance.isOnGround() && Distance.isKeyDown(17)) {
        Distance.setMotion(
            Distance.getMotionX(),
            0.42, // Jump velocity
            Distance.getMotionZ()
        );
    }
});
```

---

### `render2d`

Fires when rendering the 2D overlay (HUD).

**Parameters:** `resolution` (screen info)

**Use for:**
- Drawing HUD elements
- Displaying text
- Rendering UI components
- Status indicators

```javascript
Distance.on("render2d", function(resolution) {
    // Draw your HUD here
});
```

**Example - Simple HUD:**
```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    
    var health = Distance.getHealth();
    var x = Distance.getPlayerX().toFixed(1);
    var y = Distance.getPlayerY().toFixed(1);
    var z = Distance.getPlayerZ().toFixed(1);
    
    Distance.drawTextShadow("Health: " + health, 10, 10, 0xFF5555);
    Distance.drawTextShadow("Pos: " + x + ", " + y + ", " + z, 10, 25, 0x55FF55);
});
```

**Example - FPS Counter:**
```javascript
Distance.on("render2d", function() {
    var fps = Distance.getFPS();
    var color = fps >= 60 ? 0x55FF55 : 0xFF5555;
    
    Distance.drawTextShadow("FPS: " + fps, 10, 10, color);
});
```

---

### `render3d`

Fires when rendering the 3D world.

**Parameters:** `partialTicks` (float between 0.0 and 1.0)

**Use for:**
- 3D rendering
- World space effects
- Custom entity rendering

```javascript
Distance.on("render3d", function(partialTicks) {
    // partialTicks: used for smooth rendering between ticks
    // 3D rendering code here
});
```

**Note:** Most scripts use `render2d` for HUD elements. Use `render3d` only when you need to render in world space.

---

### `attack`

Fires when the player attacks an entity.

**Parameters:** `target` (Entity object being attacked)

**Use for:**
- Hit counters
- Combat statistics
- Attack effects
- Sound/particle feedback

```javascript
Distance.on("attack", function(target) {
    var targetName = target.getName();
    Distance.chat("You attacked: " + targetName);
});
```

**Example - Hit Counter:**
```javascript
var hitCount = 0;

Distance.on("attack", function(target) {
    hitCount++;
    Distance.chat("&e⚔ Hit #" + hitCount + " on " + target.getName());
    Distance.playSound("random.successful_hit", 0.5, 1.5);
});
```

**Example - Attack Particles:**
```javascript
Distance.on("attack", function(target) {
    // Spawn particles at target location
    if (target.posX !== undefined) {
        Distance.spawnParticle("CRIT", 
            target.posX, 
            target.posY + 1, 
            target.posZ, 
            0, 0, 0
        );
    }
});
```

---

### `chatReceive`

Fires when a chat message is received.

**Parameters:** `message` (string - unformatted chat text)

**Use for:**
- Chat filters
- Message logging
- Auto-responses
- Keyword detection

```javascript
Distance.on("chatReceive", function(message) {
    Distance.log("Chat: " + message);
    
    if (message.includes("hello")) {
        Distance.sendMessage("Hello!");
    }
});
```

---

### `itemPickup`

Fires when the player picks up an item.

**Parameters:** `item` (object with `name` and `count`)

**Use for:**
- Item tracking
- Auto-sorting
- Pickup notifications

```javascript
Distance.on("itemPickup", function(item) {
    Distance.chat("&aPicked up: " + item.name + " x" + item.count);
    Distance.playSound("random.orb", 1.0, 1.0);
});
```

---

### `itemDrop`

Fires when the player drops an item.

**Parameters:** `item` (object with `name` and `count`)

**Use for:**
- Drop logging
- Item tracking

```javascript
Distance.on("itemDrop", function(item) {
    Distance.chat("&cDropped: " + item.name + " x" + item.count);
});
```

---

### `chestOpen`

Fires when a chest GUI is opened.

**Parameters:** `items` (array of item objects in the chest)

**Use for:**
- Chest organization
- Auto-looting
- Inventory management
- Item detection

```javascript
Distance.on("chestOpen", function(items) {
    Distance.chat("&eChest opened with " + items.length + " items!");
    
    for (var i = 0; i < items.length; i++) {
        var item = items[i];
        Distance.log("Slot " + item.slot + ": " + item.name + " x" + item.count);
        
        if (item.name.includes("Diamond")) {
            Distance.chat("&bFound diamond item in slot " + item.slot);
        }
    }
});
```

**Item object properties:**
- `name` - Display name
- `count` - Stack size
- `slot` - Slot index
- `id` - Item registry ID

---

### `chestClose`

Fires when a chest GUI is closed.

**Parameters:** `null`

```javascript
Distance.on("chestClose", function() {
    Distance.chat("&eChest closed!");
});
```

---

### `inventoryOpen`

Fires when the inventory GUI is opened.

**Parameters:** `null`

```javascript
Distance.on("inventoryOpen", function() {
    Distance.log("Inventory opened");
});
```

---

### `inventoryClose`

Fires when the inventory GUI is closed.

**Parameters:** `null`

```javascript
Distance.on("inventoryClose", function() {
    Distance.log("Inventory closed");
});
```

---

### `key`

Fires when a key is pressed.

**Parameters:** `keyCode` (integer key code)

**Use for:**
- Custom keybindings
- Hotkeys
- Toggle controls

```javascript
Distance.on("key", function(keyCode) {
    Distance.log("Key pressed: " + keyCode);
    
    if (keyCode === 38) {
        Distance.chat("L key pressed!");
    }
});
```


---

### `click`

Fires when a mouse button is clicked.

**Parameters:** `button` (integer: 0=left, 1=right, 2=middle)

**Use for:**
- Custom click actions
- Click counters
- Mouse-based controls

```javascript
var clickCount = 0;

Distance.on("click", function(button) {
    clickCount++;
    Distance.chat("Click #" + clickCount + " (button " + button + ")");
    
    if (button === 0) {
        Distance.chat("Left click!");
    } else if (button === 1) {
        Distance.chat("Right click!");
    }
});
```

---

### `worldChange`

Fires when the player joins a world.

**Parameters:** `null`

**Use for:**
- Resetting state
- Server detection
- World-specific initialization

```javascript
Distance.on("worldChange", function() {
    Distance.chat("&aJoined world!");
    var server = Distance.currentServerIP();
    Distance.chat("Server: " + server);
});
```

---


## Custom Events (Script Communication)

Scripts can fire and receive their own events using `Distance.emit()`:

```javascript
// In any script
Distance.emit("myCustomEvent", { value: 42 });

// In any loaded script (including the sender)
Distance.on("myCustomEvent", function(data) {
    Distance.chat("Got: " + data.value);
});
```

This is the primary way for multiple scripts to communicate. See [Advanced Features](advanced-features.md#script-communication) for a full example.

---

## Multiple Event Handlers

You can register multiple handlers for the same event:

```javascript
Distance.on("tick", function() {
    Distance.log("Handler 1");
});

Distance.on("tick", function() {
    Distance.log("Handler 2");
});

// Both will execute every tick
```

## Event Best Practices

### 1. Always Check for Null

```javascript
Distance.on("tick", function() {
    // GOOD
    if (Distance.isPlayerNull()) return;
    var health = Distance.getHealth();
});

Distance.on("tick", function() {
    // BAD - will crash if player is null
    var health = Distance.getHealth();
});
```

### 2. Avoid Heavy Computation in Render

Compute values in `tick`, display in `render2d`:

```javascript
// GOOD
var cachedSpeed = 0;

Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    var mx = Distance.getMotionX();
    var mz = Distance.getMotionZ();
    cachedSpeed = Math.sqrt(mx * mx + mz * mz);
});

Distance.on("render2d", function() {
    Distance.drawText("Speed: " + cachedSpeed.toFixed(2), 10, 10, 0xFFFFFF);
});
```

### 3. Use Tick Counters for Timing

```javascript
var tickCounter = 0;

Distance.on("tick", function() {
    tickCounter++;
    
    // Do something every 20 ticks (1 second)
    if (tickCounter % 20 === 0) {
        Distance.chat("One second passed!");
    }
    
    // Do something every 3 ticks
    if (tickCounter % 3 === 0) {
        // Spawn particles less frequently
    }
});
```

### 4. Limit Chat Spam

```javascript
var lastChatTime = 0;

Distance.on("tick", function() {
    var now = Date.now();
    
    if (someCondition && now - lastChatTime > 1000) {
        Distance.chat("Message");
        lastChatTime = now;
    }
});
```

### 5. Handle Errors Gracefully

```javascript
Distance.on("tick", function() {
    try {
        if (Distance.isPlayerNull()) return;
        
        // Your code here
        
    } catch (e) {
        Distance.error("Script error: " + e.message);
    }
});
```

## Complete Event Example

```javascript
// Combat Statistics Tracker

var totalHits = 0;
var sessionStart = Date.now();
var lastSpeed = 0;

// Tick event - update calculations
Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    
    var mx = Distance.getMotionX();
    var mz = Distance.getMotionZ();
    lastSpeed = Math.sqrt(mx * mx + mz * mz) * 20;
});

// Render event - display HUD
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    
    var elapsed = (Date.now() - sessionStart) / 1000;
    var hitsPerMinute = (totalHits / elapsed) * 60;
    
    Distance.drawRoundedRect(10, 10, 150, 60, 3, 0x90000000);
    Distance.drawTextShadow("Combat Stats", 15, 15, 0xFFAA00);
    Distance.drawTextShadow("Hits: " + totalHits, 15, 30, 0xFFFFFF);
    Distance.drawTextShadow("APM: " + hitsPerMinute.toFixed(1), 15, 45, 0xFF5555);
    Distance.drawTextShadow("Speed: " + lastSpeed.toFixed(2), 15, 60, 0x55FF55);
});

// Attack event - count hits
Distance.on("attack", function(target) {
    totalHits++;
    Distance.playSound("random.successful_hit", 0.5, 1.5);
});
```

## Next Steps

- Learn about [Rendering](rendering.md) for HUD creation
- Explore [GUI Settings](gui-settings.md) for user controls
- View complete [Examples](examples.md)
