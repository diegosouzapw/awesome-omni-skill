---
name: dayz-dev
description: DayZ Enforce Script development orchestrator. Dynamically fetches class APIs, script references, and mod documentation. Supports vanilla, Community Framework, and Expansion development for DayZ 1.28+.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch
---

# DayZ Development

> **Dynamic documentation orchestrator** for DayZ mod development.
> Supports vanilla Enforce Script, Community Framework (CF), and DayZ Expansion.
> Target version: **DayZ 1.28+ (v1.28.161464)**

## Philosophy

1. **Fetch, don't memorize** - Always get latest from authoritative sources
2. **Framework-aware thinking** - Detect vanilla vs CF vs Expansion, adapt patterns
3. **Enforce Script correctness** - DayZ uses Enforce Script (C-like), NOT C#/C++/Lua
4. **Server-side validation** - Never trust client-side data
5. **Null-safe always** - Every Cast<>, GetInventory(), GetIdentity() must be null-checked

---

## CRITICAL: No Hallucination Policy

**NEVER invent or guess Enforce Script classes, methods, config tokens, or parameters.**

### Rules:
1. **If unsure about a class/method** -> MUST fetch from DayZ Scripts API or Script Diff repo
2. **If unsure about config.cpp tokens** -> MUST fetch from BI wiki or DayZ Central Economy repo
3. **If a class doesn't exist** -> Tell user honestly, suggest alternatives
4. **If parameters unknown** -> Fetch documentation, don't guess
5. **NEVER use C#/C++ syntax** -> Enforce Script looks like C but has key differences

### Before writing any class or method call:
- [ ] Is this a real DayZ class? -> Verify at dayz-scripts.yadz.app or DayZ-Script-Diff
- [ ] Is this the correct method signature? -> Check parameter types and order
- [ ] Does this work on server/client/both? -> Check script module (3_Game/4_World/5_Mission)
- [ ] Am I null-checking accessors? -> Cast<>, GetInventory(), GetIdentity(), GetPlayer()

### When you don't know:
```
"I'm not 100% certain about this class/method. Let me fetch the documentation..."
[Use WebFetch to get accurate info]
```

### Verification Sources:
| Type | Source | Action |
|------|--------|--------|
| Script API (v1.28) | https://dayz-scripts.yadz.app/ | WebFetch for class/method docs |
| Script Diff (official) | https://github.com/BohemiaInteractive/DayZ-Script-Diff | Check exact source code |
| Enforce Syntax | https://community.bistudio.com/wiki/DayZ:Enforce_Script_Syntax | Language reference |
| Config tokens | https://community.bistudio.com/wiki/CfgVehicles_Config_Reference | Config.cpp reference |
| Central Economy | https://github.com/BohemiaInteractive/DayZ-Central-Economy | types.xml, events.xml |
| CF docs | https://github.com/Arkensor/DayZ-CommunityFramework | CF source + docs |
| Expansion wiki | https://github.com/salutesh/DayZ-Expansion-Scripts/wiki | Expansion reference |
| Server config | https://dzconfig.com/wiki/ | Server XML/JSON configs |
| DeepWiki Expansion | https://deepwiki.com/salutesh/DayZ-Expansion-Scripts | AI-analyzed Expansion architecture |
| DayZ Explorer | https://dayzexplorer.zeroy.com/ | Enforce essentials, Math, FileIO, Widget API |

### Example - WRONG:
```csharp
// DON'T: Using C# syntax or inventing methods
player.GetComponent<Inventory>().AddItem("AK74");  // NOT Enforce Script!
```

### Example - RIGHT:
```c
// DO: Use verified Enforce Script with null checks
PlayerBase player = PlayerBase.Cast(GetGame().GetPlayer());
if (player)
{
    EntityAI item = player.GetInventory().CreateInInventory("AKM");
    if (item)
    {
        // item created successfully
    }
}
```

---

## Content Map

**Read ONLY relevant files based on the request:**

