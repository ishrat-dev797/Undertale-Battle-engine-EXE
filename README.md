# Dark Byte Studios — UT Battle Engine (Electron Edition)

**An Undertale-inspired battle engine built with HTML, CSS, JavaScript, and Electron.**

Create custom boss fights, attacks, dialogue, cinematic intros, multi-phase battles, survival sequences, visual effects, and endings by editing `game.js`. The engine runtime lives in `script.js`, so you usually won't need to modify it unless you're changing how the engine itself works.

This edition runs as a desktop application instead of requiring you to open the game in a browser.

## Getting started

### Run from source

**Requirements:** Node.js and npm.

1. Download or clone this repository.
2. Open a terminal in the project folder.
3. Install the dependencies:

   ```bash
   npm install
   ```

4. Launch the engine:

   ```bash
   npm start
   ```

### Build the Windows installer

```bash
npm install
npm run dist
```

The generated build files are placed in `dist/`.

> Building the installer requires the project dependencies and a working development environment. The generated installer is separate from the source code.

### Download a release

If a prebuilt installer is published, download it from the repository's **Releases** page, run the installer, and launch the application.

## Controls

| Key | Action |
|---|---|
| `Z` / `Enter` | Confirm, select, or continue dialogue |
| `X` / `Shift` | Cancel, go back, or slow the soul |
| `Arrow keys` / `WASD` | Move the soul |
| `P` / `Esc` | Pause or resume |
| `M` | Toggle mute |
| `V` | Switch visual effects |
| `F3` | Enable debug controls |
| `F4` | Defeat the current phase while debugging |
| `B` | Pause or resume using the mobile controls |

The mobile controls are primarily relevant to the browser edition. Input availability can depend on the build and platform.

## Project structure

| File or folder | Purpose |
|---|---|
| `main.js` | Electron main process and application window |
| `index.html` | Main application interface |
| `game.js` | Game configuration, dialogue, attacks, phases, and encounter logic |
| `script.js` | Engine runtime and gameplay systems |
| `style.css` | Interface styling |
| `package.json` | Dependencies, scripts, and Electron Builder configuration |
| `assets/` | Soul, bone, blaster, bullet, and other image assets |
| `battle sprites/` | Enemy sprites, including `.png` and animated `.gif` files |
| `audio/` | Music tracks |
| `sfx/` | Sound effects and voice clips |
| `fonts/` | Fonts, including `DeterminationMono.ttf` |
| `build/` | Build resources, such as application icons, when included |
| `pkg/` | Separate packaged web-edition resources |
| `dist/` | Generated installer and build output |

Some folders may differ between source distributions and packaged releases. Keep the existing asset paths intact when modifying the project.

`assets/gaster blaster animation sprites/` contains the original artwork. The engine uses the cleaned copies in `assets/blaster/`, which have transparent backgrounds.

---

## Custom sprites, music, and sounds

You can register your own images, music tracks, sound effects, and enemy sprites in `game.js`.

```js
music: "audio/my theme.mp3",

enemy: {
  sprite: "battle sprites/boss.gif",
  voice: "myVoice"
},

images: {
  star: "assets/star.png",
  bone: {
    src: "assets/mybone.png",
    crop: [0.3, 0.03, 0.4, 0.94]
  }
},

sounds: {
  myVoice: "sfx/voice.mp3",
  ding: "sfx/ding.wav"
},
```

### Image cropping

`crop` uses the format `[x, y, w, h]`, with values expressed as fractions from `0` to `1`. It lets you crop away transparent or empty space around an image.

### Replacing default assets

Naming a registered image or sound `soul`, `bone`, `hurt`, `heal`, `hit`, or `dialogue` can replace the corresponding default asset.

Use registered sounds in an attack:

```js
a.sound("ding");
```

Or through the game API:

```js
g.sound("ding");
```

Use custom images as bullets:

```js
a.sprite({
  img: "star",
  x: 320,
  y: 200,
  w: 32,
  h: 32
});
```

Change the music during a battle:

```js
g.music("audio/phase2.mp3");
g.stopMusic();
```

Keep in mind that asset paths must match the actual files and the application's expected directory structure.

---

## Creating attacks

Attacks are defined inside the `attacks` object in `game.js`.

```js
attacks: {
  myAttack: {
    box: [220, 140], // Box size: [width, height]
    mode: "red",     // "red" for free movement; "blue" for gravity

    duration: 400,   // Frames; 60 frames is approximately 1 second

    start(a) {
      // Runs once when the attack begins
    },

    update(a) {
      // Runs every frame
    },

    end(a) {
      // Runs once when the attack ends
    },

    draw(ctx, a) {
      // Optional: draw custom graphics on the canvas
    }
  }
}
```

To use the attack, add an entry like this to `turns`:

```js
{
  say: "* Get ready.",
  attack: "myAttack"
}
```

You can also provide an attack object directly through `attack:` instead of referencing a registered attack by name.

### Attack helper API (`a`)

The `a` object provides helpers for creating bullets, moving the battle box, changing soul modes, and controlling the attack's state.

