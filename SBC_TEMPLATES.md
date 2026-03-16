# SBC Templates — Space Engineers

Copy-paste XML templates for common modding patterns. Always cross-reference against vanilla SBCs at `D:\SteamLibrary\steamapps\common\SpaceEngineers\Content\Data\` for exact schema.

---

## File Header (Required on all SBC files)

```xml
<?xml version="1.0" encoding="utf-8"?>
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <!-- content here -->
</Definitions>
```

---

## Text Surface Script Registration

```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <LCDScripts>
    <LCDScript>
      <Id Type="MyObjectBuilder_LCDScript" Subtype="SCRIPT_SUBTYPE_ID" />
      <DisplayName>Script Display Name</DisplayName>
      <Description>What this screen shows</Description>
    </LCDScript>
  </LCDScripts>
</Definitions>
```

The `Subtype` must exactly match the first argument of `[MyTextSurfaceScript("SCRIPT_SUBTYPE_ID", "...")]` in the C# attribute.

---

## Mod Adjuster Patch (Minimal Override)

Only include the fields you're changing. Leave everything else out — the game merges, not replaces.

### Reactor Power Override
```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CubeBlocks>
    <Definition xsi:type="MyObjectBuilder_ReactorDefinition">
      <Id Type="MyObjectBuilder_Reactor" Subtype="TARGET_SUBTYPE" />
      <MaxPowerOutput>500</MaxPowerOutput>
    </Definition>
  </CubeBlocks>
</Definitions>
```

### Component Cost Override
```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CubeBlocks>
    <Definition xsi:type="MyObjectBuilder_CubeBlockDefinition">
      <Id Type="MyObjectBuilder_CubeBlock" Subtype="TARGET_SUBTYPE" />
      <Components>
        <Component Subtype="SteelPlate" Count="5" />
        <Component Subtype="Construction" Count="3" />
        <Component Subtype="SteelPlate" Count="1" />
      </Components>
      <CriticalComponent Subtype="Construction" Index="0" />
    </Definition>
  </CubeBlocks>
</Definitions>
```

### Refinery Speed / Efficiency Override
```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CubeBlocks>
    <Definition xsi:type="MyObjectBuilder_RefineryDefinition">
      <Id Type="MyObjectBuilder_Refinery" Subtype="TARGET_SUBTYPE" />
      <RefineSpeed>1.0</RefineSpeed>
      <MaterialEfficiency>0.8</MaterialEfficiency>
    </Definition>
  </CubeBlocks>
</Definitions>
```

---

## Full Block Definition (Large Grid, Simple Block)

```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CubeBlocks>
    <Definition xsi:type="MyObjectBuilder_CubeBlockDefinition">
      <Id Type="MyObjectBuilder_CubeBlock" Subtype="MY_BLOCK_ID" />
      <DisplayName>My Block Name</DisplayName>
      <Description>What this block does</Description>
      <Icon>Textures\GUI\Icons\Cubes\MyBlock.dds</Icon>
      <CubeSize>Large</CubeSize>
      <BlockTopology>TriangleMesh</BlockTopology>
      <Size x="1" y="1" z="1" />
      <ModelOffset x="0" y="0" z="0" />
      <Model>Models\Cubes\Large\MyBlock.mwm</Model>
      <Components>
        <Component Subtype="SteelPlate" Count="20" />
        <Component Subtype="Construction" Count="10" />
        <Component Subtype="Motor" Count="2" />
        <!-- Critical component last (or use CriticalComponent tag) -->
        <Component Subtype="SteelPlate" Count="5" />
      </Components>
      <CriticalComponent Subtype="Motor" Index="0" />
      <BuildProgressModels>
        <Model BuildPercentUpperBound="0.33" File="Models\Cubes\Large\MyBlock_BS1.mwm" />
        <Model BuildPercentUpperBound="0.67" File="Models\Cubes\Large\MyBlock_BS2.mwm" />
        <Model BuildPercentUpperBound="1.00" File="Models\Cubes\Large\MyBlock_BS3.mwm" />
      </BuildProgressModels>
      <MountPoints>
        <MountPoint Side="Bottom" StartX="0" StartY="0" EndX="1" EndY="1" />
        <MountPoint Side="Top" StartX="0" StartY="0" EndX="1" EndY="1" />
        <MountPoint Side="Left" StartX="0" StartY="0" EndX="1" EndY="1" />
        <MountPoint Side="Right" StartX="0" StartY="0" EndX="1" EndY="1" />
        <MountPoint Side="Front" StartX="0" StartY="0" EndX="1" EndY="1" />
        <MountPoint Side="Back" StartX="0" StartY="0" EndX="1" EndY="1" />
      </MountPoints>
      <BlockPairName>MY_BLOCK_ID</BlockPairName>
      <EdgeType>Light</EdgeType>
      <PCU>25</PCU>
      <IsAirTight>false</IsAirTight>
    </Definition>
  </CubeBlocks>