| File | Description | When to Read |
|------|-------------|--------------|
| `scripting/enforce-script.md` | Enforce Script language quick reference | Writing any code |
| `scripting/class-hierarchy.md` | Class tree and key singletons | Looking up classes |
| `scripting/client-server.md` | Script module architecture | New mod, client/server questions |
| `scripting/memory-management.md` | ref, autoptr, Managed patterns | Memory/lifecycle issues |
| `systems/mod-structure.md` | Mod folders, config.cpp, meta.cpp | Creating new mods |
| `systems/networking.md` | RPC, NetSync, CF NetworkedVariables | Multiplayer sync |
| `systems/inventory.md` | Inventory system, InventoryLocation | Item manipulation |
| `systems/actions.md` | Action system hierarchy | Custom actions |
| `systems/weapons.md` | Weapon FSM, configs | Weapon mods |
| `systems/vehicles.md` | Vehicle config, SimulationModule | Vehicle mods |
| `frameworks/framework-detection.md` | Detect vanilla vs CF vs Expansion | Starting new task |
| `frameworks/community-framework.md` | CF modules, RPC, NetworkedVariables | Using CF |
| `frameworks/expansion.md` | Expansion systems overview | Using Expansion |
| `config/config-cpp.md` | config.cpp reference and patterns | Item/vehicle config |
| `config/types-xml.md` | types.xml, economy system | Loot spawning |
| `config/server-config.md` | Server configuration files | Server setup |
| `compatibility/version-128.md` | 1.28 breaking changes and new features | Version questions, migration |

---

## Dynamic Fetching - Decision Tree

### Step 1: Classify the Request

| If user asks about... | Action |
|-----------------------|--------|
| Enforce Script class/method (EntityAI, PlayerBase, etc.) | **FETCH from DayZ Scripts API** |
| Config.cpp tokens (CfgVehicles, CfgWeapons) | **FETCH from BI Wiki** |
| Central Economy (types.xml, events.xml) | **FETCH from DayZ-Central-Economy repo** |
| CF feature (RPCManager, Modules, NetworkedVariables) | **FETCH from CF GitHub** |
| Expansion system (Market, Quests, AI, Basebuilding) | **FETCH from Expansion wiki** |
| Script diff between versions | **FETCH from DayZ-Script-Diff repo** |
| Server configuration | **FETCH from DZconfig wiki** |
| Mod structure, best practices | **READ local files** |
| 1.28 compatibility/changes | **READ local compatibility file** |

### Step 2: WebFetch URLs

#### Script API Reference (v1.28)
**Base URL:** `https://dayz-scripts.yadz.app/`

```
WebFetch(
  url: "https://dayz-scripts.yadz.app/",
  prompt: "Find documentation for the class or method '{CLASS_OR_METHOD}'.
           Include: inheritance, methods, parameters, return types."
)
```

**Key API Pages:**
| Category | URL |
|----------|-----|
| Enforce Essentials | https://dayz-scripts.yadz.app/d5/d78/group___enforce |
| Math Library | https://dayz-scripts.yadz.app/d5/d98/group___math |
| String Methods | https://dayz-scripts.yadz.app/d5/da2/group___strings |
| Widget UI System | https://dayz-scripts.yadz.app/d9/d0e/group___widget_a_p_i |
| Math Class | https://dayz-scripts.yadz.app/d4/d34/class_math |

#### Alternate API Reference (older but comprehensive)
**Base URL:** `https://dayzexplorer.zeroy.com/`

| Page | URL |
|------|-----|
| Enforce Core | https://dayzexplorer.zeroy.com/group___enforce.html |
| Math Functions | https://dayzexplorer.zeroy.com/group___math.html |
| FileIO API | https://dayzexplorer.zeroy.com/group___file.html |
| Particle Effects | https://dayzexplorer.zeroy.com/group___particle_effect.html |
| Widget API | https://dayzexplorer.zeroy.com/group___widget_a_p_i.html |
| DiagMenu | https://dayzexplorer.zeroy.com/group___diag_menu.html |
| Weather Class | https://dayzexplorer.zeroy.com/class_weather.html |
| Vector Class | https://dayzexplorer.zeroy.com/classvector.html |

