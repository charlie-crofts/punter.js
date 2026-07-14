# Simple 2D Game Engine

A lightweight, dependency-free JavaScript engine for building 2D canvas games (sprites, animation, sound, collision, mobile touch support). No build step — just include the script.

## Requirements

- A modern browser with `fetch` and `Promise` support (polyfill older browsers if needed)
- A `<canvas>` element on the page

## Quick Start

```html
<canvas id="game"></canvas>
<script src="engine.js"></script>
<script>
  engine.init({
    canvas: '#game',
    debug: false, // set true to see FPS, bounds boxes, sprite labels
    sprites: {
      player: 'images/player.png',
      enemy: 'images/enemy.png'
    },
    sounds: {
      jump: 'sounds/jump.mp3'
    }
  });

  engine.on('ready', function () {
    engine.go('level1');
  });

  engine.scene('level1', function () {
    var player = engine.createSprite({
      id: 'player',
      key: 'player',
      x: 50,
      y: 50
    });

    engine.on('update', function () {
      if (engine.keys['ArrowRight']) player.moveX(2);
    });

    engine.on('draw', function () {
      player.draw(this);
    });
  });
</script>
```

## Core Concepts

### 1. `engine.init(config)`
Call this once. Loads all sprite images and sounds, sets up the canvas, and fires `ready` when everything's loaded.

| Option    | Type           | Description                                              |
|-----------|----------------|----------------------------------------------------------|
| `canvas`  | string/element | CSS selector or a `<canvas>` element                     |
| `debug`   | boolean        | Shows FPS counter, sprite bounds, and position labels    |
| `sprites` | object         | `{ key: imageUrl }` map — preloaded before `ready` fires |
| `sounds`  | object         | `{ key: soundUrl }` map — decoded before `ready` fires   |

### 2. Scenes
A scene is just a named function that sets up `update`/`draw` handlers.

```js
engine.scene('menu', function () {
  // set up sprites and handlers here
});

engine.go('menu'); // switches to it and starts the game loop
```

### 3. Events (`engine.on(event, fn)`)
| Event    | When it fires                            |
|----------|------------------------------------------|
| `ready`  | Once, after all assets are loaded        |
| `update` | ~60 times/sec — put game logic here      |
| `draw`   | Every animation frame — put drawing here |
| `resize` | Whenever the canvas resizes              |
| `go`     | Whenever `engine.go()` switches scenes   |

### 4. Sprites (`engine.createSprite(opts)`)
```js
var enemy = engine.createSprite({
  id: 'enemy1',       // must be unique
  key: 'enemy',        // matches a key from config.sprites (or an array for animation)
  x: 100, y: 200,      // position (number or '%' string, e.g. '50%')
  w: 32, h: 32,        // optional size (auto-detected from image if omitted)
  collidable: true,    // computes bounding box for collisions (default: true)
  outline: 'red'       // optional debug outline color
});
```

Useful sprite methods:
- `sprite.draw(ctx)` — draw it (call inside your `draw` handler)
- `sprite.moveX(dx)` / `sprite.moveY(dy)` — move by a delta
- `sprite.center()` / `centerX()` / `centerY()` — center on canvas
- `sprite.animate(delayMs)` — cycle through frames if `key` is an array
- `sprite.bounce(range, speed)` — simple floating/bobbing effect
- `sprite.parallaxScrollX(speed, respawnMs)` — scroll and respawn offscreen (great for backgrounds/obstacles)
- `sprite.loopScrollX(speed)` — scroll and instantly wrap around
- `sprite.isCollidingWith(otherSprite)` — box collision check
- `sprite.destroy()` — remove it from the engine

### 5. Input
```js
engine.keys['ArrowUp']   // true while held down
engine.mouse.x, engine.mouse.y, engine.mouse.clicked  // works for touch too
```

### 6. Sound
```js
engine.playSound('jump', { volume: 0.8, loop: false, speed: 1 });
engine.stopSound('jump');
```

## How the Game Loop Works

`engine.go(sceneName)` auto-starts a fixed-timestep loop (60 updates/sec) once a scene is active:
1. `update` runs 0+ times to catch up to real time (max 100ms per frame to avoid spiral of death)
2. Canvas is cleared
3. `draw` runs once
4. Repeats via `requestAnimationFrame`

Use `engine.pause()` / `engine.resume()` to stop/restart it.

## Responsive Canvas

The engine maintains a **375×667 internal resolution** (portrait phone) and letterboxes/scales to fit any screen, so game logic coordinates stay consistent across devices. Sprites automatically reposition/resize via their `resize()` method — you don't need to do anything manually.

## Debug Mode

Set `debug: true` in `init()` (or `engine.debug = true` later) to see:
- FPS, frame count, canvas size, orientation (bottom-right overlay)
- Red collision-box outlines on every collidable sprite
- Position/size labels under each sprite

## Gotchas for Beginners

- **Sprite IDs must be unique** — reusing one throws an error.
- **Images must finish loading before creating a sprite manually outside `sprites` config** — the engine throws if the image isn't ready.
- **`repeatX` and `repeatY` are mutually exclusive** on a single sprite.
- Percentage strings (`'50%'`) for `x`/`y`/`w`/`h` are resolved relative to canvas size and re-resolve automatically on resize.
- Don't manually resize the canvas — the engine owns that via `resize()`.