</Definitions>
```

---

## Physical Item Definition

```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <PhysicalItems>
    <PhysicalItem xsi:type="MyObjectBuilder_PhysicalItemDefinition">
      <Id Type="MyObjectBuilder_Ore" Subtype="MY_ORE" />
      <DisplayName>My Ore</DisplayName>
      <Description>Description of this ore</Description>
      <Icon>Textures\GUI\Icons\component\MY_ORE.dds</Icon>
      <Size>
        <X>0.07</X><Y>0.07</Y><Z>0.07</Z>
      </Size>
      <Mass>1</Mass>
      <Volume>0.037</Volume>
      <Model>Models\Ores\MY_ORE.mwm</Model>
      <PhysicalMaterial>Rock</PhysicalMaterial>
      <MaxStackAmount>1000000</MaxStackAmount>
      <CanSpawnFromScreen>false</CanSpawnFromScreen>
      <CanBeDropped>true</CanBeDropped>
    </PhysicalItem>
  </PhysicalItems>
</Definitions>
```

---

## Blueprint (Crafting Recipe)

```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Blueprints>
    <Blueprint>
      <Id Type="MyObjectBuilder_BlueprintDefinition" Subtype="MY_RECIPE" />
      <DisplayName>My Recipe Name</DisplayName>
      <Icon>Textures\GUI\Icons\component\MY_COMPONENT.dds</Icon>
      <Prerequisites>
        <Item Amount="10" TypeId="Ore" SubtypeId="Iron" />
        <Item Amount="2" TypeId="Ore" SubtypeId="Nickel" />
      </Prerequisites>
      <Result Amount="1" TypeId="Component" SubtypeId="MY_COMPONENT" />
      <BaseProductionTimeInSeconds>5</BaseProductionTimeInSeconds>
    </Blueprint>
  </Blueprints>
</Definitions>
```

---

## Block Category (Toolbar/G-menu grouping)

```xml
<Definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CategoryClasses>
    <Category>
      <Id Type="MyObjectBuilder_GuiBlockCategoryDefinition" Subtype="MY_CATEGORY" />
      <DisplayName>My Category</DisplayName>
      <ItemIds>
        <string>MyObjectBuilder_CubeBlock/MY_BLOCK_ID_1</string>
        <string>MyObjectBuilder_CubeBlock/MY_BLOCK_ID_2</string>
      </ItemIds>
    </Category>
  </CategoryClasses>
</Definitions>
```

---

## Common SubtypeId Reference (Vanilla Components)

Use these exact SubtypeIds when writing component costs — typos silently fail.

| SubtypeId | Display Name |
|-----------|-------------|
| `SteelPlate` | Steel Plate |
| `Construction` | Construction Component |
| `MetalGrid` | Metal Grid |
| `InteriorPlate` | Interior Plate |
| `SmallTube` | Small Tube |
| `LargeTube` | Large Tube |
| `Motor` | Motor |
| `Display` | Display |
| `BulletproofGlass` | Bulletproof Glass |
| `Girder` | Girder |
| `SolarCell` | Solar Cell |
| `PowerCell` | Power Cell |
| `Superconductor` | Superconductor |
| `Computer` | Computer |
| `Reactor` | Reactor Component |
| `Thrust` | Thruster Component |
| `GravityGenerator` | Gravity Component |
| `Medical` | Medical Component |
| `RadioCommunication` | Radio Component |
| `Detector` | Detector Component |
| `Explosives` | Explosives |
| `ZoneChip` | Zone Chip |

> Cross-reference against `D:\SteamLibrary\steamapps\common\SpaceEngineers\Content\Data\Components.sbc` for definitive list including mass/volume values.