#### Official Script Source (for exact implementations)
```
WebFetch(
  url: "https://github.com/BohemiaInteractive/DayZ-Script-Diff/tree/main/scripts",
  prompt: "Find the source code for '{CLASS_NAME}' in the DayZ script tree.
           Show the class definition, methods, and inheritance."
)
```

#### Config References
```
WebFetch(
  url: "https://community.bistudio.com/wiki/CfgVehicles_Config_Reference",
  prompt: "Find the config token '{TOKEN_NAME}' and its usage.
           Include: type, default value, parent class, example."
)
```

#### Central Economy
```
WebFetch(
  url: "https://github.com/BohemiaInteractive/DayZ-Central-Economy",
  prompt: "Find the economy configuration for '{ITEM_OR_SETTING}'.
           Include: types.xml entry, spawn parameters, nominal/min values."
)
```

#### Community Framework
```
WebFetch(
  url: "https://github.com/Arkensor/DayZ-CommunityFramework/tree/production/docs",
  prompt: "Find documentation for CF '{FEATURE}'.
           Include: API, usage examples, required setup."
)
```

#### Expansion Scripts
```
WebFetch(
  url: "https://github.com/salutesh/DayZ-Expansion-Scripts/wiki",
  prompt: "Find documentation for Expansion '{SYSTEM}'.
           Include: settings, configuration, scripting API."
)
```

#### Server Configuration
```
WebFetch(
  url: "https://dzconfig.com/wiki/",
  prompt: "Find documentation for '{CONFIG_FILE}'.
           Include: all parameters, types, default values, examples."
)
```

---

## Request Router - Pattern Matching

### RULE 1: Enforce Script Class/Method Detection
**Triggers when:**
- Class names (PascalCase like `EntityAI`, `PlayerBase`, `ItemBase`, `CarScript`)
- Method calls (`GetInventory()`, `CreateInInventory()`, `SetHealth()`)
- "enforce script", "dayz class", "dayz method", "script API"

**Action:** Fetch from `https://dayz-scripts.yadz.app/` or `DayZ-Script-Diff`

### RULE 2: Config.cpp / CfgVehicles Detection
**Triggers when:**
- `CfgVehicles`, `CfgWeapons`, `CfgMagazines`, `CfgAmmo`
- `CfgPatches`, `CfgMods`, `DamageSystem`
- "config.cpp", "model config", "item config", "vehicle config"
- `scope`, `displayName`, `model`, `hiddenSelections`

**Action:** Read local `config/config-cpp.md` + Fetch from BI wiki if needed

### RULE 3: Central Economy Detection
**Triggers when:**
- `types.xml`, `events.xml`, `cfgspawnabletypes.xml`, `cfgeconomycore.xml`
- "loot spawn", "item spawn", "economy", "nominal", "min", "restock"
- `randompresets.xml`, `cfgenvironment.xml`

**Action:** Read local `config/types-xml.md` + Fetch from DayZ-Central-Economy repo

### RULE 4: CF / Expansion Framework Detection
**Triggers when:**
- `RPCManager`, `CF_ModuleWorld`, `NetworkedVariables`, `ModStorage`
- `ExpansionMarket`, `ExpansionQuest`, `ExpansionAI`, `ExpansionTerritory`
- "community framework", "CF module", "expansion", "market system"

**Action:** Detect framework -> Fetch from appropriate docs

### RULE 5: Networking / RPC Detection
**Triggers when:**
- `ScriptRPC`, `GetRPCManager()`, `SendRPC`, `AddRPC`
- `RegisterNetSyncVariable`, `SetSynchDirty`, `OnVariablesSynchronized`
- "sync variable", "RPC", "network", "client-server communication"

**Action:** Read local `systems/networking.md` + Fetch if needed

