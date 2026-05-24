# Advanced Features

Advanced capabilities including storage, script communication, color utilities, world queries, and more.

## Table of Contents

- [Persistent Storage](#persistent-storage)
- [Script Communication](#script-communication)
- [Color Utilities](#color-utilities)
- [Block Scanning](#block-scanning)
- [Entity Queries](#entity-queries)
- [Clipboard](#clipboard)
- [Web Requests](#web-requests)
- [Image Rendering](#image-rendering)
- [3D Rendering](#3d-rendering)
- [Notifications](#notifications)

---

## Persistent Storage

Scripts can save and load data that persists across game sessions. Data is stored in `config/DistanceMod/scripts/<scriptName>.dat`.

```javascript
// Save values
Distance.saveData("score", 9001);        // number
Distance.saveData("name", "Steve");      // string
Distance.saveData("active", true);       // boolean

// Load values
var score  = Distance.loadDataDouble("score", 0);
var name   = Distance.loadData("name", "Unknown");
var active = Distance.loadDataBool("active", false);

// Delete a key
Distance.deleteData("tempKey");
```

**Typical use — persist user settings:**
```javascript
var volume = Distance.loadDataDouble("vol", 1.0);

Distance.addSlider("Volume", volume, 0.0, 1.0, function(value) {
    volume = value;
    Distance.saveData("vol", value);
});
```

---

## Script Communication

Scripts can send and receive events from each other using `emit` and `on`.

### Sending an Event

```javascript
// ScriptA.dscript
Distance.emit("playerAlert", { message: "Enemy spotted!", x: Distance.getPlayerX() });
```

### Receiving an Event

```javascript
// ScriptB.dscript
Distance.on("playerAlert", function(data) {
    Distance.notification("Alert", data.message, 3);
    Distance.chat("Alert at X: " + data.x);
});
```

`emit` broadcasts to **all** loaded scripts simultaneously — including the sender. Use `Distance.getScriptNames()` to inspect what's currently running:

```javascript
var scripts = Distance.getScriptNames();
Distance.chat("Running: " + scripts.join(", "));
```

---

## Color Utilities

### Building Colors

```javascript
var red   = Distance.rgb(255, 0, 0);           // opaque
var semiR = Distance.argb(128, 255, 0, 0);     // 50% alpha
var hex   = Distance.parseColor("#3498DB");    // from hex string
```

### Interpolating Colors

```javascript
// Smooth health bar color: green → red
var hp      = Distance.getHealth() / Distance.getMaxHealth();
var barColor = Distance.lerpColor(0xFFFF0000, 0xFF00FF00, hp);
Distance.drawProgressBar(10, 10, 100, 6, hp, barColor, barColor);
```

### Chroma / Rainbow

```javascript
Distance.on("render2d", function() {
    Distance.drawTextShadow("Chroma Text!", 10, 10, Distance.getChromaColor());
    Distance.drawTextShadow("Fast Chroma!", 10, 25, Distance.getChromaColor(3.0));
});
```

### Resolving User Colors

When using color settings from `addColor`, always use `resolveColor` — it handles plain hex, `"Chroma"`, and `"Multi"` values:

```javascript
var color = 0xFFFFFFFF;

Distance.addColor("Color", "FFFFFF", function(value) {
    color = Distance.resolveColor(value);
});
```

---

## Block Scanning

### Single Block Lookup

```javascript
var block = Distance.getBlockAt(Distance.getPlayerX(), Distance.getPlayerY() - 1, Distance.getPlayerZ());
// { id: "minecraft:stone", name: "Stone", meta: 0, isAir: false }
```

### Area Scan

Scan all non-air blocks within a radius:

```javascript
// Ore finder (radius 8 around player)
Distance.on("key", function(key) {
    if (key !== 38) return; // L key
    
    var blocks = Distance.scanBlocks(
        Distance.getPlayerX(),
        Distance.getPlayerY(),
        Distance.getPlayerZ(),
        8
    );
    
    var ores = blocks.filter(function(b) {
        return b.id.includes("ore");
    });
    
    Distance.chat("Found " + ores.length + " ore blocks nearby");
    
    Distance.on("render3d", function() {
        ores.forEach(function(b) {
            Distance.drawOutline(b.x, b.y, b.z, 0xFFFFAA00, 2.0);
        });
    });
});
```

> Keep radius ≤ 8 for real-time use. Larger scans are fine for one-shot queries on a key press.

---

## Entity Queries

### Nearby Players

```javascript
var nearby = Distance.getNearbyPlayers(10);
Distance.chat("Players within 10 blocks: " + nearby.length);
```

### Closest Target

```javascript
var closest = Distance.getClosestPlayer();
if (closest != null) {
    Distance.chat("Nearest player: " + closest.name + " at " + closest.distance.toFixed(1) + "m");
}
```

### Equipment Check

```javascript
var target = Distance.getClosestPlayer();
if (target != null) {
    var eq = Distance.getEquipment(target.id);
    if (eq != null && eq.helmet != null) {
        Distance.chat(target.name + " is wearing: " + eq.helmet.name);
    }
}
```

### Entity by ID

```javascript
Distance.on("attack", function(target) {
    var entity = Distance.getEntityById(target.id);
    if (entity != null) {
        Distance.chat("Attacked: " + entity.name + " HP: " + entity.health);
    }
});
```

---

## Clipboard

```javascript
// Copy current coordinates
Distance.on("key", function(key) {
    if (key !== 38) return;
    var coords = Distance.getPlayerX().toFixed(0) + " " + Distance.getPlayerY().toFixed(0) + " " + Distance.getPlayerZ().toFixed(0);
    Distance.copyToClipboard(coords);
    Distance.chat("&aCopied: " + coords);
});

// Read clipboard
var text = Distance.getClipboard();
Distance.log("Clipboard: " + text);
```

---

## Web Requests

### GET Request

```javascript
Distance.httpGet("https://api.example.com/status", function(response) {
    var data = JSON.parse(response);
    Distance.chat("Server status: " + data.status);
});
```

### POST Request

```javascript
var payload = JSON.stringify({
    player: Distance.getName(),
    uuid: Distance.getUUID(),
    score: 1000
});

Distance.httpPost("https://api.example.com/scores", payload, function(response) {
    Distance.chat("&aScore submitted!");
});
```

### Custom Headers

```javascript
Distance.httpGet(
    "https://api.example.com/private",
    { "Authorization": "Bearer my-token", "Accept": "application/json" },
    function(response) {
        Distance.log(response);
    }
);
```

All requests are non-blocking. Callbacks run on the main thread — safe to call any Distance API inside them.

---

## Image Rendering

### Downloading an Image

```javascript
Distance.downloadImage(
    "https://crafatar.com/avatars/" + Distance.getUUID(),
    "my_avatar",
    function() {
        Distance.chat("&aAvatar loaded!");
    }
);
```

### Drawing an Image

```javascript
Distance.on("render2d", function() {
    // Draw in top-right corner using negative coords
    Distance.drawImage("my_avatar", -74, 10, 64, 64);
});
```

### Drawing Minecraft Resources

```javascript
Distance.on("render2d", function() {
    Distance.drawResource("textures/items/diamond.png", 10, 10, 16, 16);
    Distance.drawResource("minecraft:textures/entity/creeper/creeper.png", 30, 10, 32, 32);
});
```

---

## 3D Rendering

### Entity ESP

```javascript
Distance.on("render3d", function() {
    var players = Distance.getPlayers();
    for (var i = 0; i < players.length; i++) {
        var p = players[i];
        Distance.drawOutline(p, 0xFFFF0000, 2.0);
        
        var pos = Distance.worldToScreen(p.x, p.y + 2.2, p.z);
        if (pos != null && pos.visible) {
            Distance.drawTextCenteredShadow(p.name, pos.x, pos.y, 0xFFFFFF);
        }
    }
});
```

### Waypoint Markers

```javascript
var waypoints = [];

Distance.on("key", function(key) {
    if (key !== 38) return; // L key
    waypoints.push({ x: Distance.getPlayerX(), y: Distance.getPlayerY(), z: Distance.getPlayerZ() });
    Distance.chat("&aWaypoint saved! Total: " + waypoints.length);
});

Distance.addButton("Clear Waypoints", function() { waypoints = []; });

Distance.on("render3d", function() {
    if (Distance.isPlayerNull()) return;
    var px = Distance.getPlayerX(), py = Distance.getPlayerY(), pz = Distance.getPlayerZ();
    
    for (var i = 0; i < waypoints.length; i++) {
        var wp = waypoints[i];
        Distance.drawFilledBox(wp.x - 0.5, wp.y, wp.z - 0.5, 1, 2, 1, 0x8000FFFF);
        Distance.drawLine3D(px, py + 1, pz, wp.x, wp.y + 1, wp.z, 0x4000FFFF);
    }
});

Distance.on("render2d", function() {
    for (var i = 0; i < waypoints.length; i++) {
        var wp = waypoints[i];
        var pos = Distance.worldToScreen(wp.x, wp.y + 2.5, wp.z);
        if (pos != null && pos.visible) {
            var dist = Distance.distance3D(
                Distance.getPlayerX(), Distance.getPlayerY(), Distance.getPlayerZ(),
                wp.x, wp.y, wp.z
            );
            Distance.drawTextCenteredShadow(dist.toFixed(0) + "m", pos.x, pos.y, 0x00FFFF);
        }
    }
});
```

---

## Notifications

```javascript
Distance.notification("Title", "Message text", 5); // shows for 5 seconds
```

**Example — low health alert:**
```javascript
var lastHealth = 20;
var alerted = false;

Distance.on("tick", function() {
    if (Distance.isPlayerNull()) return;
    var hp = Distance.getHealth();
    
    if (hp < 6 && hp < lastHealth && !alerted) {
        Distance.notification("⚠ Low Health!", "HP: " + hp.toFixed(1), 4);
        Distance.playSound("random.explode", 0.5, 1.5);
        alerted = true;
    }
    if (hp > 8) alerted = false;
    lastHealth = hp;
});
```

---

## Complete Advanced Script

```javascript
// ================================================
// Advanced HUD — combines most features above
// ================================================

Distance.createTab("Advanced HUD", "visuals");

var hudEnabled  = Distance.loadDataBool("hud_on", true);
var espEnabled  = Distance.loadDataBool("esp_on", false);
var hudColor    = Distance.resolveColor(Distance.loadData("hud_color", "FFFFFF"));

Distance.addToggle("Show HUD", hudEnabled, function(v) {
    hudEnabled = v;
    Distance.saveData("hud_on", v);
});

Distance.addToggle("Player ESP", espEnabled, function(v) {
    espEnabled = v;
    Distance.saveData("esp_on", v);
    Distance.chat(v ? "&aESP &aON" : "&cESP &cOFF");
});

Distance.addColor("HUD Color", "FFFFFF", function(v) {
    hudColor = Distance.resolveColor(v);
    Distance.saveData("hud_color", v);
});

Distance.addButton("Copy Coords", function() {
    var coords = Distance.getPlayerX().toFixed(0) + " " + Distance.getPlayerY().toFixed(0) + " " + Distance.getPlayerZ().toFixed(0);
    Distance.copyToClipboard(coords);
    Distance.notification("Coords Copied", coords, 2);
});

// Fetch something from a web API on load
var serverNote = "";
Distance.httpGet("https://api.example.com/motd", function(response) {
    try {
        serverNote = JSON.parse(response).message;
    } catch(e) {}
});

Distance.on("render2d", function() {
    if (!hudEnabled || Distance.isPlayerNull()) return;
    
    var speed = Distance.getSpeed() * 20;
    var ping  = Distance.getServerPing();
    var biome = Distance.getBiome(Distance.getPlayerX(), Distance.getPlayerY(), Distance.getPlayerZ());
    var hp    = Distance.getHealth() / Distance.getMaxHealth();
    
    Distance.drawRoundedRect(10, 10, 160, 75, 4, 0xA0000000);
    
    var hpColor = Distance.lerpColor(0xFFFF3333, 0xFF33FF66, hp);
    Distance.drawProgressBar(14, 14, 152, 5, hp, 2, 0x40000000, hpColor);
    
    Distance.drawTextShadow("Speed: " + speed.toFixed(1) + " b/s", 14, 26, hudColor);
    Distance.drawTextShadow("Ping: " + (ping >= 0 ? ping + "ms" : "SP"), 14, 38, hudColor);
    Distance.drawTextShadow("Biome: " + biome, 14, 50, hudColor);
    Distance.drawTextShadow("XP: " + Distance.getXpLevel() + " (" + (Distance.getXpProgress() * 100).toFixed(0) + "%)", 14, 62, hudColor);
    
    if (serverNote) {
        Distance.drawTextShadow(serverNote, 14, 80, 0xFFFF88);
    }
});

Distance.on("render3d", function() {
    if (!espEnabled) return;
    var players = Distance.getPlayers();
    for (var i = 0; i < players.length; i++) {
        var p = players[i];
        var hpRatio = p.health / p.maxHealth;
        var color = Distance.lerpColor(0xCCFF3333, 0xCC33FF66, hpRatio);
        Distance.drawOutline(p, color, 1.5);
    }
});

Distance.on("worldChange", function() {
    Distance.chat("&aWorld loaded — " + Distance.currentServerIP());
});
```
