# Space Engineers Modding Expert

Expert guidance for all Space Engineers mod and script development.

---

## CRITICAL: Know Which Type You're Working On

Four completely different environments. Get this wrong and nothing works.

| Type | How It Runs | C# Access | Invoked By |
|------|-------------|-----------|------------|
| **Text Surface Script** | Compiled DLL, runs on LCD blocks | Full .NET, all namespaces | InfoLCD mod screens |
| **Session Component** | Compiled DLL, runs as game component | Full .NET, all namespaces | Background game logic |
| **SBC-only Mod** | XML loaded at game start | No C# | Block/item/balance definitions |
| **Mod Adjuster Mod** | Session component + XML patches | Patches live definitions at runtime | Non-destructive cross-mod balance overrides |
| **PB Script** | Sandboxed VM inside the game | Whitelist only — NO I/O, NO threading, NO reflection | Player-written ingame scripts |

> **InfoLCD is a Text Surface Script mod.** Full .NET access. None of the PB sandbox restrictions apply.
>
> For Mod Adjuster details see [MOD_ADJUSTER.md](MOD_ADJUSTER.md).
> For PB scripting details see [PB_SCRIPTS.md](PB_SCRIPTS.md).

---

## Text Surface Script — The Basics

### Class Structure

```csharp
using Sandbox.ModAPI;
using VRage.Game.GUI.TextPanel;
using VRageMath;

[MyTextSurfaceScript("MyScriptId", "Display Name")]
public class MyScreen : MyTextSurfaceScriptBase
{
    IMyTerminalBlock Block => (IMyTerminalBlock)m_block;

    public MyScreen(IMyTextSurface surface, IMyCubeBlock block, Vector2 viewport)
        : base(surface, block, viewport) { }

    public override ScriptUpdate NeedsUpdate => ScriptUpdate.Update10;

    public override void Run()
    {
        // ⚠️ ALWAYS bail on dedicated server
        if (MyAPIGateway.Utilities?.IsDedicated ?? false) return;

        try
        {
            using (var frame = mySurface.DrawFrame())
            {
                // Your drawing code here
            }
        }
        catch { } // Never crash the game
    }

    public override void Dispose() { }
}
```

### The Update10 Rule — CRITICAL

`ScriptUpdate.Update10` means Run() fires every **10 game ticks** (6 times per second).

```csharp
// ❌ WRONG — increments 1 per call, so 1/6th of expected speed
ticksSinceLastScroll++;

// ✅ CORRECT — reflects actual ticks elapsed per call
ticksSinceLastScroll += 10;
```

`scrollSpeed = 60` → ~1 second. `scrollSpeed = 120` → ~2 seconds. Always design timers in ticks (60 = ~1 second at Update10).

### Dedicated Server Guard

```csharp
if (MyAPIGateway.Utilities?.IsDedicated ?? false) return;
```

Use `?.` because `Utilities` can be null during early init.

---

## Drawing on LCD Surfaces

### Sprite Frame Pattern

```csharp
using (MySpriteDrawFrame frame = mySurface.DrawFrame())
{
    frame.Add(new MySprite()
    {
        Type = SpriteType.TEXT,
        Data = "Hello World",
        Position = new Vector2(x, y),
        Color = Color.White,
        FontId = "White",
        Alignment = TextAlignment.LEFT,
        RotationOrScale = textSize
    });
}
```

### Surface Sizing

```csharp
Vector2 screenSize = mySurface.SurfaceSize;
Vector2 viewportOffset = (mySurface.TextureSize - screenSize) / 2f;
float localX = viewportOffset.X + padding;
float localY = viewportOffset.Y + padding;
```

### Position-Based Space Calculation (for scrolling lists)

```csharp
float lineHeight = 30 * surfaceData.textSize;
float currentY = position.Y - surfaceData.viewPortOffsetY;
float remainingHeight = mySurface.SurfaceSize.Y - currentY;
int availableLines = Math.Max(1, (int)(remainingHeight / lineHeight));
```

---

## Accessing Game State

### Grid and Blocks

```csharp
IMyCubeGrid grid = Block.CubeGrid;

var batteries = new List<IMyBatteryBlock>();
grid.GetFatBlocks(batteries);
```

### Inventory Scanning

```csharp
// ⚠️ ALWAYS use composite key — SubtypeId alone is not unique
var typeId = item.Type.TypeId.Split('_')[1];
var subtypeId = item.Type.SubtypeId;
string key = $"{typeId}_{subtypeId}";

// ⚠️ Amounts are VRage.MyFixedPoint, not int/float
int amount = item.Amount.ToIntSafe();
```

### CustomData Config (MyIni Pattern)

```csharp
private MyIni _config = new MyIni();

private void ParseConfig()
{
    if (!_config.TryParse(Block.CustomData)) return;
    _toggleScroll = _config.Get("MyScreen", "EnableScroll").ToBoolean(false);
    _scrollSpeed  = _config.Get("MyScreen", "ScrollSpeed").ToInt32(60);
}
```

---

## Scrolling List Pattern

### Approach 1 — Flat List

