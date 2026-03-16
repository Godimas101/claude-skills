# C# Patterns — Space Engineers Mod Development

Extended reference for C# patterns used in Space Engineers compiled mods (text surface scripts, session components). This is **not** Programmable Block scripting — full .NET access is available.

---

## Key Namespaces

```csharp
using Sandbox.Common.ObjectBuilders;
using Sandbox.Game.EntityComponents;
using Sandbox.ModAPI;
using Sandbox.ModAPI.Ingame;         // For IMyTerminalBlock, IMyBatteryBlock, etc.
using SpaceEngineers.Game.ModAPI;    // For game-specific block types
using VRage.Game;
using VRage.Game.Components;
using VRage.Game.GUI.TextPanel;      // For IMyTextSurface, MySpriteDrawFrame
using VRage.Game.ModAPI;
using VRage.ModAPI;
using VRageMath;
```

---

## Session Component (Background Mod Logic)

When you need logic that runs independently of any specific block:

```csharp
[MySessionComponentDescriptor(MyUpdateOrder.BeforeSimulation)]
public class MySessionComponent : MySessionComponentBase
{
    public static MySessionComponent Instance;

    public override void LoadData()
    {
        Instance = this;
        // Initialize — called when session starts
    }

    public override void UpdateBeforeSimulation()
    {
        // Called every game tick (60fps)
        // Keep this fast — expensive work every N ticks
    }

    protected override void UnloadData()
    {
        Instance = null;
        // Cleanup
    }
}
```

---

## Power System Queries

```csharp
// Get power consumption of a block
var sink = block.Components.Get<MyResourceSinkComponent>();
if (sink != null)
{
    float currentDraw = sink.CurrentInputByType(MyResourceDistributorComponent.ElectricityId);
    float maxDraw = sink.MaxRequiredInputByType(MyResourceDistributorComponent.ElectricityId);
}

// Get power output of a generator/battery
var source = block.Components.Get<MyResourceSourceComponent>();
if (source != null)
{
    float currentOutput = source.CurrentOutputByType(MyResourceDistributorComponent.ElectricityId);
    float maxOutput = source.MaxOutputByType(MyResourceDistributorComponent.ElectricityId);
}

// Battery specific
var battery = block as IMyBatteryBlock;
if (battery != null)
{
    float chargeRatio = battery.CurrentStoredPower / battery.MaxStoredPower;
    bool isCharging = battery.IsCharging;
}

// Quick power grid summary (if you have all blocks)
float totalOutput = 0f, totalInput = 0f, totalStored = 0f, maxStored = 0f;
foreach (var block in powerBlocks)
{
    var bat = block as IMyBatteryBlock;
    if (bat != null)
    {
        totalStored += bat.CurrentStoredPower;
        maxStored += bat.MaxStoredPower;
    }
    // etc.
}
```

---

## Gas System Queries

```csharp
var gasTank = block as IMyGasTank;
if (gasTank != null)
{
    float fillRatio = (float)gasTank.FilledRatio;  // 0.0 to 1.0
    float capacity = gasTank.Capacity;
    float stored = fillRatio * capacity;
    bool isStockpiling = gasTank.Stockpile;
}

// Determine gas type from block definition (hydrogen vs oxygen)
// Check block CustomName or BlockDefinitionId.SubtypeName for "Hydrogen"/"Oxygen"
```

---

## Inventory Queries

```csharp
// Get all items from a cargo container
var cargo = block as IMyCargoContainer;
if (cargo != null)
{
    var inventory = cargo.GetInventory(0);
    var items = new List<VRage.Game.ModAPI.Ingame.MyInventoryItem>();
    inventory.GetItems(items);

    foreach (var item in items)
    {
        // ⚠️ Always use composite key — SubtypeId alone is not unique
        string typeId = item.Type.TypeId.Split('_').Last();  // e.g., "Component"
        string subtypeId = item.Type.SubtypeId;              // e.g., "SteelPlate"
        string key = $"{typeId}_{subtypeId}";

        // ⚠️ Amount is MyFixedPoint — convert before arithmetic
        int amount = item.Amount.ToIntSafe();
        float amountF = (float)(double)item.Amount;  // More precise
    }
}

// Check if inventory can accept item
bool canAdd = inventory.CanItemsBeAdded(100, new VRage.Game.MyDefinitionId(
    typeof(MyObjectBuilder_Component), "SteelPlate"));

// Transfer items between inventories
IMyInventory from = sourceBlock.GetInventory(0);
IMyInventory to = destBlock.GetInventory(0);
from.TransferItemTo(to, 0);  // Transfer item at index 0
```