| Helper | Description |
|---|---|
| `a.frame` | Current attack frame |
| `a.every(n, offset)` | Checks whether the current frame matches a repeating interval |
| `a.box` | Battle-box bounds: `{left, top, right, bottom, w, h}` |
| `a.soul` | Soul position: `{x, y}` |
| `a.vars` | Variables associated with the attack |
| `a.bone({x,y,w,h,vx,vy,ax,ay,color,damage,horizontal})` | Creates a bone projectile |
| `a.orb({x,y,vx,vy})` | Creates an orb |
| `a.aim(x,y,speed)` | Fires an orb toward the soul |
| `a.ring({n,speed,x,y,angle,radius})` | Creates a circular pattern of orbs |
| `a.fan({n,arc,x,y,speed,angle})` | Creates a fan of orbs, aimed at the soul by default |
| `a.wall(side,gap,gapSize,speed,{color})` | Creates a sweeping wall of bones with a gap |
| `a.sprite({img,x,y,w,h,vx,vy,rot,spin,color,damage})` | Creates a projectile using a custom image |
| `a.blaster({to,from,angle,size,hold,charge,beamW,color,damage})` | Creates a Gaster Blaster attack |
| `a.setBox(w,h,cx,cy,speed)` | Resizes the battle box and can optionally move it |
| `a.setMode("blue","left")` | Changes the soul's mode and gravity direction |
| `a.slam("right",speed)` | Turns the soul blue and launches it toward a wall |
| `a.warn({x,y,w,h,frames,color,then})` | Displays a harmless warning zone and runs `then()` when it finishes |
| `a.clear()` | Removes bullets; blasters remain unless `a.clear(true)` is used |
| `a.angleTo(x,y)` | Calculates the angle from a point toward the soul |
| `a.state` | Attack-specific scratch object |

#### Projectile colors

Bones support these colors:

- `white` — normal bone behavior.
- `blue` — the soul must stand still to avoid damage.
- `orange` — the soul must keep moving to avoid damage.

Blue soul mode uses gravity. The default gravity direction is downward, but attacks can change it to `left`, `right`, or `up`.

#### Moving the battle box

You can resize the box and optionally move it to a new center:

```js
a.setBox(240, 150, 320, 300, 0.22);
```

Here, `cx` and `cy` are the desired center coordinates, and `speed` controls the movement speed. The default slide speed is `0.22`.

#### Gaster Blasters

The `angle` is in degrees:

- `0` = right
- `90` = down

If no explicit direction is provided, the blaster can aim at the soul by default.

#### Custom projectile updates

Individual bullets can have their own `update(b)` function and `life` value.

```js
a.bone({
  x: 100,
  y: 100,
  w: 20,
  h: 50,
  vx: 0,
  vy: 1,
  life: 180,

  update(b) {
    b.vy += 0.1;
  }
});
```

Use this to create acceleration, curved movement, timed projectiles, and other custom patterns.

---

## Game script API (`g`)

The `g` object exposes game state and functions. It is available in hooks, acts, items, turns, and attack helpers where supported.

### State values

| Property | Description |
|---|---|
| `g.vars` | Custom game variables |
| `g.turn` | Current turn |
| `g.hp` | Player HP |
| `g.maxHp` | Maximum player HP |
| `g.enemyHp` | Enemy HP |
| `g.spareable` | Whether the enemy can be spared |
| `g.phase` | Current phase |
| `g.phaseCount` | Number of phases |

### Common functions

| Function | Description |
|---|---|
| `g.heal(n)` | Heals the player |
| `g.damage(n)` | Damages the player |
| `g.hurtEnemy(n)` | Damages the enemy |
| `g.setSpareable(bool)` | Changes whether the enemy can be spared |
| `g.setFlavor(text)` | Changes the enemy's flavor text |
| `g.setEnemySprite(imageKey)` | Changes the enemy sprite |
| `g.flash(color, frames)` | Triggers a screen flash |
| `g.shake(amt, frames)` | Shakes the screen |
| `g.particles(x,y,n,{color,speed,life,size,gravity})` | Spawns particles |
| `g.vfx("off"|"low"|"high")` | Changes the VFX level |
| `g.sound(name)` | Plays a registered sound |
| `g.music(path)` | Starts music |
| `g.stopMusic()` | Stops music |
| `g.after(frames, fn)` | Schedules a function after a number of frames |
| `g.rand(a,b)` | Generates a random number |
| `g.randInt(a,b)` | Generates a random integer |
| `g.pick(list)` | Picks an entry from a list |
| `g.vfxColor(color, mote)` | Changes the VFX color configuration |
| `g.glitch(frames, power)` | Triggers a glitch effect |
| `g.aberrate(px, frames)` | Applies chromatic aberration |
| `g.cam(options, frames)` | Controls the camera |
| `g.cam("reset", frames)` | Resets the camera |
| `g.camKick(amount)` | Triggers a quick camera zoom punch |
| `g.tilt(deg, frames)` | Tilts the camera |
| `g.bars(bool, frames)` | Shows or hides cinematic bars |
| `g.freeze(frames)` | Temporarily freezes the action |
| `g.pulse(amount)` | Controls the background pulse |
| `g.ring(x,y,options)` | Creates a visual shockwave ring |
| `g.float(x,y,text,options)` | Displays floating text |
| `g.popup(options)` | Displays a pop-up image effect |
| `g.pageTitle(text)` | Changes the browser tab title in the web edition |

