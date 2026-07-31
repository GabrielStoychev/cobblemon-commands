# WorldCrystal Mod — Setup & Usage Guide

A server-side Fabric 1.21.1 mod that spawns interactive crystal zones. Players punch blocks or Cobblemon Pokémon entities to break them, triggering particles, debuffs, explosions, projectiles and rewards. Integrates with GlobalChallenge.

---

## GitHub Setup

### First Time

```bash
# 1. Extract the source zip to a folder
# e.g. worldcrystal-mod-source-v43.zip → C:\PokeKingdom\worldcrystal\

# 2. Open terminal in that folder and initialize git
git init
git add .
git commit -m "WorldCrystal v43 — initial commit"

# 3. Create a repo on GitHub (github.com → New repository)
# Name: worldcrystal  |  Private  |  No README

# 4. Push
git branch -M main
git remote add origin https://github.com/YOURUSERNAME/worldcrystal.git
git push -u origin main
```

### Pushing Future Updates

```bash
git add .
git commit -m "describe what changed"
git push
```

---

## Building the Jar

Requires Java 21.

```bash
# In the worldcrystal folder
./gradlew build          # Mac/Linux
gradlew.bat build        # Windows
```

Output jar: `build/libs/worldcrystal-x.x.x.jar`

Drop into your server's `mods/` folder and restart.

---

## Installation

1. Drop `worldcrystal-x.x.x.jar` into `mods/`
2. Optionally install **GlobalChallenge** for break tracking integration
3. Optionally install **Cobblemon** if using Pokémon crystal entities
4. Restart — configs generate at `config/worldcrystal/zones/`

---

## Quick Start

### 1. Select a Zone In-Game

Hold a **Diamond Pickaxe** (requires op level 2):

| Action | Result |
|---|---|
| **Left-click** a block | Sets Corner 1 |
| **Right-click** a block | Sets Corner 2 |

> **Always select vertically.** Aim for at least 8-10 blocks of Y height so `ground`, `air` and `ceiling` spawn modes all have room to work.

```
/wc save myzone
```

Creates `config/worldcrystal/zones/myzone.json`.

### 2. Edit the JSON

Configure your blocks, effects and rewards. Then:
```
/wc reload
```

### 3. Run the Event

```
/wc start myzone
/wc stop myzone
```

> After editing a JSON always do `/wc stop` → `/wc reload` → `/wc start`. Crystals already spawned keep their old config until fully restarted.

---

## Commands

All commands require **op level 2**.

| Command | Description |
|---|---|
| `/wc save <name>` | Save diamond pickaxe selection as a zone |
| `/wc start <name>` | Start zone — spawns crystals |
| `/wc stop <name>` | Stop zone — removes all crystals, cancels respawns |
| `/wc clear <name>` | Remove crystals only (keeps zone active) |
| `/wc spawn <name> <amount>` | Manually spawn more crystals |
| `/wc reload` | Reload all zone configs from disk |
| `/wc list` | List all configured zones |
| `/wc info <name>` | Show zone details |
| `/wc delete <name>` | Delete a zone permanently |
| `/wc selection` | Show your current diamond pickaxe selection |
| `/wc clearsel` | Clear your selection |

---

## Zone JSON — Top-Level Fields

| Field | Description |
|---|---|
| `name` | Must match the filename (without `.json`) |
| `corner1` / `corner2` | Two opposite corners of the 3D zone box |
| `dimension` | `minecraft:overworld`, `minecraft:the_nether`, `minecraft:the_end` |
| `active` | Managed automatically — don't edit |
| `initialSpawnAmount` | How many crystals spawn on `/wc start` |
| `autoRefillIntervalSeconds` | Seconds between auto top-up. `-1` = disabled |

---

## Block Crystal Fields

| Field | Description |
|---|---|
| `blockId` | Any vanilla block ID |
| `weight` | Spawn probability relative to other blocks |
| `hitsRequired` | Hits needed to break |
| `spawnMode` | See Spawn Modes below |
| `randomRespawn` | `true` = random position on respawn |
| `respawnSeconds` | Seconds before respawn. `-1` = never |
| `reportToGlobalChallenge` | Count toward active `BREAK_BLOCK` challenge |
| `rewardCommands` | Commands run on breaker. Use `{player}` as placeholder |

---

## Spawn Modes

| Mode | Behaviour | Use when |
|---|---|---|
| `ground` | Scans downward for solid block | Open terrain |
| `air` | Places in upper half of zone Y range | Open sky |
| `ceiling` | Scans upward for solid block underside | Caves, indoor areas with a roof |
| `surface` | Always at zone minimum Y | Flat floor |
| `random` | Anywhere in zone that is air | Universal fallback |

> No ceiling in your zone? Use `air` instead — `ceiling` fails silently in open sky.

