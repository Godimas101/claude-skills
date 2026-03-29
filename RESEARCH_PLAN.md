# Space Engineers Skill — Research Plan

Working notes for ongoing skill improvement. Tracks what's been researched, what's pending, and what to do with results.

---

## Status Legend
- ✅ Done — incorporated into skill files
- 🔄 In progress — agent running or results waiting to be processed
- ⏳ Queued — planned, not started yet
- ❌ Failed — hit rate limit or errored, needs retry

---

## Completed Research

### Reference Pages (spaceengineers.wiki.gg/wiki/Modding/Reference/*)
- ✅ `Modding/Reference/SBC` — universal SBC rules, DefinitionBase fields, override vs additive behavior → **SBC_TEMPLATES.md**
- ✅ `Modding/Reference/ModScripting` — script folder structure, namespace ambiguity, profiler injection, Save/Sync patterns → **CSHARP_PATTERNS.md**
- ✅ `Modding/Reference/Materials` — _cm/_ng/_add/_alphamask channels, technique values, transparent materials, texconv commands → **SKILL.md**
- ✅ `Modding/Reference/Old_SBM_and_BIN_Files_Explained` — .sbm files safe to delete, .sbm_ folders in Storage are mod data (do NOT delete) → **SKILL.md**
- ✅ `Modding/Reference/Overview_of_Modding-Relevant_Changes_in_Game_Patches` — all 9 sub-pages (1.200–1.208) → **PATCH_NOTES.md**

---

## In Progress

### Beginner / Getting Started Pages
- 🔄 `Modding/Introduction_to_Modding_Space_Engineers` — mod type overview for beginners
- 🔄 `Modding/Tutorials/Setting_Up_a_Modding_Environment` — VS Code setup, project config, DLL references, whitelist analyzer
- 🔄 `Modding/Tutorials/Creating_And_Uploading_Mods` — metadata.mod, modinfo.sbmi, Workshop publishing steps
- 🔄 `Modding/Tutorials/Modifying_Mods_by_Other_Creators` — cross-mod asset referencing (high priority)
- 🔄 `Modding/Troubleshooting` — error messages, log reading, common failures

**Target output:** `GETTING_STARTED.md` (new file) + additions to SKILL.md log-reading section

---

## Queued — Tutorial Pages (spaceengineers.wiki.gg/wiki/Modding/Tutorials/*)

These all hit the rate limit. Relaunch after 1pm in focused batches.

### Batch A — SBC Tutorials
- ⏳ All pages under `Modding/Tutorials/SBC/`
  - Block definitions (CubeBlocks)
  - Blueprints / BlueprintClasses
  - Components, items, ammo
  - Planet/environment definitions
  - Audio SBC
  - Localization
  - LCD/screen SBC definitions
  - Block categories / variant groups

**Target output:** Expand **SBC_TEMPLATES.md** with real worked examples

### Batch B — C# / Scripting Tutorials
- ⏳ All pages under `Modding/Tutorials/` that cover C#
  - Session component examples
  - Text Surface Script / LCD app mod script
  - Game logic components
  - Networking / sync patterns
  - Debugging and logging
  - Visual Scripting Tool (lower priority)

**Target output:** Expand **CSHARP_PATTERNS.md** with real worked examples

### Batch C — Asset Pipeline Tutorials
- ⏳ All pages under `Modding/Tutorials/` that cover assets
  - 3D models — FBX → MWM pipeline, LODs, build stages
  - Textures — detailed DDS workflow
  - Voxel textures
  - Audio / XWM
  - Animations
  - LCD textures

**Target output:** New **ASSETS.md** file

### Batch D — Recipes / Workflow Tutorials
- ⏳ All pages under `Modding/Tutorials/Recipes/`
  - LCD App Mod Script recipe
  - Any other end-to-end workflow guides

**Target output:** Incorporate into relevant files or a RECIPES.md

---

## Planned New Skill Files

| File | Status | Contents |
|------|--------|----------|
| `GETTING_STARTED.md` | ⏳ Waiting on in-progress agents | Beginner onboarding: mod types, VS Code setup, uploading, cross-mod assets, troubleshooting |
| `ASSETS.md` | ⏳ Waiting on Batch C | Full asset pipeline: models, textures, audio, animations |
| `TUTORIALS.md` | ⏳ Waiting on all batches | Index of all tutorials with one-line summaries and links |

---

## Notes

- **Rate limit:** Hits at ~35-40 tool uses per agent session. Wide wiki crawls always fail. Keep batches focused — max ~10 pages per agent.
- **Cross-mod asset referencing** (`Modifying_Mods_by_Other_Creators`) is high-value content — make sure it gets into GETTING_STARTED.md prominently.
- **VS Code preference:** When writing GETTING_STARTED.md, steer users toward VS Code (free, works with Claude Code skill) over Visual Studio.
- **GETTING_STARTED.md purpose:** Beginner onboarding only. Not a reference doc. Should answer "where do I start?" not "what does field X do?"