Some functions are intended for specific effects or phases. Check their existing usage in `game.js` before relying on them in a new encounter.

### Game hooks

All hooks are optional and are defined in the `Game` configuration:

```js
onStart(g)
onTurnStart(g, n)
onTurnEnd(g, n)
onHurt(g, dmg)
onAct(g, act)
onItem(g, item)
onWin(g, kind)
onGameOver(g)
onPhaseStart(g, n)
```

`onHurt(g, dmg)` can return a number to modify the damage being applied.

### Custom acts and items

Acts and items can run custom code:

```js
{
  name: "Pet",

  run(g) {
    g.heal(5);
    return "* You petted it.";
  }
}
```

Text can also be generated dynamically:

```js
text(g) {
  return "* HP: " + g.hp;
}
```

### Dynamic enemy turns

You can define `turns` as a function to select dialogue and attacks based on game state:

```js
turns: (n, g) => ({
  say: "* The battle continues...",
  attack: g.hp < 30 ? "blasters" : "boneWave"
})
```

This allows attacks and dialogue to change as the player's HP, turn number, or other variables change.

---

## Cinematic intro

Use `cinematic` to create an optional intro before the battle.

```js
cinematic: {
  enabled: false
}
```

Set `enabled: true` to activate it. Players can hold `X` to skip the intro where supported.

Scenes play in order, and different scene keys can be combined within a scene.

| Key | Description |
|---|---|
| `wait: 60` | Waits for a number of frames |
| `title: "NAME"` | Displays a large fading title |
| `sub: "small"` | Adds a subtitle |
| `frames` | Controls the duration of supported cinematic elements |
| `text: "line"` | Displays typed dialogue |
| `text: ["page 1", "page 2"]` | Displays multiple text pages |
| `auto: 90` | Automatically advances text after a delay |
| `size`, `color`, `y` | Adjust supported text properties |
| `image: "key"` | Displays a registered image |
| `fadeIn`, `fadeOut` | Controls image fade timing |
| `zoom`, `scale`, `x`, `y` | Controls image presentation |
| `fade: "out"` / `fade: "in"` | Fades to or from black |
| `bg: "#001"` | Changes the background color |
| `flash: true` | Triggers a flash |
| `shake: [6, 20]` | Triggers screen shake |
| `sfx: "hit"` | Plays a sound effect |
| `music: "audio/intro.mp3"` | Starts intro music |
| `music: null` | Stops intro music |
| `fn(c) {}` | Runs custom cinematic code |

Images used in a cinematic must be registered in `images`.

The intro music stops when the cinematic ends, unless your configuration or custom code changes that behavior.

---

## Visual effects (VFX)

Configure visual effects in `game.js`:

```js
vfx: {
  level: "high",
  color: "#4b1a86"
}
```

Supported levels:

- `off` — disables the optional visual effects.
- `low` — includes effects such as the soft bottom gradient, sparks, and hit flashes.
- `high` — adds floating motes, glow, soul trails, and vignette effects.

Players can press `V` to switch VFX levels.

Some effects, including glitch effects, depend on VFX being enabled. Consider accessibility when designing frequent flashes, screen shake, and glitch sequences.

---

## Multi-phase battles and cutscenes

When enemy HP reaches zero, the next entry in `phases` begins. A phase can have its own enemy configuration, attacks, dialogue, music, and optional cutscene.

```js
phases: [
  {
    hp: 100,
    heal: 10,
    enemyFirst: true,
    intro: "* The air feels heavy.",

    music: "audio/phase2.mp3",

    enemy: {
      sprite: "battle sprites/p2.gif",
      flavor: [],
      check: [],
      acts: []
    },

    attacks: {
      // Phase-specific attacks
    },

    turns: [
      // Phase-specific turns
    ],

    cutscene: [
      { music: null, hit: true, flash: true, wait: 90 },
      { say: "* The battle changes..." }
    ]
  }
]
```

The phase's enemy configuration can override the inherited enemy properties. Properties not overridden can remain inherited from the previous configuration.

`turns` can also be a function:

```js
turns: (n, g) => ({
  say: "* Choose carefully.",
  attack: g.hp < 30 ? "blasters" : "boneWave"
})
```

### Cutscene step keys

Cutscene steps can combine supported keys:

| Key | Description |
|---|---|
| `wait` / `frames` | Delays the next step |
| `say` | Displays a dialogue bubble |
| `text` | Displays narration |
| `auto` | Controls automatic text progression |
| `music` | Starts music, stops it with `null`, or uses `"next"` where supported |
| `loop` | Controls supported looping behavior |
| `sfx` | Plays a sound effect |
| `shake: [amt, frames]` | Shakes the screen |
| `flash` | Triggers a flash |
| `hit` | Triggers a hit effect |
| `dark: 0..1` | Controls the dark overlay |
| `fadeEnemy: 0..1` | Controls enemy fading |
| `sprite: "next"` | Advances to the next enemy sprite where supported |
| `particles: [x,y,n,{...}]` | Spawns particles |
| `ui: true` | Controls supported interface visibility |
| `fn(g)` | Runs custom game code |

