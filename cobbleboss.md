# CobbleBoss

A server-side Fabric mod for **Cobblemon 1.7.3+** that adds configurable boss Pokémon spawning to defined world regions.

---

## Features

- **Region-based spawning** — mark any two corners in-world with a golden pickaxe and define a boss arena
- **Weighted boss pools** — each region has its own pool of possible bosses, with configurable weights
- **Fully configurable bosses** — species, level, stat multiplier(doesn't work), visual scale, forced movesets, aspects (skins), display names, and per-boss rewards
- **Uncatchable bosses** — all spawned bosses use Cobblemon's native `uncatchable=true` property
- **Aspect support** — native compatibility with Aspect skin resource packs
- **Reward commands** — run any command when a boss is defeated, with per-reward chance values
- **Live reload** — reload both config files without restarting the server
- **Player-proximity gating** — regions only spawn new bosses when a player is nearby, preventing uncontrolled stacking in remote arenas
- **Server-only** — no client mod required

---

## Configuration

All config lives in `config/cobbleboss/` on the server.

---

### `bosses.json`

Defines every possible boss. This is a JSON array — each entry is one boss type.

```json
[
  {
    "id": "galaxy_rayquaza",
    "species": "rayquaza",
    "level": 100,
    "statMultiplier": 1.5,
    "visualScale": 1.4,
    "moves": ["dragonclaw", "earthquake", "stoneedge", "swordsdance"],
    "aspects": ["galaxy"],
    "aggroRange": 0.0,
    "weight": 1.0,
    "displayName": "&5&lGalaxy Rayquaza",
    "healthMultiplierBonus": 1.0,
    "rewards": [
      {
        "command": "give %player% minecraft:netherite_ingot 3",
        "asPlayer": false,
        "chance": 1.0
      },
      {
        "command": "say %player% has defeated the Galaxy Rayquaza!",
        "asPlayer": false,
        "chance": 1.0
      }
    ]
  }
]
```

#### Boss Fields

| Field | Type | Description |
|---|---|---|
| `id` | String | Unique identifier. Used in `/cobbleboss spawn` and `regions.json` bossPool. |
| `species` | String | Cobblemon species name (e.g. `rayquaza`) or namespaced custom species (e.g. `cobbleboss:bidoof_titan`). |
| `level` | Int | Boss level. Keep at or below your server's `maxPokemonLevel` (default 100). |
| `statMultiplier` | Double | Maxes IVs and scales EVs. `1.0` = unchanged. `1.5` = modest boost. |
| `visualScale` | Double | Model size multiplier. `1.0` = normal. `2.0` = double size. High values (10+) may impact performance. |
| `moves` | Array | Up to 4 move IDs in **Showdown format** (lowercase, no spaces or underscores). e.g. `dragonclaw`, `willowisp`, `stoneedge`. |
| `aspects` | Array | Cobblemon aspect strings (e.g. `["galaxy"]`, `["shiny"]`). Empty array for none. |
| `aggroRange` | Double | Reserved for future use. Set to `0.0`. |
| `weight` | Double | Relative spawn weight within a region's bossPool. Higher = more common. |
| `displayName` | String | Name shown above the boss. Supports Minecraft color codes (`&5`, `&l`, etc.). |
| `healthMultiplierBonus` | Double | Currently unused. Keep at `1.0`. |
| `rewards` | Array | Commands to run when this boss is defeated. See below. |

#### Move Name Format

Cobblemon uses **Pokémon Showdown's internal move IDs** — lowercase with all spaces and punctuation removed:

| In-game name | Config value |
|---|---|
| Dragon Claw | `dragonclaw` |
| Stone Edge | `stoneedge` |
| Will-O-Wisp | `willowisp` |
| Swords Dance | `swordsdance` |
| Flare Blitz | `flareblitz` |

#### Reward Fields

| Field | Type | Description |
|---|---|---|
| `command` | String | Any server command. Use `%player%` as a placeholder for the winning player's name. |
| `asPlayer` | Boolean | `false` = run as server console. `true` = run as the player. |
| `chance` | Double | Probability this reward fires. `1.0` = always, `0.5` = 50% chance. |

---

### `regions.json`

Defines every spawn region. This is a JSON array — each entry is one region.

```json
[
  {
    "name": "phase1",
    "world": "minecraft:overworld",
    "minX": -8501,
    "minY": 211,
    "minZ": 5437,
    "maxX": -8374,
    "maxY": 239,
    "maxZ": 5665,
    "active": true,
    "bossPool": [
      { "bossId": "galaxy_rayquaza" },
      { "bossId": "galaxy_regigigas", "weight": 0.5 }
    ],
    "maxBosses": 10,
    "spawnIntervalSeconds": 10,
    "cooldownSeconds": 10
  }
]
```

#### Region Fields

| Field | Type | Description |
|---|---|---|
| `name` | String | Unique region name. Used in commands. |
| `world` | String | World dimension ID (e.g. `minecraft:overworld`, `minecraft:the_nether`). |
| `minX/Y/Z` | Int | One corner of the spawn box. |
| `maxX/Y/Z` | Int | Opposite corner of the spawn box. |
| `active` | Boolean | `true` = region is live and spawning. `false` = paused. |
| `bossPool` | Array | Which bosses can spawn here. Each entry needs a `bossId` matching one in `bosses.json`. Optional per-entry `weight` overrides the boss's own weight for this region specifically. |
| `maxBosses` | Int | Maximum number of live bosses allowed in this region simultaneously. |
| `spawnIntervalSeconds` | Int | Seconds between spawn attempts while below `maxBosses`. |
| `cooldownSeconds` | Int | After the region is fully wiped (zero bosses), how long to wait before respawning. |

> ⚠️ **Important:** `minY` and `maxY` must be **different values** giving real vertical room (at least 10 blocks). If both are the same, the ground-position finder will almost always fail silently and nothing will spawn.

> ⚠️ **Important:** `bossPool` entries must be **objects** (`{ "bossId": "..." }`), not plain strings (`"..."`). A string array will cause a parse error on reload.

---

## Commands

All commands require **operator level 2** by default.

| Command | Description |
|---|---|
| `/cobbleboss wand` | Toggle marking mode on/off. Hold a golden pickaxe, then left-click corner 1 and right-click corner 2. |
| `/cobbleboss region create <name>` | Save the currently marked area as a new region. |
| `/cobbleboss region toggle <name>` | Enable or disable a region without editing the JSON. |
| `/cobbleboss region list` | List all regions with their current alive boss count. |
| `/cobbleboss spawn <bossId> <regionName>` | Manually spawn a specific boss in a region. Useful for testing. |
| `/cobbleboss reload` | Reload both `bosses.json` and `regions.json` from disk without restarting. |

---

## Creating a Region In-Game

1. Run `/cobbleboss wand` to enter marking mode.
2. Hold a **golden pickaxe**.
3. **Left-click** a block → sets Corner 1.
4. **Right-click** the opposite corner → sets Corner 2. Chat will show the save command.
5. Run `/cobbleboss region create <yourname>`.
6. Open `config/cobbleboss/regions.json` and:
   - Add your bosses to `bossPool`
   - Set `maxBosses`, `spawnIntervalSeconds`, `cooldownSeconds`
   - Widen `minY`/`maxY` to give real vertical range
7. Run `/cobbleboss reload`.

---

## Custom Titan Species (Datapack)

The mod ships with support for custom Cobblemon species via a companion datapack (`cobbleboss-titans-datapack.zip`), which adds amplified-stat versions of small Pokémon as Phase 2 bosses.

### Installation

1. Extract `cobbleboss-titans-datapack.zip` into your world's `datapacks/` folder:
   ```
   world/datapacks/cobbleboss-titans/pack.mcmeta
   world/datapacks/cobbleboss-titans/data/...
   ```
2. **Fully restart the server** — `/reload` does not work for Cobblemon species data.
3. Reference the custom species in `bosses.json` using the namespaced ID:
   ```json
   "species": "cobbleboss:bidoof_titan"
   ```

### Custom Species IDs

| Display Name | Species ID |
|---|---|
| Bidoof Titan | `cobbleboss:bidoof_titan` |
| Chimchar Titan | `cobbleboss:chimchar_titan` |
| Piplup Titan | `cobbleboss:piplup_titan` |
| Turtwig Titan | `cobbleboss:turtwig_titan` |
| Squirtle Titan | `cobbleboss:squirtle_titan` |

### Visuals (Resource Pack)

Custom species visuals (models, textures) are distributed as part of your server's existing `resources.zip`. Ensure players have the resource pack active to see correct models and galaxy skins.

---

## Tips

- **Bosses in remote arenas** — the mod only spawns new bosses when a player is within ~128 blocks of the region's center. This prevents uncontrolled stacking when chunks unload in unvisited areas.
- **Phase toggling** — use `/cobbleboss region toggle <name>` to manually switch between event phases without editing JSON.
- **Galaxy skins** — set `"aspects": ["galaxy"]` in a boss entry. Requires the galaxy resource pack to be active client-side.
- **Testing** — use `/cobbleboss spawn <bossId> <regionName>` to test a specific boss before enabling auto-spawning.
- **Reload vs restart** — `bosses.json` and `regions.json` support live reload via `/cobbleboss reload`. The titan species datapack always requires a full server restart to take effect.

---

## Compatibility

| Mod | Notes |
|---|---|
| Cobblemon 1.7.3 | Required |
| Ledger | Fully compatible — boss entities are excluded from Ledger's kill-logging save path |
| GlobalChallenge | Compatible — use `DEFEAT_NAMED` trigger with the boss's display name (e.g. `"Bidoof Titan"`) |
| LuckPerms | Can be used to restrict `/cobbleboss` commands to specific ranks |
| Lithium | Compatible |

---

## License

All rights reserved. This mod was built for PokéKingdom / CobbleKingdoms. Do not redistribute without permission.