### RULE 6: Server Configuration Detection
**Triggers when:**
- `serverDZ.cfg`, `cfgGameplay.json`, `storage_1`
- "server config", "gameplay settings", "admin", "BattlEye"

**Action:** Read local `config/server-config.md` + Fetch from DZconfig wiki

### RULE 7: 1.28 Compatibility
**Triggers when:**
- "1.28", "update", "breaking change", "migration", "compatibility"
- `sealed`, `Obsolete`, `Contact`, `SurfaceProperties`
- "vehicle brake", "useNewNetworking", "parameter limit"

**Action:** Read local `compatibility/version-128.md`

### RULE 8: Local Knowledge
**Triggers when:**
- Mod structure, folder layout, PBO packaging
- Best practices, anti-patterns, common pitfalls
- Memory management, lifecycle patterns
- Action system, weapon system, inventory system

**Action:** Read relevant local markdown file

---

## Mod Framework Auto-Detection

When starting a task, detect what frameworks the mod uses:

### Check config.cpp dependencies
```cpp
// Vanilla only
requiredAddons[] = {"DZ_Data"};

// Community Framework
requiredAddons[] = {"DZ_Data", "JM_CF_Scripts"};

// Expansion
requiredAddons[] = {"DZ_Data", "JM_CF_Scripts", "DayZExpansion_Core"};
```

### Check script imports
```c
// CF detection
GetRPCManager()     // Uses CF RPC system
CF_ModuleWorld      // Uses CF Module system

// Expansion detection
ExpansionMarketModule       // Uses Expansion Market
ExpansionQuestModule        // Uses Expansion Quests
eAIBase                     // Uses Expansion AI
```

### Check mod.cpp / meta.cpp
```
// CF dependency
dependency[] = {"Community Framework"};

// Expansion dependency
dependency[] = {"DayZ Expansion Core", "DayZ Expansion Scripts"};
```

---

## Best Practices (Quick Reference)

### Enforce Script Rules
| Rule | Why |
|------|-----|
| Use `!GetGame().IsDedicatedServer()` for client check | `IsClient()` returns FALSE during init |
| Assign getter results to local var before foreach | `foreach` on inline getter returns fails |
| Always parenthesize bitwise ops: `(a & b) == b` | Bitwise ops have lower precedence than comparisons |
| Use `ref` ONLY for member variables | `ref` in function params is WRONG |
| Check last file before reported error location | Compiler errors often point to wrong file |
| Never leave empty preprocessor blocks | Can cause segfaults |

### Memory Management Rules
| Rule | Why |
|------|-----|
| `ref` for member variables only | Controls reference lifetime |
| Never `delete` manually | Enforce Script uses GC |
| Inherit from `Managed` for ref counting | Enables `ref`/`autoptr` usage |
| `autoptr` auto-deletes when scope exits | Use for temporary owned references |

### Null Safety Rules
| Rule | Example |
|------|---------|
| Always check `Cast<>` results | `PlayerBase p = PlayerBase.Cast(entity); if (p) {...}` |
| Always check `GetInventory()` | `if (player.GetInventory()) {...}` |
| Always check `GetIdentity()` | `if (player.GetIdentity()) {...}` |
| Always check `GetGame().GetPlayer()` | Can be null during init/cleanup |

### Security Rules
| Rule | Reason |
|------|--------|
| Validate on server side | Client can be tampered |
| Check `GetGame().IsServer()` before gameplay logic | Prevent client-side exploitation |
| Use RPC callbacks, not direct events | Prevent event spoofing |
| Validate player identity on server RPCs | Prevent impersonation |

### Mod Structure
```
MyMod/
├── mod.cpp                    # Mod metadata
├── meta.cpp                   # Workshop metadata (auto-generated)
├── Keys/                      # BIS key for server signing
├── Addons/
│   └── MyMod/
│       ├── config.cpp         # CfgPatches, CfgMods, CfgVehicles
│       ├── $PREFIX$            # Mod prefix file
│       └── scripts/
│           ├── config.cpp     # Script module registration
│           ├── 3_Game/        # Game-level scripts (available everywhere)
│           ├── 4_World/       # World-level scripts (entities, items, players)
│           └── 5_Mission/     # Mission-level scripts (HUD, menus, mission logic)
```