Hold `X` to skip skippable cutscenes.

### Phase checkpoints

Dying inside a phase restarts that phase by default. Set:

```js
checkpoints: false
```

to restart the entire encounter instead.

### Additional phase and attack options

- `gravity: "left"` — starts a blue-soul attack with gravity directed toward the left wall.
- `at: [cx, cy]` — positions the battle box at a custom center instead of its default bottom-center location.
- `onPhaseStart(g, n)` — runs when a phase starts.
- `g.phase` and `g.phaseCount` — expose phase information.
- `g.vfxColor(color, mote)` — changes the VFX color configuration.

---

## Enemy dodging

The `dodges` setting makes an enemy evade a set number of player attacks.

```js
enemy: {
  dodges: 10
}
```

The enemy sidesteps the first ten attacks, displaying `MISS`. The eleventh attack can land and defeat the phase, assuming no other mechanics interfere.

A later phase can disable this behavior:

```js
enemy: {
  dodges: 0
}
```

Missing the attack bar does not count as an enemy dodge.

---

## Survival phases

A survival phase replaces the normal turn-based menu with one continuous attack.

```js
{
  survival: "myAttack",

  heal: 30,
  skippable: false,

  music: "audio/fast.mp3",

  enemy: {
    sprite: "battle sprites/p25.png"
  },

  cutscene: [
    {
      music: null,
      hit: true,
      wait: 90
    }
  ],

  attacks: {
    myAttack: {
      box: [240, 150],
      at: [320, 312],
      duration: 2460,

      update(a) {
        // Survival attack logic
      }
    }
  }
}
```

`skippable: false` prevents holding `X` from skipping the phase's cutscene.

When the attack duration ends, the ending sequence can play: a white flash, the enemy crumbling to dust, music fading, and a final message.

Configure the ending in `game.js`:

```js
ending: {
  text: "Thanks for playing!",
  musicFade: 330,
  restart: false
}
```

- `musicFade` controls the music fade duration in frames.
- `restart: true` allows players to press `Z` to play again.

Dying during a survival phase restarts that survival phase. During debugging, press `F3` and then `F4` to skip directly to the ending.

### Phase 2.5 segments

The Phase 2.5 survival fight is built from the `P25` segment list in `game.js`.

Each segment defines a battle-box size and center, a duration, and a pattern function:

```js
[w, h, centerX, centerY]
```

The pattern function receives the attack helper and segment-related parameters, commonly in the form:

```js
run(a, n, s)
```

You can edit, reorder, or add segments to customize the fight.

Built-in fairness measures include:

- A dashed outline previews the next battle-box position.
- Bullets do not spawn while the box is sliding.
- Bullets are cleared when the box jumps.
- Walls avoid spawning directly next to the soul.
- Each segment ends with a quiet tail.

---

## Sound configuration

Register sounds with a simple path:

```js
sounds: {
  ding: "sfx/ding.wav"
}
```

Or use an object to configure playback:

```js
sounds: {
  ding: {
    src: "sfx/ding.wav",
    volume: 0.8,
    maxDur: 3,
    poly: 2,
    gap: 100
  }
}
```

| Option | Description |
|---|---|
| `src` | Sound file path |
| `volume` | Playback volume |
| `maxDur` | Maximum clip duration in seconds |
| `poly` | Maximum number of overlapping copies |
| `gap` | Merges identical sounds started within this many milliseconds |

The `gap` option helps prevent several identical sounds from becoming excessively loud when many projectiles fire simultaneously.

Default sound names include:

- `hurt` — the soul takes damage.
- `hit` — the player's attack lands.
- `dialogue` — dialogue sound.
- `voice_sans` — Sans voice sound.
- `blasterCharge` — blaster charging.
- `blasterFire` — blaster firing.

Built-in beeps are available for `dodge`, `swoosh`, `slam`, and `break` when custom sound files are not supplied.

Sounds pause with `P`, stop on Game Over, and obey the mute toggle.

---

## Additional battle mechanics

### Orange soul

Use:

```js
mode: "orange"
```

or:

```js
a.setMode("orange");
```

The orange soul must keep moving. Standing still for `Game.orange.grace` frames causes damage, followed by recurring damage at approximately one-second intervals while the condition persists.

A closing ring warns the player, and `KEEP MOVING!` appears when the mode activates.

Phase segments can also use `mode: "orange"`. The windmill and spinning-arm patterns use this mechanic.

### Slash effects

The engine supports slash effects through:

```js
a.slash();
g.slash();
```

A cutscene can also use:

```js
{ slash: true }
```

The slash plays `enemySlash` (`sfx/enemy slash.mp3`) and displays a slash animation over Sans at 60% opacity. The animation drifts while fading away, alternating directions between consecutive slashes. Each phase starts with a rightward slash.

Customize an individual slash:

```js
a.slash({
  dir: 1,
  frames: 30,
  dist: 100,
  opacity: 0.6
});
```

`dir` can be `1` or `-1`.

Defaults can be adjusted using `slash: { ... }` in `game.js`. See the `Slash` implementation in `script.js`.