---

## Production Block Queries

```csharp
var assembler = block as IMyAssembler;
if (assembler != null)
{
    bool isProducing = assembler.IsProducing;
    bool isQueueEmpty = assembler.IsQueueEmpty;

    var queue = new List<VRage.Game.ModAPI.Ingame.MyProductionItem>();
    assembler.GetQueue(queue);
    foreach (var queueItem in queue)
    {
        string name = queueItem.BlueprintId.SubtypeName;
        decimal amount = (decimal)queueItem.Amount;
    }
}

var refinery = block as IMyRefinery;
if (refinery != null)
{
    bool isProducing = refinery.IsProducing;
    // Refinery has input inventory [0] and output inventory [1]
    var inputInv = refinery.GetInventory(0);
    var outputInv = refinery.GetInventory(1);
}
```

---

## Door and Airtight Queries

```csharp
var door = block as IMyDoor;
if (door != null)
{
    var status = door.Status;
    bool isOpen = status == DoorStatus.Open;
    bool isClosed = status == DoorStatus.Closed;
    bool isMoving = status == DoorStatus.Opening || status == DoorStatus.Closing;
}

var hangar = block as IMyAirtightHangarDoor;
// etc. — various door types implement IMyDoor

// Check grid airtightness at a position
bool sealed = block.CubeGrid.IsRoomAtPositionAirtight(block.Position);
```

---

## Text Surface Drawing Helpers

### Draw a Text Line

```csharp
private void DrawText(MySpriteDrawFrame frame, string text, Vector2 pos, float scale, Color color,
    TextAlignment align = TextAlignment.LEFT, string font = "White")
{
    frame.Add(new MySprite
    {
        Type = SpriteType.TEXT,
        Data = text,
        Position = pos,
        Color = color,
        FontId = font,
        Alignment = align,
        RotationOrScale = scale
    });
}
```

### Draw a Progress Bar

```csharp
private void DrawBar(MySpriteDrawFrame frame, Vector2 pos, Vector2 size, float fillRatio,
    Color fillColor, Color bgColor)
{
    // Background
    frame.Add(new MySprite
    {
        Type = SpriteType.TEXTURE,
        Data = "SquareSimple",
        Position = pos,
        Size = size,
        Color = bgColor,
        Alignment = TextAlignment.LEFT
    });

    // Fill
    var fillSize = new Vector2(size.X * Math.Max(0f, Math.Min(1f, fillRatio)), size.Y);
    if (fillSize.X > 0)
    {
        frame.Add(new MySprite
        {
            Type = SpriteType.TEXTURE,
            Data = "SquareSimple",
            Position = pos,
            Size = fillSize,
            Color = fillColor,
            Alignment = TextAlignment.LEFT
        });
    }
}
```

### Clear the Screen (Important for flicker prevention)

```csharp
// Always clear before drawing, or sprites accumulate
using (var frame = mySurface.DrawFrame())
{
    frame.Add(MySprite.CreateClearClipRect());
    // ... draw your sprites
}
```

---

## Config Pattern — Full MyIni Implementation

