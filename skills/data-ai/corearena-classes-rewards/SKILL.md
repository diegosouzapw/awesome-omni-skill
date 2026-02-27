---
name: corearena-classes-rewards
description: 'Troubleshooting class selection, tier upgrades, experience, and nugget economy'
---

# Classes & Economy Troubleshooting

## Diagnostic Flow — "Player can't use class / not getting rewards"

1. **Ask what error message** they see on join
2. If "no class": check `Give_Random_Class_If_Not_Selected` in settings.yml and whether player has class permissions
3. If no nuggets after game: check `Experience.Reward_On_Escape` setting

## Common Mistakes

- **Potion level is 1-based in config but 0-based internally** — level 1 in YAML = amplifier 0 in Bukkit. Users see "weaker than expected" effects because they think level 1 = amplifier 1
- **Class permission uses `{file_lowercase}` auto-replacement** — `classes/Warrior.yml` requires permission `corearena.class.warrior`. Users often try the exact filename case
- **`Experience.Reward_On_Escape: false` blocks nuggets on disconnect** — players who leave via command or disconnect get no nuggets. Only natural arena ends (countdown, all monsters killed) award nuggets
- **Upgrade "locked" depends on current arena phase** — the upgrade's `Available_From_Phase` must be reached in the current game. This resets each game