The cleaned animation is:

`battle sprites/last-breath phase 2 slash animation clean.gif`

The original GIF remains in the project folder.

### Songs that follow one another

The engine exposes music playback helpers:

```js
Engine.playMusic(src, {
  loop: false,

  onEnd() {
    // Runs when the song finishes
  }
});
```

`Engine.musicTime()` returns the current song time in seconds, or `-1` when music is not playing.

If browser autoplay restrictions block music in the web edition, playback can be retried after the next key press or click.

`a.finish()` ends the current attack.

The Phase 3 `rebirth` intro can play three consecutive songs using `musicTalk`, `musicAfter`, and `musicBreak`. The buttons break at `breakAt` seconds into the third song. The fight itself can play songs D and E and follow their timing.

### Custom attack helpers

The project includes additional attack helpers and patterns in `game.js`:

- `cutLine` / `cutThrough` — slash-cut patterns.
- `spinArm` — spinning bone arm.
- `stabCols` — bone stabs.

Newer Phase 2 patterns include:

- `p2_slash`
- `p2_crossfan`
- `p2_stab`
- `p2_windmill`

Other helpers include:

- `a.at(frame, fn)` — runs a function at a specified attack frame.
- `a.vring(x, y, {...})` — creates a visual shockwave.
- `a.ring(...)` — creates a circle of orbs, not the visual shockwave helper.
- `a.pattern("name", {...})` — runs a registered custom pattern.

Define reusable patterns under:

```js
patterns: {
  name(a, options) {
    // Pattern logic
  }
}
```

Use them through `a.pattern("name", {...})`.

---

## BLOCKED, glitch endings, and rebirth

### BLOCKED

Configure an enemy with:

```js
enemy: {
  blocks: [15, 20]
}
```

Player attacks can land but display `BLOCKED` without dealing damage. The phase ends after the configured number of blocks.

The array specifies a random threshold between the two values. A single number can be used for a fixed threshold.

`g.vars.blocked` tracks how many blocks have occurred and can be used in dialogue or phase logic.

### Glitch ending

A phase ending can use:

```js
ending: {
  text: "Thanks for playing!",
  musicFade: 330,
  glitchAt: 60,
  glitchFor: 30,
  blackFor: 60
}
```

When another phase follows and `glitchAt` is configured, the ending scene can glitch at the specified frame, transition to black, and start the next phase immediately.

### Rebirth

The `rebirth` phase option controls a cinematic transition into a later battle:

```js
rebirth: {
  hold: 60,
  fade: 30,
  musicWait: 60,
  text: "* Begin again.",
  auto: 90,
  bpm: 120,
  beatOffset: 0,
  every: 2,
  calm: 60,
  restartMusic: true
}
```

The transition can fade the box, HUD, and buttons; play music and centered dialogue; and break buttons in rhythm. After the configured calm period, a flash can restore HP and start the survival attack.

Dying in this phase restarts the fight directly instead of replaying the long intro.

### Phase 3 pattern configuration

Phase 3 uses `P3_PATTERNS` and `P3_ORDER` in `game.js`.

- `P3_PATTERNS` contains eight beat-synchronized patterns.
- `P3_ORDER` determines the pattern sequence.
- The default arrangement contains 24 segments of 600 frames each.
- `P3_BPM` controls the tempo.
- `P3_OFFSET` adjusts beat alignment.
- `DMG3` controls Phase 3 damage.

Tune these values if attacks feel out of sync with the music.

---

## Camera, cinematics, and advanced VFX

Version 0.6.0 introduced an expanded camera and effects system.

### Camera API

```js
g.cam({
  zoom: 1.1,
  x: 0,
  y: 0,
  rot: 0,
  focus: [320, 240]
}, 60);

g.cam("reset", 60);
g.camKick(0.03);
g.tilt(5, 30);

g.bars(true, 30);
g.freeze(5);
g.pulse(0.5);

g.ring(320, 240, {
  r1: 20,
  life: 30,
  w: 3,
  color: "#ffffff"
});

g.float(320, 200, "BLOCKED", {
  color: "#ffffff"
});
```

- `g.cam(...)` controls zoom, position, rotation, and focus.
- `g.cam("reset", frames)` restores the camera.
- `g.camKick(amount)` creates a quick zoom punch.
- `g.tilt(deg, frames)` tilts the camera.
- `g.bars(bool, frames)` controls cinematic bars.
- `g.freeze(frames)` creates hit-stop.
- `g.pulse(0..1)` controls the background pulse.
- `g.ring(...)` creates a visual ring.
- `g.float(...)` displays floating text.

### Cinematic step keys

Additional supported cutscene keys include:

- `cam: { zoom, x, y, rot, focus, frames }`
- `cam: "reset"`
- `bars`
- `barsFrames`
- `kick`
- `rkick`
- `aberrate: [px, frames]`
- `freeze`
- `pulse`
- `ring: [x, y, {...}]`
- `soul: [x, y]` or `soul: false`
- `uiIn: frames`
- `title`
- `sub`

The exact effect depends on the values supplied and the current phase or cutscene implementation.

### Phase opening

Use `opening` to create a cinematic battle opening before the first turn:

