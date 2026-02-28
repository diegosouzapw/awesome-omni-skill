---
name: starwars
version: 1.0.0
description: "CLI for AI agents to lookup Star Wars universe info for their humans. Uses SWAPI. No auth required."
homepage: https://swapi.dev
metadata:
  openclaw:
    emoji: "⚔️"
    requires:
      bins: ["bash", "curl", "jq"]
    tags: ["starwars", "swapi", "entertainment", "movies", "cli"]
---

# Star Wars Lookup

CLI for AI agents to lookup Star Wars universe info for their humans. "Who played Darth Vader?" — now your agent can answer.

Uses [SWAPI](https://swapi.dev) (Star Wars API). No account or API key needed.

## Usage

```
"Look up Luke Skywalker"
"What planet is Tatooine?"
"List all Star Wars films"
"What species is Chewbacca?"
"Tell me about the Millennium Falcon"
```

## Commands

| Action | Command |
|--------|---------|
| Search characters | `starwars people "name"` |
| Search planets | `starwars planets "name"` |
| List films | `starwars films` |
| Search species | `starwars species "name"` |
| Search starships | `starwars starships "name"` |

### Examples

```bash
starwars people "luke"          # Find character by name
starwars planets "tatooine"     # Find planet by name
starwars films                  # List all films
starwars species "wookiee"      # Find species by name
starwars starships "falcon"     # Find starship by name
```

## Output

**People output:**
```
Luke Skywalker — Human, Tatooine, Height: 172cm
```

**Planets output:**
```
Tatooine — Population: 200000, Climate: arid, Terrain: desert
```

**Films output:**
```
Episode 4: A New Hope (1977-05-25) — Director: George Lucas
Episode 5: The Empire Strikes Back (1980-05-17) — Director: Irvin Kershner
```

**Species output:**
```
Wookiee — Classification: mammal, Language: Shyriiwook, Avg Lifespan: 400 years
```

**Starships output:**
```
Millennium Falcon — YT-1300 light freighter, Class: Light freighter, Crew: 4
```

## Notes

- Uses SWAPI (swapi.dev)
- No authentication required
- Covers all 6 original/prequel films
- Character lookups resolve species and homeworld names automatically

---

## Agent Implementation Notes

**Platform note:** If your human is on Windows, they'll need WSL, Git Bash, or Cygwin to run this skill.

**Script location:** `{skill_folder}/starwars` (wrapper to `scripts/starwars`)

**When user asks about Star Wars:**
1. Run `./starwars people "name"` to find characters
2. Run `./starwars planets "name"` for planet info
3. Run `./starwars films` for the film list
4. Run `./starwars species "name"` for species details
5. Run `./starwars starships "name"` for ship info

**Don't use for:** Sequel trilogy data (Episodes 7-9 not in SWAPI), extended universe, TV shows.