---

## Particle Format

```
"particleId:count:spreadX:spreadY:spreadZ:speed"
```

Example: `"minecraft:crit:12:0.3:0.3:0.3:0.1"`

**Available particles:**
```
minecraft:crit              minecraft:enchant           minecraft:end_rod
minecraft:flame             minecraft:smoke             minecraft:soul_fire_flame
minecraft:dragon_breath     minecraft:witch             minecraft:flash
minecraft:explosion         minecraft:heart             minecraft:poof
```

---

## Effect Format

```json
{ "effect": "minecraft:slowness", "durationSeconds": 5, "amplifier": 1, "showParticles": true }
```

`amplifier`: `0` = level I, `1` = level II, `2` = level III

**Debuffs:**
```
minecraft:slowness      minecraft:blindness     minecraft:nausea
minecraft:wither        minecraft:poison        minecraft:darkness
minecraft:weakness      minecraft:hunger        minecraft:levitation
minecraft:instant_damage    minecraft:mining_fatigue
```

**Buffs:**
```
minecraft:speed         minecraft:regeneration  minecraft:strength
minecraft:jump_boost    minecraft:resistance    minecraft:absorption
minecraft:fire_resistance   minecraft:night_vision  minecraft:haste
```

---

## Explosion Format

```json
"explosion": { "power": 2.5, "destroyBlocks": false, "createFire": false }
```

| Power | Equivalent |
|---|---|
| 1.5 | Small blast |
| 2.5 | Medium |
| 4.0 | TNT strength |

Set `"explosion": null` to disable.

---

## On-Break Projectiles

```json
"projectiles": [
  { "type": "wither_skull", "count": 3, "direction": "outward", "speed": 1.0, "explosionPower": 0 }
]
```

**Directions:** `random` `outward` `up` `down`

**Types:**

| Type | Effect |
|---|---|
| `fireball` | Explodes on impact |
| `small_fireball` | Sets target on fire |
| `wither_skull` | Applies wither |
| `charged_wither_skull` | Bigger explosion + wither |
| `dragon_fireball` | Lingering cloud |
| `arrow` | Arrow damage |
| `spectral_arrow` | Arrow + glowing |
| `snowball` | Knockback only |
| `egg` | Knockback only |

---

## Shoot At Player

Crystal fires at nearest player **while alive**.

```json
"shootAtPlayer": {
  "enabled": true,
  "projectileType": "wither_skull",
  "intervalSeconds": 5,
  "rangeBlocks": 15,
  "speed": 1.0,
  "explosionPower": 0
}
```

Set `"shootAtPlayer": null` to disable. Keep `intervalSeconds` at 4+ to avoid TPS issues.

---

## Lightning

```json
"lightningOnBreak": true,        ← real strike, deals damage
"lightningVisualOnBreak": true   ← visual only, no damage
```

Only one should be `true` at a time.

---

## Pokémon Crystal Fields

Spawns a Cobblemon Pokémon players punch like a block. No Pokéball, no battle — just left-click hits.

```json
"pokemonCrystals": [
  {
    "enabled": true,
    "species": "cobblemon:crystal",
    "level": 50,
    "hitsRequired": 3,
    "respawnSeconds": 30,
    "randomRespawn": true,
    "reportToGlobalChallenge": true,
    "rewardCommands": ["give {player} cobblemon:rare_candy 1"],
    "hitParticles": ["minecraft:enchant:8:0.3:0.3:0.3:0.05"],
    "breakParticles": ["minecraft:crit:20:0.5:0.5:0.5:0.2"],
    "hitSound": "minecraft:block.amethyst_cluster.hit",
    "hitSoundVolume": 1.0, "hitSoundPitch": 1.2,
    "breakSound": "minecraft:block.amethyst_cluster.break",
    "breakSoundVolume": 1.0, "breakSoundPitch": 0.9,
    "debuffsOnHit": [],
    "debuffsOnBreak": [],
    "explosion": null,
    "lightningOnBreak": false,
    "lightningVisualOnBreak": false,
    "shootAtPlayer": null
  }
]
```

---

## Finite Events

To make crystals that never respawn once broken:

```json
"autoRefillIntervalSeconds": -1,
"respawnSeconds": -1
```

Set on all blocks. The initial spawn depletes as players break crystals — nothing comes back until `/wc start` is run again.

---

## GlobalChallenge Integration

Set `"reportToGlobalChallenge": true` on any block, then in GlobalChallenge use:

```json
"trigger": {
  "type": "BREAK_BLOCK",
  "blockIds": ["minecraft:amethyst_cluster", "minecraft:glowstone"]
}
```

For Pokémon crystals:
```json
"trigger": {
  "type": "BREAK_BLOCK",
  "blockIds": ["worldcrystal:pokemon_crystal"]
}
```