```js
opening: {
  skippable: true,
  steps: [
    // Opening steps
  ]
}
```

The opening plays in the battle scene and does not repeat after a Game Over.

`enemyFirst: true` makes Sans attack before the menu appears. `musicAfterFirstAttack: true` delays the music until the first attack ends. The opening attack in the current game is `p1_opening`, which uses camera movement, a stretched battle box, bones, blaster patterns, and a squeeze effect. Its damage is configured by `P1_DMG`.

The older title-card cinematic can be disabled with:

```js
cinematic: {
  enabled: false
}
```

### Camera tilt

Phases can configure automatic camera tilting:

```js
tilt: {
  max: 5,
  chance: 0.2,
  warn: 15,
  hold: 30,
  every: 120,
  first: 60
}
```

The camera can warn the player with a creak and shudder before tilting and returning to its original position. An attack can opt out with:

```js
noTilt: true
```

### Interrupt slashes

Phase 2 supports interrupt slashes through `enemy.interrupt`.

```js
enemy: {
  interrupt: {
    remaining: 7,
    after: 180,
    chance: 0.5,
    perTurn: 1,
    warn: 20,
    damage: 1,
    say: "* ..."
  }
}
```

The interrupt system can make Sans slash during the player's turn while the player is navigating menus, lists, or the attack bar.

The system warns the player before the slash lands. Starting Sans's next attack cancels pending slashes. The chip damage is designed not to kill the player.

The `INT_TABLE` configuration in `script.js` controls the timing and intensity of the interrupt slashes. Its rows use:

`[frames between slashes, randomness, warning frames, damage multiplier, cuts]`

The table makes the slashes faster and more unpredictable as fewer blocks remain. At very low block counts, double or triple cuts can occur with shorter warnings.

### Phase 2 repel

Blocked hits can zoom the screen in, trigger glitches, and emit shockwaves.

### Phase 3 attacks

Phase 3 replaces its former slash attacks with orb fans such as `crossfan` and falling bones such as `hail`.

### Expanded VFX

The effects system includes:

- Stars and light shafts.
- Enemy aura.
- Shockwave rings.
- Floating damage numbers.
- Bullet trails.
- Blaster charging glow and sparks.
- Battle-box ripples.
- HP lag bars.
- Low-HP pulses.
- Hit-stop.
- Cinematic Game Over effects.

`VFX.spawn` supports shape options such as `square`, `circle`, and `line`, along with `shrink` and `add`.

---

## Menus and customization

The engine includes a title screen, pause menu, options, and battle interface customization.

### Title and pause menus

Configure the title screen and pause menu in `game.js` using `menu` and `pause`.

These configurations can support:

- `items` — custom menu entries such as `{ label, run(m), page, at }`.
- `pages` — custom screens.
- `hide` — hides supported menu elements.
- `title` — changes the title.
- `startLabel` — changes the start label.
- `resumeLabel` — changes the resume label.
- `controls` — replaces the controls-page text.

The title screen can be restyled using:

```js
menu: {
  color: "#4b1a86",
  mote: "#a855f7",
  sub: "A custom subtitle"
}
```

### Options

The browser edition includes options for:

- Visual effects.
- Music volume.
- Sound effects.
- Screen shake.
- Camera behavior (`FULL`, `NO TILT`, or `OFF`).
- Flash and glitch effects.

The browser edition saves these settings in the browser. Storage behavior in the Electron edition depends on the application's configuration.

### Mobile controls

The browser edition includes a phone-button toggle for mobile controls.

- Four transparent directional buttons appear on the left.
- `X` and `Z` appear on the right.
- The controls support multiple simultaneous touches.
- Sliding a finger from one arrow to another releases the old direction and presses the new one.
- The `B` button pauses or resumes by triggering `P`.
- In blue soul mode, `Z` also jumps.

These controls are primarily for the web edition and touch-enabled environments.

---

## Phase 3 memes and animation

### Meme pop-ups

`g.popup(...)` creates an image pop-up behind the battle box and bullets so it does not obscure the attack.

```js
g.popup({
  img: "meme1",
  x: 320,
  y: 200,
  h: 120,
  frames: 60,
  from: "top",
  rot: 0
});
```

The helper accepts additional options, including positioning, rotation, and timing.

The three registered images are `meme1`, `meme2`, and `meme3`.

Phase 2 includes the attacks `p2_meme1`, `p2_meme2`, and `p2_meme3`. They start with a calmer pattern before the meme appears and Sans reacts with shaking, dialogue, and telegraphed attacks.

### Phase 3 events

`P3_EVENTS` in `game.js` configures timed events in the Phase 3 sequence.

An event can include:

```js
{
  t: 10,
  meme: [
    {
      img: "meme1",
      spot: "left",
      h: 120
    }
  ],
  freak: {
    kind: "glitch",
    frames: 30
  },
  say: ["* What is happening?"],
  cap: "A caption",
  fn: () => {}
}
```

The exact available `spot` values and `freak.kind` values depend on the implementation in `game.js`.

`MEME_SPOT` controls the placement of meme images.

### Phase 3: the animation

Phase 3 is designed as a film that Sans plays and repeatedly disrupts. The sequence is synchronized to the song, with a timeline along the bottom and subtitles in the top-left corner.

