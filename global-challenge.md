# GlobalChallenge Mod — Setup & Usage Guide


## Config Folder Structure

```
config/globalchallenge/
├── challenges/          ← one JSON file per challenge
│   ├── galaxy_boss_1.json
│   ├── summer_water_hunt.json
│   └── ...
├── queues/              ← queue configs (run challenges back to back)
│   └── galaxy_event.json
├── state.json           ← active challenge state (auto-managed)
├── pending_rewards.json ← crash-safe reward queue (auto-managed)
└── reward_log.json      ← full audit log of every reward delivered
```

---

## Commands

All commands require **op level 2**.

### Challenge Commands

| Command | Description |
|---|---|
| `/gc start <id>` | Start a challenge by its JSON id |
| `/gc stop` | Stop the currently running challenge |
| `/gc list` | List all loaded challenges |
| `/gc status` | Show current challenge progress |
| `/gc top` | Show the leaderboard |
| `/gc reload` | Reload all configs from disk |

### Queue Commands

| Command | Description |
|---|---|
| `/gc queue start <id>` | Start a queue (runs challenges in order) |
| `/gc queue stop` | Stop the active queue |
| `/gc queue skip` | Skip to the next challenge in the queue |
| `/gc queue list` | List all configured queues |
| `/gc queue status` | Show current queue position |

### Reward Commands

| Command | Description |
|---|---|
| `/gc rewards status` | Show how many rewards are queued |
| `/gc rewards log [count]` | Review last N reward deliveries with timestamps |
| `/gc rewards pending` | Inspect the crash-recovery pending file |

---

## Challenge JSON Fields

```json
{
  "id": "my_challenge",
  "displayName": "My Challenge",
  "autoStart": false,
  "durationMinutes": 60,
  "goalCount": 100,
  "bossBarColor": "PURPLE",
  "bossBarStyle": "PROGRESS",
  "leaderboardTitle": "&eTop Hunters",
  "leaderboardDisplayCount": 3,
  "progressBroadcastIntervalMinutes": 15,
  "startMessage": "&a[GlobalChallenge] The challenge begins!",
  "completeMessage": "&a[GlobalChallenge] Goal reached!",
  "timeoutMessage": "&c[GlobalChallenge] Failed! {current}/{goal}.",
  "trigger": { ... },
  "rewards": [ ... ],
  "topRewardTiers": [ ... ]
}
```

### Boss Bar Colors
`PINK` `BLUE` `RED` `GREEN` `YELLOW` `PURPLE` `WHITE`

### Boss Bar Styles
`PROGRESS` `NOTCHED_6` `NOTCHED_10` `NOTCHED_12` `NOTCHED_20`

---

## Trigger Types

### Pokémon Triggers

```json
"trigger": {
  "type": "CATCH_TYPE",
  "pokemonType": "water",
  "minLevel": 10,
  "requireShiny": false,
  "requiredAspect": ""
}
```

| Type | Fires when |
|---|---|
| `CATCH_ANY` | Player catches any Pokémon |
| `CATCH_NAMED` | Player catches specific species |
| `CATCH_TYPE` | Player catches a specific elemental type |
| `DEFEAT_ANY` | Player defeats any Pokémon |
| `DEFEAT_NAMED` | Player defeats specific species |
| `DEFEAT_TYPE` | Player defeats a specific elemental type |
| `DEFEAT_HIGHER_LEVEL` | Player defeats a Pokémon higher level than theirs |
| `LEVEL_UP_NAMED` | A Pokémon levels up (filter by name or leave blank for any) |
| `EVOLVE_NAMED` | A Pokémon evolves (filter by name or leave blank for any) |

### Multiple Species (pokemonNames)

```json
"trigger": {
  "type": "DEFEAT_NAMED",
  "pokemonNames": ["blastoise", "grovyle", "aipom"],
  "minLevel": 90
}
```

### Aspect Filter (custom skins/variants)

```json
"trigger": {
  "type": "DEFEAT_NAMED",
  "pokemonNames": ["blastoise", "grovyle", "aipom"],
  "requiredAspect": "one-piece"
}
```

### Block Triggers

```json
"trigger": {
  "type": "BREAK_BLOCK",
  "blockIds": ["minecraft:amethyst_cluster", "minecraft:glowstone"]
}
```

| Type | Fires when |
|---|---|
| `BREAK_BLOCK` | Player breaks one of the listed blocks |
| `BREAK_ANY` | Player breaks any block |
| `PLACE_BLOCK` | Player places one of the listed blocks |

---

## Reward Format

### Base Rewards (everyone who participates)

```json
"rewards": [
  {
    "type": "COMMAND",
    "value": "give {player} cobblemon:rare_candy 1",
    "delayTicks": 0
  },
  {
    "type": "MESSAGE",
    "value": "&7Participation reward: &eRare Candy x1!",
    "delayTicks": 20
  }
]
```

### Top Reward Tiers

```json
"topRewardTiers": [
  {
    "topCount": 1,
    "label": "MVP",
    "announceMessage": "&d{player} is the MVP with {count} defeats!",
    "rewards": [
      {
        "type": "COMMAND",
        "value": "give {player} cobblemon:master_ball 1",
        "delayTicks": 0
      }
    ]
  },
  {
    "topCount": 10,
    "label": "Top 10",
    "announceMessage": "&a{player} finished #{rank} with {count} defeats!",
    "rewards": [
      {
        "type": "COMMAND",
        "value": "give {player} cobblemon:rare_candy 5",
        "delayTicks": 0
      }
    ]
  }
]
```

**Notes:**
- Tiers are checked in ascending `topCount` order — rank 1 only gets the MVP tier, not Top 10
- To give tier winners the base participation reward too, add it as an extra command inside the tier's `rewards` list
- `{player}` — player name placeholder
- `{rank}` — their leaderboard position
- `{count}` — their contribution count
- Rewards are staggered 1 per second automatically (crash-safe queue)

### Reward Types

| Type | Description |
|---|---|
| `COMMAND` | Runs any server command. Use `{player}` as placeholder |
| `MESSAGE` | Sends a private message to the player |
| `BROADCAST` | Broadcasts to the whole server |

---

## Queue JSON

Runs a series of challenges back-to-back with a countdown between them.

```json
{
  "id": "galaxy_event",
  "challenges": ["galaxy_boss_1", "galaxy_evolve_any", "galaxy_level_up_high"],
  "countdownSeconds": 60,
  "announceCountdown": true,
  "loop": false
}
```

Place in `config/globalchallenge/queues/`.

---

## Reward Safety

**Staggered delivery** — rewards fire 1 per second, never all at once. With 50 players each getting tier rewards, they trickle out over ~50 seconds instead of spiking the server in a single tick.

**Crash recovery** — `pending_rewards.json` is updated after every dispatch. If the server crashes mid-event, pending rewards resume automatically on next startup — nothing is lost.

**Audit log** — every reward (delivered or skipped for offline player) is logged to `reward_log.json`. Use `/gc rewards log 50` to review the last 50 entries in-game.

---

## WorldCrystal Integration

Set `"reportToGlobalChallenge": true` on any block in a WorldCrystal zone. Then use `BREAK_BLOCK` trigger:

```json
"trigger": {
  "type": "BREAK_BLOCK",
  "blockIds": ["minecraft:amethyst_cluster", "minecraft:glowstone"]
}
```

Players breaking those blocks via WorldCrystal will count toward the challenge automatically.
