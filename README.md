# Space Engineers Modding Skill for Claude Code

An expert skill for Claude Code covering all three types of Space Engineers mod development:

- **Compiled mods** — C# text surface scripts (LCD screens), session components, SBC XML definitions
- **Mod Adjuster mods** — Runtime definition patching via the [Mod Adjuster](https://steamcommunity.com/workshop/filedetails/?id=3017795356) framework
- **Programmable Block scripts** — Sandboxed ingame scripts (Main loop, GridTerminalSystem, IGC, etc.)

---

## Install

1. Copy the `space-engineers/` folder into your Claude Code skills directory:

   **Windows:**
   ```
   C:\Users\[YourName]\.claude\skills\space-engineers\
   ```

   **Linux/Mac:**
   ```
   ~/.claude/skills/space-engineers/
   ```

2. That's it. The skill is available as `/space-engineers` in any Claude Code session.

---

## Recommended VS Code Workspace Setup

The skill is significantly more useful when Claude can browse your actual game files directly. Add these as **additional working directories** in your VS Code workspace (or Claude Code settings):

| Directory | Why |
|-----------|-----|
| `[Steam]\steamapps\common\SpaceEngineers\` | Vanilla SBC files — the ground truth for all 107 block/item definition files |
| `[Steam]\steamapps\common\SpaceEngineersModSDK\` | API DLLs with 25+ XML documentation files + modding tools |
| `[Steam]\steamapps\workshop\content\244850\` | Your subscribed mods — useful for cross-referencing other mods |
| Your mod project folder | Your actual mod source files |

**ModSDK includes these tools out of the box (no extra downloads needed):**
- `Tools\xWMAEncode.exe` — WAV → XWM audio conversion for sound mods
- `Tools\AdpcmEncode.exe` — WAV → ADPCM audio conversion
- `Tools\VRageEditor\` — ModelViewer, ModelBuilder (FBX→MWM), AnimationController, VisualScripting

> **Default Steam path on Windows:** `C:\Program Files (x86)\Steam\` or `D:\SteamLibrary\` depending on your install.

### How to add workspace directories in Claude Code

In your VS Code project's Claude Code settings (`.claude/settings.json`) or via the Claude Code sidebar, add the paths above as additional working directories. This lets Claude browse game files without you having to copy-paste paths into every prompt.

---

## What's Inside

| File | Contents |
|------|---------|
| `SKILL.md` | Main skill — compiled mods, text surface scripts, SBC XML, scrolling patterns, gotchas |
| `CSHARP_PATTERNS.md` | Extended C# reference — power/gas/inventory queries, drawing helpers, config patterns, performance rules |
| `SBC_TEMPLATES.md` | Copy-paste XML templates for common SBC patterns |
| `MOD_ADJUSTER.md` | Full Mod Adjuster guide — file structure, XML format, all definition types, patch examples |
| `PB_SCRIPTS.md` | Full PB scripting guide — Main loop, UpdateFrequency, block interfaces, coroutines, IGC, sandbox restrictions |

---

## Usage Examples

```
/space-engineers
> I want to add scrolling to the battery list on my status LCD screen
```

```
/space-engineers
> Help me write a Mod Adjuster patch to reduce the PCU cost of all armor blocks by 50%
```

```
/space-engineers
> Write a PB script that monitors battery charge and broadcasts a warning via IGC when below 20%
```

---

## Requirements

- [Claude Code](https://www.anthropic.com/claude-code)
- Space Engineers installed via Steam
- Space Engineers ModSDK installed (free via Steam Tools)
- For Mod Adjuster mods: [Mod Adjuster](https://steamcommunity.com/workshop/filedetails/?id=3017795356) subscribed in Steam Workshop

---

## Credits

Built by **Godimas** and **Claude**.

Mod Adjuster XML format reverse-engineered from the [Mod Adjuster](https://steamcommunity.com/workshop/filedetails/?id=3017795356) mod source (Workshop ID: 3017795356).
