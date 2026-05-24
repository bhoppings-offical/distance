# GUI Settings

Create custom configuration panels for your scripts in the Distance GUI.

## Overview

Scripts can add a dedicated tab to the Distance GUI with any combination of:

- **Toggles** — ON/OFF switches
- **Sliders** — Numeric range inputs
- **Color pickers** — Color selectors (hex, Chroma, Multi)
- **Mode selectors** — Dropdown-style option choosers
- **Text inputs** — Free-form string fields
- **Buttons** — Clickable actions

Call all GUI registration functions at **script load time** (top level), not inside event handlers.

---

## Creating a Tab

### `Distance.createTab(name, icon)`

Must be called before adding any settings.

```javascript
Distance.createTab("My Script", "star");
```

**Available icons:** `announcement`, `audio`, `effects`, `folder`, `general_settings`, `heart`, `image`, `irc`, `placeholder`, `premium`, `script`, `search`, `star`, `sun`, `targeter`, `user`, `visuals`, `world`

---

## Toggle

### `Distance.addToggle(label, defaultValue, callback)`

```javascript
var enabled = false;

Distance.addToggle("Enable Feature", false, function(value) {
    enabled = value;
    Distance.chat(value ? "&aON" : "&cOFF");
});
```

---

## Slider

### `Distance.addSlider(label, defaultValue, min, max, callback)`

```javascript
var speedMult = 1.5;

Distance.addSlider("Speed Multiplier", 1.5, 1.0, 3.0, function(value) {
    speedMult = value;
});
```

---

## Color Picker

### `Distance.addColor(label, defaultColorString, callback)`

The callback receives a color string. Pass it through `Distance.resolveColor()` to handle Chroma/Multi values automatically.

```javascript
var myColor = 0xFFFFFFFF;

Distance.addColor("Text Color", "FFFFFF", function(value) {
    myColor = Distance.resolveColor(value);
});
```

You can also use `Distance.parseColor(value)` if you only expect plain hex.

---

## Mode Selector

### `Distance.addSelector(label, options[], defaultValue, callback)`

```javascript
var renderMode = "Box";

Distance.addSelector("ESP Mode", ["Box", "Outline", "Filled"], "Box", function(value) {
    renderMode = value;
});
```

---

## Text Input

### `Distance.addTextInput(label, defaultValue, callback)`

```javascript
var prefix = "[Script]";

Distance.addTextInput("Chat Prefix", "[Script]", function(value) {
    prefix = value;
});
```

---

## Button

### `Distance.addButton(label, action)`

Displays a clickable button that runs a function immediately when pressed.

```javascript
Distance.addButton("Clear Waypoints", function() {
    waypoints = [];
    Distance.notification("Waypoints", "All cleared!", 2);
});
```

---

## Complete Example

```javascript
// ============================================================
// Combat Tracker with full GUI setup
// ============================================================

Distance.createTab("Combat Tracker", "targeter");

// Load persisted settings
var enabled   = Distance.loadDataBool("ct_enabled", false);
var hitColor  = Distance.resolveColor(Distance.loadData("ct_color", "FFAA00"));
var mode      = Distance.loadData("ct_mode", "All");

Distance.addToggle("Enable", enabled, function(value) {
    enabled = value;
    Distance.saveData("ct_enabled", value);
    Distance.chat(value ? "&aTracker ON" : "&cTracker OFF");
    Distance.playSound(value ? "random.levelup" : "random.pop", 1.0, 1.0);
});

Distance.addColor("Hit Color", "FFAA00", function(value) {
    hitColor = Distance.resolveColor(value);
    Distance.saveData("ct_color", value);
});

Distance.addSelector("Track Mode", ["All", "Players Only", "Mobs Only"], mode, function(value) {
    mode = value;
    Distance.saveData("ct_mode", value);
    Distance.chat("Mode: " + value);
});

Distance.addButton("Reset Hit Counter", function() {
    hitCount = 0;
    Distance.notification("Combat Tracker", "Counter reset!", 2);
});

// ---- Logic ----

var hitCount = 0;

Distance.on("attack", function(target) {
    if (!enabled) return;
    if (mode === "Players Only" && target.type !== "Player") return;
    if (mode === "Mobs Only" && target.type === "Player") return;
    hitCount++;
});

Distance.on("render2d", function() {
    if (!enabled || Distance.isPlayerNull()) return;
    Distance.drawRoundedRect(10, 10, 100, 24, 3, 0x90000000);
    Distance.drawTextShadow("Hits: " + hitCount, 15, 17, hitColor);
});
```

---

## Best Practices

**Persist settings with storage** so they survive script reloads:
```javascript
Distance.addSlider("Speed", 1.5, 1.0, 3.0, function(value) {
    speedMult = value;
    Distance.saveData("speed", value);
});
// Load on start:
var speedMult = Distance.loadDataDouble("speed", 1.5);
```

**Set safe defaults.** Features that modify gameplay should default to OFF.

**Give feedback.** Play a sound or send a chat message when a setting changes so users know it registered.

**Label clearly.** "Jump Height Multiplier" is better than "Jump".

**Order settings logically.** Master toggles first, then dependent options.
