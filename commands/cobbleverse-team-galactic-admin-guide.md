# Cobbleverse — Team Galactic Admin Guide
> Datapack: `COBBLEVERSE-RCT-DP-v19.zip` — powered by the **RCT mod** (Radical Cobblemon Trainers)

---

## ⚠️ Important Notes

- These NPCs are **not** spawned with `/spawnnpc` — they use the RCT mod's own command system (`/rctmod`)
- Structure NPCs (Mars, Jupiter, Saturn, Cyrus, Charon) require a **Trainer Spawner block** + a **Signature Item** inserted by the player to trigger the battle
- There are **two variants** of each commander — a world-spawning version (with hex ID suffix) and a structure version (prefixed `team_galactic_`)
- All battles are **Gen 9 Doubles** format

---

## 📋 NPC Variants — Two Systems

| Commander | World-Spawn ID | Structure ID |
|---|---|---|
| Mars | `commander_mars_03c2` | `team_galactic_commander_mars` |
| Jupiter | `commander_jupiter_041d` | `team_galactic_commander_jupiter` |
| Saturn | `commander_saturn_041f` | `team_galactic_commander_saturn` |
| Cyrus | — | `team_galactic_cyrus` |
| Charon | — | `team_galactic_charon` |

---

## 🗺️ Progression Chain (Structure NPCs)

```
team_galactic_commander_mars
        ↓
team_galactic_commander_jupiter  (requires beating Mars)
        ↓
team_galactic_commander_saturn   (requires beating Jupiter)
        ↓
team_galactic_cyrus              (requires beating Saturn)
```

---

## 🧪 Signature Items (insert into Trainer Spawner to trigger battle)

| NPC | Signature Item | Give Command |
|---|---|---|
| Mars | `lumymon:verity_proof` | `/give <player> lumymon:verity_proof` |
| Jupiter | `lumymon:acuity_proof` | `/give <player> lumymon:acuity_proof` |
| Saturn | `lumymon:valor_proof` | `/give <player> lumymon:valor_proof` |
| Cyrus | `lumymon:red_chain` | `/give <player> lumymon:red_chain` |
| Charon | `lumymon:heat_splinter` | `/give <player> lumymon:heat_splinter` |

---

## 🛠️ Admin Commands

### Summon an NPC (temporary)
```
/rctmod trainer summon <trainer_id>
/rctmod trainer summon <trainer_id> <x> <y> <z>
```

### Summon an NPC (persistent — stays in world, use for structure bosses)
```
/rctmod trainer summon_persistent team_galactic_commander_mars
/rctmod trainer summon_persistent team_galactic_commander_jupiter
/rctmod trainer summon_persistent team_galactic_commander_saturn
/rctmod trainer summon_persistent team_galactic_cyrus
/rctmod trainer summon_persistent team_galactic_charon
```

### Remove a persistent NPC
```
# Step 1 - get UUID by looking at the NPC and running:
/data get entity @e[type=rctmod:trainer,limit=1,sort=nearest]

# Step 2 - unregister it:
/rctmod trainer unregister_persistent <entity_uuid>

# Step 3 - kill the entity:
/kill @e[type=rctmod:trainer,limit=1,sort=nearest]
```

### Place a Trainer Spawner block
```
/setblock ~ ~ ~ rctmod:trainer_spawner
# Locked variant (players can't remove signature item):
/setblock ~ ~ ~ rctmod:trainer_spawner[lootable=false,locked=true]
```

### Fix a player's progression (force register a defeat)
```
/rctmod player set defeats <trainer_id> <playername> 1
```

### Check what a player has beaten
```
/rctmod player get progress <playername>
```

### Check a player's level cap
```
/rctmod player get level_cap <playername>
```

### Check how many times a player defeated a specific trainer
```
/rctmod player get defeats <trainer_id> <playername>
```

### Try to naturally spawn a trainer near a player
```
/rctmod trainer spawn_for <playername>
```

### Get info on a trainer
```
/rctmod trainer get type <trainer_id>
/rctmod trainer get required_level_cap <trainer_id>
/rctmod trainer get required_defeats rctmod <trainer_id>
/rctmod trainer get max_trainer_defeats <trainer_id>
```

---

## 🔧 Common Fix — Player Can't Fight a Structure Boss

If a player killed an NPC instead of battling it properly, or the spawner is broken:

```bash
# 1. Give the player the signature item
/give <playername> lumymon:acuity_proof   # for Jupiter

# 2. Manually register prerequisite defeats if needed
/rctmod player set defeats team_galactic_commander_mars <playername> 1

# 3. Place a trainer spawner at the boss location
/setblock ~ ~ ~ rctmod:trainer_spawner

# 4. OR summon the NPC directly
/rctmod trainer summon_persistent team_galactic_commander_jupiter
```

---

## 👾 Team Galactic NPC Teams

### Commander Mars — `team_galactic_commander_mars`
| Pokémon | Lvl | Item | Notable |
|---|---|---|---|
| Purugly | 75 | Silk Scarf | Fake Out + Knock Off |
| Weavile | 74 | Life Orb | Fake Out + Ice Shard |
| Salamence | 75 | Mega Stone | **Mega**, Intimidate |
| Crobat | 73 | Sharp Beak | Tailwind + Taunt |
| Raichu | 74 | Focus Sash | Lightning Rod |
| Espeon | 75 | Life Orb | **Magic Bounce** |

### Commander Jupiter — `team_galactic_commander_jupiter`
*(team data same as structure Jupiter, check `team_galactic_commander_jupiter.json`)*

### Commander Cyrus — `team_galactic_cyrus`
| Pokémon | Lvl | Item | Notable |
|---|---|---|---|
| Honchkrow | 88 | Scope Lens | Super Luck |
| Crobat | 85 | Sharp Beak | Tailwind setter |
| Gyarados | 86 | Leftovers | Intimidate |
| Weavile | 85 | Life Orb | Priority |
| Corviknight | 87 | Leftovers | **G-Max** |
| Houndoom | 88 | Life Orb + Houndoominite | **Mega** |

---

## 📁 Key Files in `COBBLEVERSE-RCT-DP-v19.zip`

| Path | What it contains |
|---|---|
| `data/rctmod/mobs/trainers/single/` | Spawn rules, required defeats, signature items |
| `data/rctmod/trainers/` | Pokémon teams, movesets, EVs/IVs |
| `data/rctmod/dialogs/trainers/single/` | NPC dialogue |
| `data/rctmod/loot_table/trainers/single/` | Drop tables on defeat |

---

*Guide based on `COBBLEVERSE-RCT-DP-v19.zip` — update version number if datapack is updated.*
