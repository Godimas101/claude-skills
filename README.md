# ⚙️ Space Engineers Skill for Claude Code

> **"Your AI co-engineer — doesn't need oxygen, never rage quits."**

An expert Claude Code skill covering all types of Space Engineers mod development, from SBC XML patches to full C# compiled mods.

---

## 🚀 What It Covers

- **Compiled mods** — C# text surface scripts (LCD screens), session components, SBC XML definitions
- **MES encounter mods** — NPC ship, drone, station, and creature encounters using Modular Encounters System
- **AI Enabled mods** — Humanoid NPCs, robots, and creature bots using the AI Enabled framework
- **Framework mods** — Child mods for community frameworks: WeaponCore, Vanilla+, Animation Engine, Scope Framework, and Tank Tracks
- **Mod Adjuster mods** — Runtime definition patching via the [Mod Adjuster](https://steamcommunity.com/workshop/filedetails/?id=3017795356) framework
- **Programmable Block scripts** — Sandboxed ingame scripts (Main loop, GridTerminalSystem, IGC, etc.)

---

## 📦 Install

1. Copy the `space-engineers/` folder into your Claude Code skills directory:

   **Windows:**
   ```
   C:\Users\[YourName]\.claude\skills\space-engineers\
   ```

   **Linux/Mac:**
   ```
   ~/.claude/skills/space-engineers/
   ```

2. That's it. Use `/space-engineers` in any Claude Code session.

---

## 🗂️ Recommended Workspace Setup

The skill works out of the box, but it's **significantly more useful** when Claude can browse your actual game files. When you invoke `/space-engineers`, Claude will check whether these directories are available and ask you to add any that are missing.

Add these as **additional working directories** in your VS Code workspace (or Claude Code settings):

| Directory | Why |
|-----------|-----|
| `[Steam]\steamapps\common\SpaceEngineers\` | Vanilla SBC files — ground truth for all block/item definitions. Required for looking up block properties, component costs, and balance values. |
| `[Steam]\steamapps\common\SpaceEngineersModSDK\` | API DLLs with XML documentation. Required for compiled mod and PB script work — intellisense, method signatures, interface definitions. |
| Your mod project folder | Your actual mod source files. Required for editing your mod. |
| `[Steam]\steamapps\workshop\content\244850\` | Subscribed Workshop mods. Required for the mod catalogue — used to look up mod names, Workshop IDs, and SBC definitions when building patches or avoiding naming collisions. |
| `%AppData%\SpaceEngineers\` | Game logs and local mod files. Needed for debugging crashes and mod load errors. |

> **Default Steam path on Windows:** `C:\Program Files (x86)\Steam\` or `D:\SteamLibrary\` depending on your install.
>
> **Default AppData path on Windows:** `C:\Users\[YourName]\AppData\Roaming\SpaceEngineers\`

**ModSDK includes these tools out of the box (no extra downloads needed):**
- `Tools\xWMAEncode.exe` — WAV → XWM audio conversion for sound mods
- `Tools\AdpcmEncode.exe` — WAV → ADPCM audio conversion
- `Tools\VRageEditor\` — ModelViewer, ModelBuilder (FBX→MWM), AnimationController, VisualScripting

---

## 🤖 What Claude Does Automatically

When you invoke `/space-engineers`, Claude will:

- **Check for required directories** — if the game directory, ModSDK, workshop folder, or AppData aren't in your workspace, Claude will ask you to add any that are missing before proceeding
- **Detect new patches** — compares installed DLC against a known catalogue; if a new patch has dropped, Claude will alert you and offer to research the new content before you start work
- **Ask what you're working on** — a quick picker to select your project type (Mod, MES/AI Enabled, Framework mod, Mod Adjuster, PB Script, or Torch/Pulsar plugin) so Claude focuses on the right tools and patterns
- **Read your mod notes** — if a `MOD_MAKING_NOTES.md` exists in your mod directory, Claude reads it first to catch up on previous sessions and what's still pending. If one doesn't exist, Claude will offer to create it.
- **Check your mod catalogue** — if a `MOD_CATALOGUE.md` exists in your workshop directory, Claude uses it to look up mod names, Workshop IDs, and SBC definitions

---

## 🗃️ What's Inside

| File | Contents |
|------|---------|
| `SKILL.md` | Main skill — all mod types, SBC XML, C# patterns, asset pipeline, log reading, gotchas |
| `CSHARP_PATTERNS.md` | Extended C# reference — power/gas/inventory queries, drawing helpers, config patterns, performance rules |
| `SBC_TEMPLATES.md` | Copy-paste XML templates for common SBC patterns |
| `MES.md` | Modular Encounters System guide — SpawnGroup, Behavior, Autopilot, Trigger, Action, SpawnConditions formats and examples |
| `AI_ENABLED.md` | AI Enabled guide — bot definitions, character SBC, MES integration for creature/NPC spawning |
| `DLC_CATALOGUE.md` | Full DLC pack listing with SubtypeIds — used at startup to detect new patches |
| `MOD_ADJUSTER.md` | Full Mod Adjuster guide — file structure, XML format, all definition types, patch examples |
| `PB_SCRIPTS.md` | Full PB scripting guide — Main loop, UpdateFrequency, block interfaces, coroutines, IGC, sandbox restrictions |
| `TORCH.md` | Torch dedicated server framework — installation, plugin development, manifest format, NexusV3 multi-server |
| `PULSAR.md` | Pulsar client plugin loader — installation, plugin development, PluginHub publishing, HarmonyLib patching |
| `WEAPONCORE.md` | WeaponCore (CoreSystems) framework — CoreParts C# definitions, weapon + ammo structure, PB API |
| `VANILLA_PLUS.md` | Vanilla+ Framework — VPFAmmoDefinition, VPFTurretDefinition, session component pattern |
| `ANIMATION_ENGINE.md` | Animation Engine (Math0424) — BSL scripting language, main.info, subpart/emitter methods |
| `SCOPE_FRAMEWORK.md` | Scope Framework — ScopeConfig.txt INI format, camera block SBC requirements, no C# required |
| `TANK_TRACKS.md` | Tank Tracks (Digi) — TankTracks.ini format, segment model setup, C# API for scripted tools |

---

## 💡 Usage Examples

```
/space-engineers
> I want to add scrolling to the battery list on my status LCD screen
```

```
/space-engineers
> Help me write a Mod Adjuster patch to increase the power output of all solar panels by 50%
```

```
/space-engineers
> Write a PB script that monitors battery charge and broadcasts a warning via IGC when below 20%
```

```
/space-engineers
> I want to create a MES encounter mod that spawns pirate drones on EarthLike at night
```

```
/space-engineers
> Help me set up an AI Enabled bot that patrols an NPC station and attacks players on sight
```

```
/space-engineers
> My mod isn't loading — here's the game log, can you find the error?
```

```
/space-engineers
> I want to make a WeaponCore child mod — help me write the Part and Ammo files for a new railgun turret
```

```
/space-engineers
> Help me add a scope view to my sniper rifle block using SteadyScope
```

```
/space-engineers
> I want to add a particle effect animation to my custom welder block using Animation Engine
```

---

## 🔩 Requirements

- [Claude Code](https://www.anthropic.com/claude-code)
- Space Engineers installed via Steam
- Space Engineers ModSDK installed (free via Steam Tools)
- For Mod Adjuster mods: [Mod Adjuster](https://steamcommunity.com/workshop/filedetails/?id=3017795356) subscribed in Workshop
- For framework mods: the corresponding framework subscribed (WeaponCore `3154371364`, Vanilla+ `2915780227`, Animation Engine `2880317963`, Scope Framework `2754014019`, Tank Tracks `3208995513`)

---

## 🙏 Credits

Built by **Godimas** and **Claude**.

Mod Adjuster XML format reverse-engineered from the [Mod Adjuster](https://steamcommunity.com/workshop/filedetails/?id=3017795356) mod source (Workshop ID: 3017795356).

---

*Please weld responsibly.*