The current sequence includes:

- Loading bar.
- Missing-texture effect, where bones become purple and black squares.
- Mouse cursor clicking the arena.
- Ctrl+C and Ctrl+V sequences.
- Buffering spinner.
- Draggable “Not Responding” window.
- Stop-motion.
- Video player that pauses, rewinds, and fast-forwards bullets.
- Quiet section with a meme.
- Spam mode.
- Gravity sequence.
- Error pop-ups with memes.
- Bouncing ball with onion-skin ghosts.
- Spinning timeline.
- Orange windmill with a meme.
- Final cut featuring all three memes.

The browser tab title changes during the sequence through `g.pageTitle(...)`. That behavior applies to the web edition.

The patterns continue to use the existing fairness rules, including dashed outlines before box jumps, telegraphs, and safe spots. Damage values remain unchanged, while some patterns are denser and overlap more.

---

## Endings and credits

The current game includes an `Outro` configuration in `game.js`, under `outro`.

After the finale, the ending sequence can play:

1. Ending video.
2. Credits.
3. Two seconds of black.
4. Title screen.

The ending video is located at:

`videos (MP3)/ending video.mp4`

Hold `X` to skip the video where supported.

Customize the credits through:

```js
outro: {
  credits: {
    cards: [
      // Credit cards
    ]
  }
}
```

Use `outro.order` to change the sequence or remove stages. For example:

```js
outro: {
  order: ["video", "black"]
}
```

This example omits the credits stage.

---

## Update history

The following notes document major features added across the engine's development. Some features are specific to the current encounter in `game.js` rather than general-purpose defaults.

### v0.7.0

#### Mobile controls

- Improved multi-touch tracking so every finger is tracked separately.
- Sliding between directional buttons releases the previous direction and presses the new one without requiring the player to lift their finger.
- Added a small `B` pause button above `X`.

#### Memes

- Added `g.popup` and the `Popups` system in `script.js`.
- Added the `meme1`, `meme2`, and `meme3` images.
- Added the Phase 2 attacks `p2_meme1`, `p2_meme2`, and `p2_meme3`.
- Added `P3_EVENTS` for timed meme appearances, visual disturbances, dialogue, captions, and custom callbacks.

#### Phase 3 rework

- Rebuilt Phase 3 as a music-synchronized animation.
- Added a timeline at the bottom and subtitles at the top-left.
- Added loading, missing-texture, mouse-cursor, copy/paste, buffering, draggable error-window, stop-motion, video-player, gravity, error pop-up, bouncing-ball, spinning-timeline, and other effects.
- Added `g.pageTitle(...)` for changing the browser tab title during the animation.
- Retained the telegraph, safe-spot, and battle-box fairness rules.

#### Ending

- Added an ending-video stage.
- Added configurable credits through `outro.credits.cards`.
- Added a black-screen interval and return to the title screen.
- Added `outro.order` for changing the order or removing stages.

### v0.6.1

- Reworked interrupt slashes through `enemy.interrupt` and `INT_TABLE` in `script.js`.
- Added faster and more unpredictable slashes as fewer blocks remain, including double and triple cuts at very low block counts.
- Made interrupt slashes deal nonlethal chip damage.
- Added mobile controls with four directional buttons, `X`, and `Z`.
- In blue soul mode, `Z` also jumps.
- Changed Phase 1 music timing so music starts when the first line begins, using `music: "next"` in the opening sequence.
- Added a flash after the first attack through `flashAfterFirstAttack`.
- Retained `musicAfterFirstAttack: true` for the older music timing behavior.
- Added the Phase 2 `friendSlash(imageKey, [line1, line2])` helper, which shows a screenshot beside Sans, displays dialogue, cuts the image into pieces, and sends the slash into the battle box.
- Added `p2_friend1`, `p2_friend2`, and `p2_friend3`.

### v0.6.0

#### Camera

- Added camera controls through `g.cam(...)`, `g.cam("reset", frames)`, `g.camKick(...)`, and `g.tilt(...)`.
- Added cinematic bars, hit-stop, background pulse, visual rings, and floating text through `g.bars(...)`, `g.freeze(...)`, `g.pulse(...)`, `g.ring(...)`, and `g.float(...)`.

#### Cutscenes

- Added camera movement, cinematic bars, zoom kicks, aberration, freeze, pulse, rings, soul positioning, interface fade-in, and title-card support to cutscene steps.

#### Phase 1 opening

- Added `opening: { skippable, steps: [...] }` for a battle-scene cinematic before the first turn.
- Added `enemyFirst: true` to make Sans attack before the menu appears.
- Added `musicAfterFirstAttack: true` to delay music until the opening attack ends.
- Added the `p1_opening` attack with a stretched battle box, camera movement, bones, shifting blaster grids, and squeeze effects.
- Added `P1_DMG` for opening-attack damage configuration.

#### Camera tilt

- Added the phase-level `tilt` configuration for occasional camera tilts, including a warning before the tilt.
- Added `noTilt: true` for attacks that should not use camera tilt.

#### Interrupt slashes

