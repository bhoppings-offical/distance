# Rendering Guide

Create custom HUDs and visual effects using Distance's 2D and 3D rendering APIs.

## Table of Contents

- [2D Rendering Basics](#2d-rendering-basics)
- [Screen Dimensions & Text Metrics](#screen-dimensions--text-metrics)
- [Text Rendering](#text-rendering)
- [Shapes](#shapes)
- [Lines & Gradients](#lines--gradients)
- [Circles, Rings & Arcs](#circles-rings--arcs)
- [Progress Bars](#progress-bars)
- [Scissor / Clipping](#scissor--clipping)
- [Images & Textures](#images--textures)
- [3D Rendering](#3d-rendering)
- [World to Screen Projection](#world-to-screen-projection)
- [Color Reference](#color-reference)
- [Complete HUD Example](#complete-hud-example)

---

## 2D Rendering Basics

All 2D draw calls must happen inside a `render2d` event handler:

```javascript
Distance.on("render2d", function() {
    // draw here
});
```

### Negative Coordinate Shorthand

Passing a negative value for `x` or `y` positions the element relative to the **right/bottom** edge:

```javascript
// 10px from right edge, 10px from top
Distance.drawTextShadow("Top Right", -80, 10, 0xFFFFFF);

// 10px from bottom-left
Distance.drawText("Bottom", 10, -20, 0xFFFFFF);
```

---

## Screen Dimensions & Text Metrics

```javascript
var w = Distance.getScreenWidth();
var h = Distance.getScreenHeight();

var tw = Distance.textWidth("Hello");   // pixel width of string
var th = Distance.textHeight();         // font height (usually 9)
```

---

## Text Rendering

| Function | Description |
|---|---|
| `drawText(text, x, y, color)` | Plain text |
| `drawTextShadow(text, x, y, color)` | Text with drop shadow — use this for readability |
| `drawTextCentered(text, x, y, color)` | Horizontally centered on x |
| `drawTextCenteredShadow(text, x, y, color)` | Centered with shadow |
| `drawTextWithBorder(text, x, y, color, borderColor)` | Solid outline around text |

Colors are `0xAARRGGBB`. If the alpha byte is `0x00`, full opacity is used.

```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    
    var cx = Distance.getScreenWidth() / 2;
    
    // Centered title
    Distance.drawTextCenteredShadow("&lDISTANCE", cx, 10, 0xFFAA00);
    
    // Right-aligned
    Distance.drawTextShadow("FPS: " + Distance.getFPS(), -50, 10, 0xFFFFFF);
    
    // Outlined
    Distance.drawTextWithBorder("ESP", 10, 30, 0xFF0000, 0x000000);
});
```

---

## Shapes

### `Distance.drawRect(x, y, w, h, color)`
Plain filled rectangle.

### `Distance.drawRoundedRect(x, y, w, h, radius, color)`
Filled rounded rectangle. Use `radius = 0` for sharp corners.

### `Distance.drawBorderedRect(x, y, w, h, radius, fillColor, borderColor)`
### `Distance.drawBorderedRect(x, y, w, h, radius, fillColor, borderColor, borderWidth)`

```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    
    var hp = Distance.getHealth();
    var maxHp = Distance.getMaxHealth();
    
    // Background panel with border
    Distance.drawBorderedRect(10, 10, 120, 50, 4, 0x90000000, 0x40FFFFFF, 1);
    
    // Content inside
    Distance.drawTextShadow("❤ " + hp.toFixed(1) + "/" + maxHp, 16, 16, 0xFF5555);
    Distance.drawTextShadow("Food: " + Distance.getFoodLevel(), 16, 28, 0xFFAA00);
    Distance.drawTextShadow("Armor: " + Distance.getArmor(), 16, 40, 0x5555FF);
});
```

---

## Lines & Gradients

### `Distance.drawLine(x1, y1, x2, y2, color, width)`

```javascript
Distance.on("render2d", function() {
    var cx = Distance.getScreenWidth() / 2;
    var cy = Distance.getScreenHeight() / 2;
    Distance.drawLine(cx - 10, cy, cx + 10, cy, 0xFFFFFFFF, 1);
    Distance.drawLine(cx, cy - 10, cx, cy + 10, 0xFFFFFFFF, 1);
});
```

### `Distance.drawGradientRect(x, y, w, h, colorTop, colorBottom)`

```javascript
// Health bar with gradient
Distance.on("render2d", function() {
    var hp = Distance.getHealth() / Distance.getMaxHealth();
    Distance.drawGradientRect(10, 10, 100 * hp, 8, 0xFFFF5555, 0xFFAA0000);
});
```

---

## Circles, Rings & Arcs

### `Distance.drawCircle(x, y, radius, color)`
Filled circle.

### `Distance.drawRing(x, y, radius, thickness, color)`
Hollow circle (outline only).

### `Distance.drawArc(x, y, radius, startDeg, endDeg, color, lineWidth)`
Partial arc. Degrees are clockwise, 0 = right.

```javascript
// Circular health indicator
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    var hp = Distance.getHealth() / Distance.getMaxHealth();
    var cx = Distance.getScreenWidth() / 2;
    var cy = Distance.getScreenHeight() / 2 + 30;
    
    // Background ring
    Distance.drawRing(cx, cy, 12, 2, 0x40FFFFFF);
    // Health arc
    Distance.drawArc(cx, cy, 12, -90, -90 + (360 * hp), 0xFFFF5555, 2);
    // Center text
    Distance.drawTextCenteredShadow(Math.ceil(Distance.getHealth()) + "", cx, cy - 4, 0xFFFFFF);
});
```

---

## Progress Bars

### `Distance.drawProgressBar(x, y, w, h, progress, bgColor, fgColor)`
### `Distance.drawProgressBar(x, y, w, h, progress, radius, bgColor, fgColor)`

`progress` is clamped to 0.0–1.0.

```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    
    var hp = Distance.getHealth() / Distance.getMaxHealth();
    var hpColor = Distance.lerpColor(0xFFFF3333, 0xFF33FF66, hp);
    
    // Rounded health bar
    Distance.drawProgressBar(10, 10, 120, 7, hp, 3, 0x80000000, hpColor);
    
    // XP bar
    var xp = Distance.getXpProgress();
    Distance.drawProgressBar(10, 22, 120, 5, xp, 0x80000000, 0xFF55FF55);
    
    // Food bar
    var food = Distance.getFoodLevel() / 20;
    Distance.drawProgressBar(10, 32, 120, 5, food, 0x80000000, 0xFFFFAA00);
});
```

---

## Scissor / Clipping

Use scissor to clip rendering to a rectangular region — useful for scroll containers or bounded panels.

```javascript
Distance.scissorStart(10, 10, 200, 100);
// Anything drawn outside (10,10)→(210,110) will be hidden
Distance.drawText("Clipped content", 10, 120, 0xFFFFFF); // invisible — below clip
Distance.drawText("Visible content", 10, 50, 0xFFFFFF);
Distance.scissorEnd();
```

---

## Images & Textures

### Remote Images

```javascript
// Load once at script start
Distance.downloadImage("https://example.com/icon.png", "my_icon", function() {
    Distance.log("Image ready");
});

Distance.on("render2d", function() {
    Distance.drawImage("my_icon", 10, 10, 32, 32);
});
```

### Minecraft Resources

```javascript
Distance.on("render2d", function() {
    Distance.drawResource("textures/items/diamond.png", 10, 10, 16, 16);
    Distance.drawResource("textures/blocks/grass_top.png", 30, 10, 16, 16);
});
```

---

## 3D Rendering

All 3D draw calls go inside `render3d`. They are rendered **without depth testing** (always visible through walls) unless you manage GL state manually.

```javascript
Distance.on("render3d", function(partialTicks) {
    // World-space rendering here
});
```

### Outlines / Wireframes

```javascript
// Around an entity
Distance.drawOutline(entity, 0xFFFF0000, 2.0);

// Around a block
Distance.drawOutline(x, y, z, 0xFF00FF00, 1.5);
```

### Filled Boxes

```javascript
// 1x1x1 block
Distance.drawFilledBox(x, y, z, 0x8000FF00);

// Custom size
Distance.drawFilledBox(x, y, z, 1, 2, 1, 0x8000FF00);
```

### Lines

```javascript
Distance.drawLine3D(
    Distance.getPlayerX(), Distance.getPlayerY() + 1, Distance.getPlayerZ(),
    targetX, targetY, targetZ,
    0xFF00FFFF
);
```

### Spheres & Pyramids

```javascript
Distance.drawSphere(x, y + 1, z, 1.5, 16, 16, 0x8000FFFF);
Distance.drawPyramid(x, y, z, 1, 2, 0x80FF8800);
```

---

## World to Screen Projection

Convert a 3D world position to 2D screen coordinates for labels, health bars, etc.

```javascript
Distance.on("render2d", function() {
    if (Distance.isPlayerNull()) return;
    
    var players = Distance.getPlayers();
    for (var i = 0; i < players.length; i++) {
        var p = players[i];
        var pos = Distance.worldToScreen(p.x, p.y + 2.2, p.z);
        
        if (pos == null || !pos.visible) continue;
        
        var hpRatio = p.health / p.maxHealth;
        var hpColor = Distance.lerpColor(0xFFFF3333, 0xFF33FF66, hpRatio);
        
        // Name tag
        Distance.drawTextCenteredShadow(p.name, pos.x, pos.y, 0xFFFFFF);
        
        // Health bar below name
        var bw = 40;
        Distance.drawProgressBar(pos.x - bw/2, pos.y + 10, bw, 3, hpRatio, 0x80000000, hpColor);
    }
});
```

Always check `pos.visible` before drawing — it prevents rendering labels for entities behind you.

---

## Color Reference

Colors are `int` values in `0xAARRGGBB` format. Alpha `0x00` is treated as fully opaque.

```javascript
0xFFFFFFFF  // white, fully opaque
0xFF000000  // black
0x80FF0000  // red, 50% transparent
0x00FFFFFF  // white, but alpha=0 → drawn fully opaque (Distance behavior)
```

### Building Colors

```javascript
Distance.rgb(255, 0, 0)            // → 0xFFFF0000
Distance.argb(128, 255, 0, 0)      // → 0x80FF0000
Distance.parseColor("#FF5500")     // parse hex string
Distance.lerpColor(a, b, 0.5)      // blend two colors
Distance.getChromaColor()          // cycling rainbow
Distance.resolveColor("Chroma")    // resolve Distance color string
```

---

## Complete HUD Example

```javascript
Distance.createTab("Pro HUD", "visuals");

var show = Distance.loadDataBool("hud_show", true);
var col  = Distance.resolveColor(Distance.loadData("hud_col", "FFFFFF"));

Distance.addToggle("Show HUD", show, function(v) { show = v; Distance.saveData("hud_show", v); });
Distance.addColor("Color", "FFFFFF", function(v) { col = Distance.resolveColor(v); Distance.saveData("hud_col", v); });

Distance.on("render2d", function() {
    if (!show || Distance.isPlayerNull()) return;
    
    var W = Distance.getScreenWidth();
    var hp = Distance.getHealth() / Distance.getMaxHealth();
    
    // Panel
    Distance.drawBorderedRect(W - 144, 10, 134, 80, 5, 0xA0000000, 0x30FFFFFF, 1);
    
    // Health bar
    var hpColor = Distance.lerpColor(0xFFFF4444, 0xFF44FF88, hp);
    Distance.drawProgressBar(W - 140, 14, 126, 6, hp, 3, 0x40000000, hpColor);
    
    // Stats
    var y = 26;
    var lineH = 12;
    Distance.drawTextShadow("❤  " + Distance.getHealth().toFixed(1) + "/" + Distance.getMaxHealth(), W - 140, y, hpColor); y += lineH;
    Distance.drawTextShadow("⚡ " + (Distance.getSpeed() * 20).toFixed(2) + " b/s", W - 140, y, col); y += lineH;
    Distance.drawTextShadow("✦  " + Distance.getXpLevel() + " XP", W - 140, y, 0x55FF55); y += lineH;
    Distance.drawTextShadow("◉  " + Distance.currentServerIP(), W - 140, y, 0xAAAAAA); y += lineH;
    Distance.drawTextShadow("⬡  " + Distance.getBiome(Distance.getPlayerX(), Distance.getPlayerY(), Distance.getPlayerZ()), W - 140, y, col);
    
    // Chroma accent line at top of panel
    Distance.drawLine(W - 144, 10, W - 10, 10, Distance.getChromaColor(0.5), 1);
});
```