---

## Anti-Patterns

| Don't | Do |
|-------|-----|
| `GetGame().IsClient()` during init | `!GetGame().IsDedicatedServer()` |
| `foreach (auto x : GetSomething())` | `auto list = GetSomething(); foreach (auto x : list)` |
| `if (flags & FLAG == FLAG)` | `if ((flags & FLAG) == FLAG)` |
| `ref` in function parameters/returns/locals | `ref`/`autoptr` only for class member variables |
| Add `: ParentClass` to `modded class` | `modded class` already inherits - never add inheritance |
| `delete obj;` | `obj = null;` (let GC handle cleanup) |
| Trust client data in RPCs | Always validate server-side |
| `GetGame()` in hot paths | Use `g_Game` global (1.28+ optimization) |
| `SurfaceIsPond()` / `SurfaceIsSea()` | `g_Game.GetWaterDepth(pos) <= 0` (much faster) |
| `GetObjectsAtPosition()` frequently | Use static arrays, triggers, or GetScene() |
| Empty `#ifdef` / `#endif` blocks | Always have content or remove entirely |
| Hardcode framework dependencies | Detect at runtime via config.cpp |
| Skip null checks on Cast<> | Always check before using result |
| Write files outside `$saves:`/`$profile:` | FileIO only works in those directories |
| Unqualified member names in modded classes | Prefix with mod name: `m_MyMod_VarName` |

---

## DayZ-Specific Patterns

### Modded Class Injection (The DayZ Way)
```c
modded class PlayerBase
{
    override void SetActions(out TInputActionMap InputActionMap)
    {
        super.SetActions(InputActionMap);
        AddAction(MyCustomAction, InputActionMap);
    }
}
```

### RPC Communication (Vanilla)
```c
// Server -> Client
ScriptRPC rpc = new ScriptRPC();
rpc.Write(someData);
rpc.Send(player, ERPCs.RPC_USER_ACTION_MESSAGE, true, player.GetIdentity());

// Client handler
void OnRPC(PlayerIdentity sender, Object target, int rpc_type, ParamsReadContext ctx)
{
    if (rpc_type == ERPCs.RPC_USER_ACTION_MESSAGE)
    {
        ctx.Read(someData);
    }
}
```

### RPC Communication (CF)
```c
// Register
GetRPCManager().AddRPC("MyMod", "MyRPCHandler", this, SingeplayerExecutionType.Both);

// Send
GetRPCManager().SendRPC("MyMod", "MyRPCHandler", new Param1<string>("data"), true, null);

// Handler
void MyRPCHandler(CallType type, ParamsReadContext ctx, PlayerIdentity sender, Object target)
{
    Param1<string> data;
    if (!ctx.Read(data)) return;
    // process data.param1
}
```

### Player Lifecycle Events
```
OnInit -> InvokeOnConnect -> OnClientReadyEvent -> OnClientDisconnectedEvent -> MissionFinish
```

### Net Sync Variables
```c
class MyEntity extends ItemBase
{
    int m_MyValue;

    void MyEntity()
    {
        RegisterNetSyncVariableInt("m_MyValue", 0, 100);
    }

    void SetMyValue(int val)
    {
        m_MyValue = val;
        SetSynchDirty();
    }

    override void OnVariablesSynchronized()
    {
        super.OnVariablesSynchronized();
        // m_MyValue is now updated on client
    }
}
```

---

## Related Skills

| Need | Skill |
|------|-------|
| UI/HUD design | Read `scripting/` files for Widget system |
| Server administration | Read `config/server-config.md` |
| Expansion modding | Read `frameworks/expansion.md` |
| Version migration | Read `compatibility/version-128.md` |