```csharp
private MyIni _ini = new MyIni();
private const string SECTION = "ScreenName";

// Fields with defaults
private bool _scrollEnabled = false;
private int _scrollSpeed = 60;
private int _maxLines = 0;
private bool _showTitle = true;

private void ReadConfig()
{
    _ini.Clear();
    if (!_ini.TryParse(Block.CustomData))
    {
        // CustomData is invalid INI — don't wipe it, just use defaults
        return;
    }

    _scrollEnabled = _ini.Get(SECTION, "ScrollEnabled").ToBoolean(_scrollEnabled);
    _scrollSpeed = _ini.Get(SECTION, "ScrollSpeed").ToInt32(_scrollSpeed);
    _maxLines = _ini.Get(SECTION, "MaxLines").ToInt32(_maxLines);
    _showTitle = _ini.Get(SECTION, "ShowTitle").ToBoolean(_showTitle);
}

private void WriteConfig()
{
    _ini.Clear();
    _ini.Set(SECTION, "ScrollEnabled", _scrollEnabled);
    _ini.Set(SECTION, "ScrollSpeed", _scrollSpeed);
    _ini.Set(SECTION, "MaxLines", _maxLines);
    _ini.Set(SECTION, "ShowTitle", _showTitle);

    Block.CustomData = _ini.ToString();
}

// For CreateConfig() output (human-readable with comments):
private void AppendConfig(StringBuilder sb)
{
    sb.AppendLine($"; [ {SECTION.ToUpper()} - OPTIONS ]");
    sb.AppendLine($"[{SECTION}]");
    sb.AppendLine("; Enable auto-scrolling of long lists");
    sb.AppendLine($"ScrollEnabled={_scrollEnabled}");
    sb.AppendLine("; Ticks between scroll steps (60 = ~1 second)");
    sb.AppendLine($"ScrollSpeed={_scrollSpeed}");
    sb.AppendLine("; Max visible lines per category (0 = unlimited)");
    sb.AppendLine($"MaxLines={_maxLines}");
    sb.AppendLine("; Show section title headers");
    sb.AppendLine($"ShowTitle={_showTitle}");
}
```

---

## Logging (Debug Output)

```csharp
// Logs to %AppData%\SpaceEngineers\SpaceEngineers.log
MyLog.Default.WriteLine($"[MyMod] Value: {someValue}");
MyLog.Default.Warning($"[MyMod] Warning: {message}");
MyLog.Default.Error($"[MyMod] Error: {ex.Message}");

// Optional: Show in HUD (visible to player, use sparingly)
MyAPIGateway.Utilities.ShowMessage("MyMod", "Debug message");
MyAPIGateway.Utilities.ShowNotification("Temporary HUD message", 2000, "White");
```

---

## Common Type Conversions

```csharp
// MyFixedPoint (inventory amounts)
int asInt = fixedPoint.ToIntSafe();
float asFloat = (float)(double)fixedPoint;
VRage.MyFixedPoint fromInt = (VRage.MyFixedPoint)someInt;

// Vector conversions
Vector3D worldPos = block.GetPosition();
Vector3I gridPos = block.Position;  // Grid coordinate

// Color utilities
Color fromHSV = Color.FromNonPremultiplied(r, g, b, a);  // 0-255
Color dimmed = color * 0.5f;  // 50% brightness

// String → TypeId
var typeId = new MyDefinitionId(typeof(MyObjectBuilder_Component), "SteelPlate");
```

---

## Performance Guidelines

1. **Never allocate in tight loops.** Cache lists and reuse them between frames.
   ```csharp
   // ❌ Allocates every Update10 call
   var blocks = new List<IMyTerminalBlock>();

   // ✅ Allocated once, cleared and refilled each call
   private List<IMyTerminalBlock> _blocks = new List<IMyTerminalBlock>();
   // In Run(): _blocks.Clear(); grid.GetFatBlocks(_blocks);
   ```

2. **Cache expensive queries.** Grid scans, inventory totals, power calculations — don't recompute every frame if data doesn't change that fast.

3. **Minimize LINQ in hot paths.** LINQ is fine for setup/init code, but in `Run()` called 6×/second, prefer for loops.

4. **Subgrid scans are expensive.** Cache subgrid block lists and refresh every 5 seconds (300 ticks), not every frame.

5. **Guard against null everywhere.** Blocks can be removed from the grid between frames. Always null-check before accessing block state.