- Added Phase 2 interrupt slashes through `enemy.interrupt`.
- Added warning indicators before a slash lands.
- Allowed fast player actions to cancel a pending slash by starting Sans's next attack.
- Ensured interrupt damage does not kill the player.

#### Phase 2 and Phase 3

- Added stronger repel effects for blocked Phase 2 attacks.
- Replaced Phase 3 slash attacks with orb fans and falling-bone patterns.

#### Menus

- Added a title screen with `START`, `OPTIONS`, and `CONTROLS`.
- Added a pause menu with `RESUME`, `OPTIONS`, `CONTROLS`, `RESTART FIGHT`, and `QUIT TO TITLE`.
- Added options for visual effects, music, sound effects, screen shake, camera behavior, flashes, and glitches.
- Added title-menu styling through `menu: { color, mote, sub }`.
- Redrew battle buttons, lists, and the attack bar.

#### VFX

- Added stars, light shafts, enemy aura, shockwave rings, floating damage numbers, bullet trails, blaster charge glow, sparks, box ripples, HP lag bars, low-HP pulses, hit-stop, and a heart-shatter Game Over effect.
- Expanded `VFX.spawn` with shape options and additional behavior controls.

### v0.5.0

- Added stronger slash-hit effects with screen glitches, shaking, and sparks.
- Added `DMG.slash` for slash damage values.
- Added `SLASH_FX` configuration, with `glitch: 0` disabling the glitch component.
- Added `g.glitch(frames, power)` and `g.aberrate(px, frames)`.
- Added the `glitch` cutscene-step option.
- Added orange soul mode through `mode: "orange"` and `a.setMode("orange")`.
- Added a keep-moving warning and recurring damage if the soul remains stationary in orange mode.
- Added orange mode to supported Phase 2.5 and Phase 3 segments.
- Improved `spinArm` with gradual acceleration and ghost trails.
- Changed Phase 3 to use a 2:32 song and 15 segments.
- Added a one-time random 20–50 HP heal triggered by pressing `Z`, configured through `P3_HEAL`.
- Connected the healing sound effect through `sounds.heal`.
- Added the Phase 3 finale cinematic, including cinema bars, camera zoom, cracks, a boom, sprite shattering, god rays, shockwaves, soul pulses, white-out, and a title.
- Added `phases[n].finale`.
- Added `phases[n].mercySlash = { delay }`, which can destroy the MERCY button after a delay and leave it unavailable in later phases.

### v0.1.0 — Dumb_Artist's UT Battle Engine

#### Phase 1

Added seven new attacks through `P1_ATTACKS` in `game.js`:

- RIB CAGE
- GRAVITY
- CROSSFIRE
- PRISM
- COLOR CODE
- PINWHEEL
- GAUNTLET

These attacks feature attack-name fades, telegraphs, and safe spots. `P1_DMG` controls Phase 1 damage.

#### Cinematic Game Over

Added a cinematic Game Over sequence featuring color drain, soul effects, shattering, a letter-by-letter `GAME OVER` title, a final enemy line, and `TRY AGAIN` / `QUIT TO TITLE` choices.

Press `Z` during the sequence to skip ahead.

Configure the sequence in `game.js`:

```js
gameOver: {
  title: "GAME OVER",
  color: "#ffffff",
  music: "audio/gameover.mp3",
  volume: 1,
  speaker: "Sans",
  quotes: [],
  counter: 0,
  titleKey: "Z"
}
```

The available values depend on the existing Game Over implementation.

#### Custom menus

Added customizable title and pause menus through `menu` and `pause`, including:

- Custom menu entries.
- Custom pages.
- Hidden menu elements.
- Titles and labels.
- A customizable controls page.

#### Custom patterns

Added reusable patterns through:

```js
patterns: {
  name(a, options) {
    // Custom attack pattern
  }
}
```

Use them through `a.pattern("name", {...})`.

#### Additional helpers

- Added `a.at(frame, fn)` for frame-specific callbacks.
- Added `a.vring(x, y, {...})` for visual shockwaves.
- Added a small battle watermark, configurable through `watermark: { show: false }`.

---

## Notes and troubleshooting

- This is the **Electron desktop edition**. Use `npm start` to launch it from source rather than opening `index.html` directly.
- Run `npm install` after downloading or cloning the source if dependencies are not installed.
- Run `npm run dist` to build the Windows installer.
- `dist/` contains generated build output; it is not the source code.
- `pkg/` contains resources for the separate packaged web edition.
- Customize encounters through `game.js`. Modify `script.js` only when changing engine behavior.
- Preserve asset names and paths unless you also update the corresponding references in the game configuration.
- Missing files, incorrect paths, and JavaScript errors may produce an error banner or console messages, depending on the build. In the browser edition, inspect the developer console with `F12`.
- Some features described in the update history are specific to this game's current configuration and may require additional setup before they can be reused in another encounter.
- The browser edition's autoplay behavior, mobile controls, browser storage, and tab-title effects may not apply identically to the Electron desktop edition.
- This is an Undertale-inspired fan project and is not affiliated with or endorsed by Toby Fox. Make sure you have permission to redistribute third-party sprites, music, sound effects, fonts, and other assets included in any release.

**Built by Dark Byte Studios.**