```csharp
// State fields
bool toggleScroll = false;
bool reverseDirection = false;
int scrollSpeed = 60;
int scrollLines = 1;
int scrollOffset = 0;
int ticksSinceLastScroll = 0;

// In Run()
if (toggleScroll)
{
    ticksSinceLastScroll += 10;   // ← ALWAYS +10, never ++
    if (ticksSinceLastScroll >= scrollSpeed)
    {
        ticksSinceLastScroll = 0;
        scrollOffset += reverseDirection ? -scrollLines : scrollLines;
    }
}
else { scrollOffset = 0; ticksSinceLastScroll = 0; }

// Draw with wraparound
int total = itemList.Count;
int startIndex = (toggleScroll && total > 0)
    ? ((scrollOffset % total) + total) % total : 0;

int drawn = 0;
for (int i = 0; i < total && drawn < availableLines; i++)
{
    int idx = (startIndex + i) % total;
    // Draw itemList[idx]
    drawn++;
}
```

### Approach 2 — Multi-Category with MaxListLines

```csharp
int maxListLines = 5;  // 0 = unlimited
if (maxListLines > 0)
    availableLines = Math.Min(availableLines, maxListLines);
```

---

## Subgrid Caching Pattern

```csharp
private List<IMyTerminalBlock> _mainBlocks = new List<IMyTerminalBlock>();
private List<IMyTerminalBlock> _subgridCache = new List<IMyTerminalBlock>();
private int _tick = 0;

private void UpdateBlocks()
{
    _mainBlocks.Clear();
    Block.CubeGrid.GetFatBlocks(_mainBlocks);

    _tick += 10;
    if (_tick >= 300)  // ~5 seconds
    {
        _tick = 0;
        _subgridCache.Clear();
        var connectedGrids = new List<IMyCubeGrid>();
        Block.CubeGrid.GetConnectedGrids(GridLinkTypeEnum.Mechanical, connectedGrids);
        foreach (var subgrid in connectedGrids)
        {
            if (subgrid == Block.CubeGrid) continue;
            var sub = new List<IMyTerminalBlock>();
            subgrid.GetFatBlocks(sub);
            _subgridCache.AddRange(sub);
        }
    }
}
```

---

## SBC XML — Quick Reference

### Register a Text Surface Script

```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <LCDScripts>
    <LCDScript>
      <Id Type="MyObjectBuilder_LCDScript" Subtype="MyScriptId" />
      <DisplayName>My Screen Display Name</DisplayName>
      <Description>What this screen shows</Description>
    </LCDScript>
  </LCDScripts>
</Definitions>
```

The `Subtype` must match `[MyTextSurfaceScript("MyScriptId", "...")]` in C#.

### Mod Folder Structure

```
MyMod/
├── Data/
│   ├── Scripts/YourNamespace/YourScript.cs
│   ├── TextSurfaceScripts.sbc
│   └── CubeBlocks/MyBlocks.sbc
├── metadata.mod
└── thumb.png
```

---

## Known Gotchas

1. **Item type collisions** — composite key always: `$"{typeId}_{subtypeId}"`
2. **MyFixedPoint** — use `.ToIntSafe()` or `(float)(double)amount`, never cast directly
3. **Block components are nullable** — always null-check `Components.Get<T>()`
4. **DetailedInfo is fragile** — localized string, format changes with game updates
5. **SBC load order** — last mod loaded wins on same Type+Subtype; use Mod Adjuster for non-destructive patches
6. **Backward compatibility** — never rename CustomData keys; add new ones, keep old ones

---

## Key Reference Files (Local)

| What | Where |
|------|-------|
| Game API DLLs | `D:\SteamLibrary\steamapps\common\SpaceEngineersModSDK\Bin64_Profile\` |
| Vanilla block SBCs | `D:\SteamLibrary\steamapps\common\SpaceEngineers\Content\Data\` |
| InfoLCD source | `c:\Users\Chris Carpenter\VS Code Projects\mods\space-engineers-mods\Mods\` |
| Mod making notes | `c:\Users\Chris Carpenter\VS Code Projects\mods\space-engineers-mods\MOD_MAKING_NOTES.md` |
| Subscribed mods | `D:\SteamLibrary\steamapps\workshop\content\244850\` |
| Mod Adjuster source | `D:\SteamLibrary\steamapps\workshop\content\244850\3017795356\` |

---

## Supporting Reference Files

- [SBC_TEMPLATES.md](SBC_TEMPLATES.md) — Copy-paste XML templates
- [CSHARP_PATTERNS.md](CSHARP_PATTERNS.md) — Extended C# patterns
- [MOD_ADJUSTER.md](MOD_ADJUSTER.md) — Full Mod Adjuster guide
- [PB_SCRIPTS.md](PB_SCRIPTS.md) — Full Programmable Block scripting guide

---

## Quick Checklist Before Shipping

- [ ] Dedicated server guard in every `Run()` method
- [ ] Scroll timer uses `+= 10`, not `++`
- [ ] All inventory keys use composite `typeId_subtypeId`
- [ ] All `Components.Get<T>()` calls are null-checked
- [ ] Both InfoLCD versions updated (Apex Update + Apex Advanced)
- [ ] CustomData keys unchanged (backward compat)
- [ ] Tested on varied LCD sizes including corner panels
