# RESEARCH.md — Forge 1.20.1 Structure Generation Mod

> Target: Minecraft Java Edition **1.20.1**, Minecraft Forge **47.4.20**, IntelliJ IDEA, Gradle/ForgeGradle **6.x**, Java **17**, Mojang official mappings.
>
> Audience: AI coding agents and humans working inside this repository.
>
> Purpose: prevent wrong-version worldgen mistakes. Minecraft worldgen APIs changed hard across 1.16 → 1.18 → 1.19 → 1.20 → 1.21. Treat random tutorials like radioactive gravel until version-checked.

---

## 0. Research status

This document is scoped to:

```text
Minecraft Java Edition: 1.20.1
Forge:                 47.4.20
ForgeGradle:           6.x project family
Java:                  17 JDK
Mappings:              Mojang official mappings unless build.gradle says otherwise
```

Forge's 1.20.1 file index lists Forge **47.4.20** for Minecraft **1.20.1**, and Forge's 1.20.1 getting-started docs require a **Java 17 JDK** for modern Forge development.

Source priority:

1. Official Forge docs for Forge lifecycle, registries, config, datagen, and Gradle workflow.
2. Vanilla Minecraft **1.20.1** data examples.
3. Maintained real-world Forge structure mods, only after checking branch/version.
4. Community docs for commands and JSON explanation, but only when checked against vanilla 1.20.1 data.

Do **not** silently import advice from Fabric, NeoForge, Bukkit, Paper, Spigot, Bedrock, or Minecraft 1.21+. Those are different ecosystems. Copying them into this repo is not "modernizing." It is poisoning the well.

---

## 1. Fixed environment

Assume this unless the repository proves otherwise:

```text
Game:        Minecraft Java Edition
MC version:  1.20.1
Loader:      Forge
Forge:       47.4.20
IDE:         IntelliJ IDEA
Build:       Gradle + ForgeGradle 6.x
Language:    Java
Java:        JDK 17
Mappings:    Mojang official mappings
```

Use a **JDK**, not just a JRE. In IntelliJ:

```text
File > Project Structure > Project SDK: JDK 17
Settings > Build Tools > Gradle > Gradle JVM: JDK 17
```

Do not switch this project to Java 21 just because newer number feel good. Minecraft 1.20.1 Forge development is a Java 17 target unless the build explicitly proves otherwise. Forge's own 1.20.1 getting-started page says Java 17 JDK.

---

## 2. Expected project layout

A normal Forge MDK-style structure-generation mod should look roughly like this:

```text
.
├─ build.gradle
├─ gradle.properties
├─ settings.gradle
├─ src/
│  ├─ main/
│  │  ├─ java/
│  │  │  └─ <owned.package>/<modid>/
│  │  │     ├─ <ModMain>.java
│  │  │     ├─ registry/
│  │  │     ├─ worldgen/
│  │  │     ├─ config/
│  │  │     ├─ util/
│  │  │     └─ client/
│  │  └─ resources/
│  │     ├─ META-INF/mods.toml
│  │     ├─ pack.mcmeta
│  │     ├─ assets/<modid>/
│  │     └─ data/<modid>/
│  │        ├─ structures/
│  │        ├─ worldgen/
│  │        │  ├─ structure/
│  │        │  ├─ structure_set/
│  │        │  ├─ template_pool/
│  │        │  └─ processor_list/
│  │        └─ tags/
│  │           └─ worldgen/
│  │              └─ biome/
│  │                 └─ has_structure/
│  └─ generated/
│     └─ resources/
└─ run/
```

Forge recommends unique top-level packages and separation of common, client, and data code. Package collisions are not aesthetic crimes; they are crash seeds.

---

## 3. Version-specific path warnings

### 3.1 NBT templates use `structures/`, plural, in 1.20.1

Correct for Minecraft **1.20.1**:

```text
data/<modid>/structures/<path>.nbt
```

Example:

```text
data/examplemod/structures/abandoned_watchtower/watchtower_base.nbt
```

Resource location:

```text
examplemod:abandoned_watchtower/watchtower_base
```

Vanilla 1.20.1 data uses `data/minecraft/structures`, plural.

Wrong for this target:

```text
data/<modid>/structure/<path>.nbt
```

That singular path appears in newer vanilla data layouts such as 1.21. It is **not applicable** to Minecraft 1.20.1.

### 3.2 Worldgen JSON paths are singular

Correct in 1.20.1:

```text
data/<modid>/worldgen/structure/<name>.json
data/<modid>/worldgen/structure_set/<name>.json
data/<modid>/worldgen/template_pool/<path>.json
data/<modid>/worldgen/processor_list/<name>.json
```

Vanilla 1.20.1 data has `worldgen/structure` and `worldgen/structure_set` directories with singular folder names.

### 3.3 Tags use datapack tag paths

Biome tags for structure spawning go here:

```text
data/<modid>/tags/worldgen/biome/has_structure/<name>.json
```

Forge's tag docs explain the datapack tag path pattern and tag file structure. Biomes and structures are supported tag key types.

---

## 4. The 1.20.1 structure-generation pipeline

A naturally spawning jigsaw structure usually needs:

```text
1. NBT template(s)
   data/<modid>/structures/<path>.nbt

2. Template pool(s)
   data/<modid>/worldgen/template_pool/<path>.json

3. Optional processor list(s)
   data/<modid>/worldgen/processor_list/<name>.json

4. Structure definition
   data/<modid>/worldgen/structure/<name>.json

5. Structure set
   data/<modid>/worldgen/structure_set/<name>.json

6. Biome tag or biome list
   data/<modid>/tags/worldgen/biome/has_structure/<name>.json
```

Flow:

```text
structure_set selects candidate chunks
        ↓
structure definition checks type, biome, step, height, and generator rules
        ↓
jigsaw generator reads start_pool
        ↓
template pool chooses NBT pieces by weight
        ↓
processors alter blocks during placement
        ↓
structure appears if placement succeeds
```

If one link is broken, the structure either fails during datapack load, places empty, refuses `/place`, or never appears naturally. There is no Forge fairy that fixes bad JSON.

---

## 5. Structure definition JSON

Path:

```text
src/main/resources/data/<modid>/worldgen/structure/<structure_name>.json
```

For ordinary custom jigsaw structures, start with vanilla:

```json
{
  "type": "minecraft:jigsaw",
  "biomes": "#examplemod:has_structure/abandoned_watchtower",
  "step": "surface_structures",
  "terrain_adaptation": "none",
  "spawn_overrides": {},
  "start_pool": "examplemod:abandoned_watchtower/start_pool",
  "size": 1,
  "start_height": {
    "absolute": 0
  },
  "project_start_to_heightmap": "WORLD_SURFACE_WG",
  "max_distance_from_center": 80,
  "use_expansion_hack": false
}
```

Vanilla 1.20.1 villages use `minecraft:jigsaw`, biome tags, `start_pool`, `size`, `start_height`, `project_start_to_heightmap`, `max_distance_from_center`, generation step, terrain adaptation, and `use_expansion_hack`. Trail ruins and ancient cities show additional variants such as `bury`, `beard_box`, `underground_structures`, `underground_decoration`, and `start_jigsaw_name`.

### 5.1 Field notes

| Field | 1.20.1 guidance |
|---|---|
| `type` | Use `minecraft:jigsaw` unless Java custom behavior is required. |
| `biomes` | Biome id, biome tag, or biome list. Prefer a tag for maintainability. |
| `step` | Common values include `surface_structures`, `underground_structures`, and `underground_decoration`. Match the structure's behavior, not a tutorial's vibes. |
| `terrain_adaptation` | `none` is boring and safe for a first test. Vanilla uses `beard_thin` for villages, `bury` for trail ruins, and `beard_box` for ancient cities. Copy only when you understand why. |
| `spawn_overrides` | Empty object is valid. Non-empty overrides are category-based and should be copied from vanilla shape carefully. |
| `start_pool` | First template pool used by jigsaw generation. Must point to a real `worldgen/template_pool` JSON. |
| `start_jigsaw_name` | Optional. Used by some vanilla structures to select a named starting jigsaw. Do not add it unless your NBT contains a matching jigsaw target/name setup. |
| `size` | Jigsaw expansion depth. Use `1` for a single-piece test. Vanilla complex structures use values like `6` or `7`; don't crank this like a goblin with a volume knob. |
| `start_height` | Height provider. `{ "absolute": 0 }` is common when using heightmap projection. |
| `project_start_to_heightmap` | Projects the start Y to a heightmap such as `WORLD_SURFACE_WG` or `OCEAN_FLOOR_WG`. |
| `max_distance_from_center` | Limits piece expansion from the start. Vanilla villages commonly use `80`. |
| `use_expansion_hack` | Vanilla compatibility behavior. Use `false` unless copying a vanilla pattern that needs it. Villages use `true`; many non-village jigsaw structures use `false`. |

### 5.2 When to use a custom Java structure type

Use Java only when vanilla `minecraft:jigsaw` cannot express the rule.

Good reasons:

```text
- reject positions after checking blocks/fluids at generation time
- require custom Y or terrain rules beyond vanilla height providers
- add special dimension or biome logic beyond tags
- use custom structure placement behavior
- add custom processor behavior
```

Bad reasons:

```text
- changing biome list
- changing spacing
- changing separation
- changing salt
- changing start pool
- changing NBT piece weights
- adding moss/cracked block processors already expressible in JSON
```

If your JSON says:

```json
"type": "examplemod:water_ruin"
```

then Java must register a matching `StructureType` and provide a codec that accepts the JSON fields. Forge's codec docs explain how Mojang-style codecs serialize and validate data; bad field names become codec errors, not polite suggestions.

---

## 6. Structure set JSON

Path:

```text
src/main/resources/data/<modid>/worldgen/structure_set/<set_name>.json
```

Example:

```json
{
  "structures": [
    {
      "structure": "examplemod:abandoned_watchtower",
      "weight": 1
    }
  ],
  "placement": {
    "type": "minecraft:random_spread",
    "spacing": 24,
    "separation": 8,
    "salt": 91827364
  }
}
```

Vanilla 1.20.1 structure sets use this shape: weighted structure entries plus a `placement` object such as `minecraft:random_spread` with `spacing`, `separation`, and `salt`.

### 6.1 Field notes

| Field | Meaning |
|---|---|
| `structures` | List of structures in this set. Each entry has `structure` and `weight`. |
| `weight` | Relative selection chance inside this set. It does **not** make the set itself spawn more often. |
| `placement.type` | `minecraft:random_spread` is the normal scattered-structure placement type. |
| `spacing` | Grid spacing in chunks. Larger means rarer attempts. |
| `separation` | Minimum chunk distance inside the grid. Must be less than `spacing`. |
| `salt` | Integer mixed with the world seed for this set. Use a stable, unique-ish salt per set. |

### 6.2 Practical rarity values

Use noisy, common values while debugging:

```text
Debug:        spacing 12, separation 4
Normal-ish:   spacing 24-40, separation 8-16
Rare:         spacing 48+, separation 16+
```

Do not test a broken structure with rare spacing and then declare worldgen cursed. You are debugging absence. Make it common, prove it works, then make it rare.

### 6.3 Old chunks do not regenerate

Natural structures are placed during chunk generation. If you change a structure set, test in:

```text
- a fresh world, or
- newly generated chunks far away from old terrain
```

Existing chunks are not your test lab unless you are specifically testing upgrade behavior.

---

## 7. Biome restrictions

Preferred pattern: use a biome tag.

Path:

```text
src/main/resources/data/<modid>/tags/worldgen/biome/has_structure/<structure_name>.json
```

Example:

```json
{
  "replace": false,
  "values": [
    "minecraft:plains",
    "minecraft:forest",
    "minecraft:taiga"
  ]
}
```

Then reference it in the structure JSON:

```json
"biomes": "#examplemod:has_structure/abandoned_watchtower"
```

Vanilla 1.20.1 uses biome tags like `has_structure/village_plains` to control where structures may appear.

### 7.1 Optional tag entries

Forge tag JSON supports optional entries using an object with `id` and `required: false`. Use this only when referencing biomes from optional dependencies. Do not use optional syntax as a lazy typo shield.

Example:

```json
{
  "replace": false,
  "values": [
    "minecraft:plains",
    {
      "id": "othermod:some_optional_biome",
      "required": false
    }
  ]
}
```

### 7.2 Biome modifiers are not the normal fix

For ordinary 1.20.1 structure spawning, use:

```text
worldgen/structure
worldgen/structure_set
tags/worldgen/biome
```

Do not rush into Forge biome modifiers because a tutorial waved a shiny JSON hammer at you. If the problem is "my structure does not spawn," inspect the structure definition, structure set, biome tag, and logs first.

---

## 8. Template pools

Path:

```text
src/main/resources/data/<modid>/worldgen/template_pool/<pool_path>.json
```

Example:

```json
{
  "fallback": "minecraft:empty",
  "elements": [
    {
      "weight": 1,
      "element": {
        "element_type": "minecraft:single_pool_element",
        "location": "examplemod:abandoned_watchtower/watchtower_base",
        "processors": "examplemod:mossify_watchtower",
        "projection": "rigid"
      }
    }
  ]
}
```

Vanilla 1.20.1 template pools use `fallback`, weighted `elements`, pool element types, NBT `location`, `processors`, and `projection`. They also show `processors` as either a referenced processor list id or an inline object such as `{ "processors": [] }`.

### 8.1 NBT location mapping

This pool entry:

```json
"location": "examplemod:abandoned_watchtower/watchtower_base"
```

requires this file:

```text
data/examplemod/structures/abandoned_watchtower/watchtower_base.nbt
```

Again: in 1.20.1, that folder is `structures/`, plural. Do not "fix" it to singular unless your plan is to break the mod.

### 8.2 Inline empty processors

For a simple test piece, this is valid and avoids creating a processor list file:

```json
{
  "fallback": "minecraft:empty",
  "elements": [
    {
      "weight": 1,
      "element": {
        "element_type": "minecraft:single_pool_element",
        "location": "examplemod:abandoned_watchtower/watchtower_base",
        "processors": {
          "processors": []
        },
        "projection": "rigid"
      }
    }
  ]
}
```

Use a separate `worldgen/processor_list` when the processor list is reused or non-trivial.

### 8.3 Pool field notes

| Field | Guidance |
|---|---|
| `fallback` | Usually `minecraft:empty`. If a selected element cannot expand, fallback can be used. |
| `weight` | Selection weight inside this pool only. It does not affect natural spawn rarity. |
| `element_type` | `minecraft:single_pool_element` is normal for one template. `minecraft:legacy_single_pool_element` exists in vanilla; use it deliberately, not randomly. |
| `location` | Resource location for an NBT template under `data/<namespace>/structures/`. |
| `processors` | Either a processor list id or inline processor list object. |
| `projection` | `rigid` keeps the piece fixed. `terrain_matching` adapts to terrain and is often used for roads/paths. |

### 8.4 Jigsaw target names

Multi-piece jigsaw structures depend on jigsaw blocks inside the NBT files. Their `name`, `target`, and pool references must line up. If `/place jigsaw` says the pool is empty or cannot expand, inspect the actual jigsaw block data in the NBT instead of inventing Java.

If the structure JSON uses `start_jigsaw_name`, the named jigsaw must exist and match. Vanilla ancient city JSON shows this field in 1.20.1.

---

## 9. Processor lists

Path:

```text
src/main/resources/data/<modid>/worldgen/processor_list/<name>.json
```

Empty processor list:

```json
{
  "processors": []
}
```

Example rule processor:

```json
{
  "processors": [
    {
      "processor_type": "minecraft:rule",
      "rules": [
        {
          "input_predicate": {
            "predicate_type": "minecraft:random_block_match",
            "block": "minecraft:cobblestone",
            "probability": 0.15
          },
          "location_predicate": {
            "predicate_type": "minecraft:always_true"
          },
          "output_state": {
            "Name": "minecraft:mossy_cobblestone"
          }
        }
      ]
    }
  ]
}
```

Vanilla 1.20.1 processor lists use this `processors` array shape and rule processors with predicates and output states.

Use processors for:

```text
- moss/crack/random decay
- block replacement
- structure variants without duplicating NBT files
- cleanup and protection rules expressible by vanilla processors
```

Do not use processors for:

```text
- spawn rate
- biome restrictions
- structure set spacing
- dimension checks
```

Wrong tool. Bad smell.

---

## 10. NBT templates

Path for Minecraft 1.20.1:

```text
src/main/resources/data/<modid>/structures/<path>.nbt
```

How to create:

1. Build the structure in a dev world.
2. Use a Structure Block.
3. Save the template.
4. Copy the `.nbt` from the world save into the mod resources.
5. Reference it from a template pool by resource location.

Example:

```text
File:
src/main/resources/data/examplemod/structures/abandoned_watchtower/watchtower_base.nbt

Resource location:
examplemod:abandoned_watchtower/watchtower_base
```

Rules:

```text
- Keep namespace and paths lowercase.
- Use structure_void where the template should not replace existing blocks.
- Use jigsaw blocks only if the structure has multiple pieces or expansion points.
- For a single-piece structure, a one-element template pool is fine.
- Remove accidental dev-only structure blocks unless intentional.
```

To test raw NBT template loading, use `/place template`, not `/place structure`. `/place template` checks the template resource. `/place structure` checks the structure registry entry and generator path. Minecraft command references distinguish `/place template`, `/place jigsaw`, and `/place structure`.

---

## 11. Datapack vs Java responsibilities

### 11.1 JSON/datapack owns this

Use JSON/data for:

```text
- structure definition
- structure set
- spacing
- separation
- salt
- biome restrictions
- template pools
- pool weights
- vanilla processors
- NBT piece references
- terrain adaptation
- generation step
```

### 11.2 Java owns this

Use Java for:

```text
- custom Structure subclass behavior
- custom StructureType registration
- custom StructurePlacement / placement type
- custom StructureProcessor / processor type
- custom checks that vanilla JSON cannot express
- debug logging or dev-only commands
- config plumbing
- datagen providers
```

### 11.3 The registry trap

Forge registry docs distinguish ordinary Forge registries from dynamic datapack registries. Dynamic registry objects are registered through data files, not code. That is exactly why structure JSON and structure sets live under `data/<modid>/worldgen/...`.

Important distinction:

```text
Structure definition:
  data/<modid>/worldgen/structure/<name>.json
  datapack registry entry

Structure set:
  data/<modid>/worldgen/structure_set/<name>.json
  datapack registry entry

Structure type:
  Java-registered type used by the structure JSON "type" field
```

Do not register a structure definition like a block. Do not put a custom Java `StructureType` only in JSON. These are different layers.

---

## 12. Java registration notes

Forge's recommended registration path is `DeferredRegister`, and `RegistryObject` is the normal holder for registered objects.

### 12.1 Custom `StructureType` registration pattern

Only use this when you have a custom `Structure` subclass.

```java
public final class ModStructureTypes {
    public static final DeferredRegister<StructureType<?>> STRUCTURE_TYPES =
            DeferredRegister.create(Registries.STRUCTURE_TYPE, ExampleMod.MOD_ID);

    public static final RegistryObject<StructureType<WaterRuinStructure>> WATER_RUIN =
            STRUCTURE_TYPES.register("water_ruin", () -> () -> WaterRuinStructure.CODEC);

    public static void register(IEventBus modBus) {
        STRUCTURE_TYPES.register(modBus);
    }

    private ModStructureTypes() {}
}
```

Main mod constructor pattern:

```java
@Mod(ExampleMod.MOD_ID)
public final class ExampleMod {
    public static final String MOD_ID = "examplemod";
    public static final Logger LOGGER = LogUtils.getLogger();

    public ExampleMod() {
        IEventBus modBus = FMLJavaModLoadingContext.get().getModEventBus();

        ModStructureTypes.register(modBus);

        modBus.addListener(this::commonSetup);
        modBus.addListener(ModDataGenerators::gatherData);

        ModLoadingContext.get().registerConfig(ModConfig.Type.COMMON, CommonConfig.SPEC);
    }

    private void commonSetup(final FMLCommonSetupEvent event) {
        event.enqueueWork(() -> LOGGER.info("Common setup complete for {}", MOD_ID));
    }
}
```

Check imports against the actual project and decompiled 1.20.1 sources. Do not "repair" this into Fabric `Registry.register` or Yarn `Identifier`. Forge 1.20.1 with Mojang mappings uses `ResourceLocation`, Forge registries, and `net.minecraft.core.registries.Registries`.

### 12.2 Codec guidance

A custom structure type is only as good as its codec. The codec must describe exactly what your JSON accepts.

Rules:

```text
- Include every required JSON field your custom structure needs.
- Do not rename fields without updating JSON.
- Do not copy 1.21 codec examples into 1.20.1.
- Do not copy Fabric/Yarn names into Mojang mappings.
- Use the IDE/decompiled 1.20.1 sources as the final authority.
```

Forge's codec docs cover `Codec`, `RecordCodecBuilder`, field definitions, and `DataResult` error behavior. Use them when reading Minecraft's codec-heavy worldgen code.

Real-world note: Repurposed Structures has a 1.20.1 branch and uses custom structure types with JSON structure entries. It is useful as a pattern reference, not as a starter template to blindly transplant.

---

## 13. Main mod class responsibilities

The main mod class should:

```text
- define MOD_ID
- create/register mod event bus hooks
- register DeferredRegister containers
- register config specs
- register datagen listeners
- do minimal setup
```

The main mod class should **not**:

```text
- contain every registry object
- parse worldgen JSON manually
- place structures manually during construction
- reference client-only classes directly
- perform dynamic registry lookups too early
```

Forge lifecycle docs distinguish the mod event bus from the main Forge event bus, and note that some lifecycle work runs in parallel and must use `enqueueWork` for synchronized work.

---

## 14. Suggested package layout

```text
<base>.<modid>/
├─ <ModMain>.java
├─ registry/
│  ├─ ModBlocks.java
│  ├─ ModItems.java
│  ├─ ModStructureTypes.java
│  ├─ ModProcessors.java
│  └─ ModPlacementTypes.java
├─ worldgen/
│  ├─ structure/
│  ├─ placement/
│  └─ processor/
├─ data/
│  ├─ ModDataGenerators.java
│  ├─ ModBiomeTagsProvider.java
│  └─ ModWorldgenProvider.java
├─ config/
│  └─ CommonConfig.java
├─ util/
│  ├─ ModIds.java
│  └─ DebugLog.java
└─ client/
```

Forge docs specifically recommend isolating client-only code so dedicated servers do not load client classes. Run the server. Let it punch weak architecture early.

---

## 15. Datagen setup

Use datagen when JSON becomes repetitive or easy to mistype.

Useful generated data:

```text
- biome tags
- block/item tags
- loot tables
- recipes
- language files
- model and blockstate JSON
- datapack registry entries, if the project is mature enough
```

Command:

```bash
./gradlew runData
```

Forge datagen docs document the `runData` task, `GatherDataEvent`, providers, and `DatapackBuiltinEntriesProvider` for dynamic registry data.

If using generated resources, Gradle usually needs something like:

```groovy
sourceSets.main.resources { srcDir 'src/generated/resources' }
```

Check the existing `build.gradle` before adding duplicate source sets.

Datagen is not mandatory for the first structure. Manual JSON is often faster for one or two structures. Datagen becomes worth it when the mod has repeated biome tags, many pools, many variants, or registry bootstrap data that must stay consistent.

Do not hand-edit generated files unless you enjoy losing your work to `runData`.

---

## 16. Config setup

Use Forge config for runtime/mod settings, not as fake datapack mutation.

Good config candidates:

```text
debugLogging = true
logStructureChecks = false
enableExperimentalStructures = true
```

Bad config candidates unless backed by deliberate custom code:

```text
spacing = 32
separation = 12
biomes = [...]
```

Why? Structure sets are datapack registry data. A TOML value does not magically rewrite already-loaded worldgen registries. Forge config uses `ForgeConfigSpec` and TOML-backed config files; it does not automatically patch datapack JSON.

If configurable spacing is required, choose one deliberately:

```text
- document that users should edit datapack JSON
- generate JSON from config before packaging
- implement custom placement logic
```

Do not pretend a config value controls worldgen unless code actually connects it.

---

## 17. IntelliJ IDEA and Gradle workflow

Correct import workflow:

```text
1. Open IntelliJ IDEA.
2. Open the repository root, not src/.
3. Import/open as a Gradle project.
4. Set Project SDK to JDK 17.
5. Set Gradle JVM to JDK 17.
6. Let IntelliJ sync.
7. Run genIntellijRuns if run configs are missing.
```

Command:

```bash
./gradlew genIntellijRuns
```

Windows:

```bat
gradlew.bat genIntellijRuns
```

ForgeGradle docs describe importing the MDK into the IDE and using `genIntellijRuns`; Forge's getting-started docs list common Gradle run tasks such as `runClient`, `runServer`, and `runData`.

Common tasks:

```bash
./gradlew genIntellijRuns
./gradlew runClient
./gradlew runServer
./gradlew runData
./gradlew build
./gradlew clean build
./gradlew --refresh-dependencies
```

Use `runServer` before release. Integrated client success means less than people think. Dedicated servers are where client-only imports go to die.

---

## 18. AI-agent guidance

### 18.1 Inspect these files first

Before editing anything, inspect:

```text
build.gradle
gradle.properties
settings.gradle
src/main/resources/META-INF/mods.toml
src/main/java/**/<main mod class>.java
src/main/java/**/registry/
src/main/java/**/worldgen/
src/main/resources/data/<modid>/worldgen/
src/main/resources/data/<modid>/tags/worldgen/biome/
src/main/resources/data/<modid>/structures/
run/logs/latest.log
run/logs/debug.log
```

Also read the repository README and existing architecture notes if present. Do not bulldoze project style because a tutorial used different class names.

### 18.2 Edit JSON when changing this

```text
- where structures spawn
- rarity
- spacing/separation/salt
- biome restrictions
- template pools
- piece weights
- processors
- terrain adaptation
- start pool
- generation step
- NBT piece references
```

### 18.3 Edit Java when changing this

```text
- custom placement validation
- custom Structure subclass behavior
- custom StructureType
- custom processor type
- custom placement type
- debug commands
- config plumbing
- datagen providers
```

If an agent edits Java to change a biome list, it is probably making the mod worse.

### 18.4 Safe modification rules

```text
1. Keep MOD_ID consistent across Java, mods.toml, resource folders, JSON ids, and registries.
2. Do not rename resource paths unless updating every reference.
3. Do not use 1.21 datapack folder names in 1.20.1.
4. Do not add Fabric, NeoForge, Bukkit, Paper, Spigot, or Bedrock imports.
5. Do not perform registry lookups in static initializers unless the project already safely does so.
6. Do not place client-only imports in common code.
7. Make the smallest correct change.
8. Validate JSON strictly: no comments, no trailing commas, lowercase ids.
9. Test in a fresh world or new chunks after worldgen changes.
10. If a tutorial says StructureFeature or ConfiguredStructureFeature, assume it is old until proven otherwise.
```

---

## 19. Debugging commands and diagnostic ladder

Use commands as layered tests, not magic rituals.

```mcfunction
/reload
/place template <modid>:<template_path> ~ ~ ~
/place jigsaw <modid>:<pool_path> <target_name> 1 ~ ~ ~
/place structure <modid>:<structure_name> ~ ~ ~
/locate structure <modid>:<structure_name>
```

Minecraft's command docs show `/locate structure` syntax, and community command references document `/place template`, `/place jigsaw`, and `/place structure` as distinct subcommands.

Diagnostic meaning:

| Command | What it tests |
|---|---|
| `/reload` | Datapack JSON parse/reload. Check logs after running. |
| `/place template` | Raw NBT template resource path. Bypasses structure JSON and structure set. |
| `/place jigsaw` | Template pool and jigsaw expansion. Useful for pool/target debugging. |
| `/place structure` | Structure registry entry and generator behavior. |
| `/locate structure` | Natural placement: structure set, biome restrictions, spacing, and valid candidate chunks. |

If `/place template` fails, stop touching Java. Your NBT path is probably wrong.

If `/place jigsaw` fails, inspect the pool, jigsaw block target names, and NBT references.

If `/place structure` fails, inspect the structure JSON, codec errors, biome restrictions, height rules, and custom type registration.

If `/place structure` works but `/locate structure` fails, inspect the structure set, spacing, separation, biome tag, dimension, and whether you are searching generated old chunks.

---

## 20. Troubleshooting

### 20.1 Structure does not spawn naturally

Checklist:

```text
- Does data/<modid>/worldgen/structure/<name>.json exist?
- Does data/<modid>/worldgen/structure_set/<name>.json exist?
- Does the structure set reference the exact structure id?
- Is spacing reasonable for testing?
- Is separation smaller than spacing?
- Is salt an integer?
- Does the structure JSON biomes field reference a real biome or tag?
- Does the biome tag exist under tags/worldgen/biome/...?
- Are you testing new chunks or a fresh world?
- Does /locate structure <modid>:<name> find it?
- Does /place structure <modid>:<name> ~ ~ ~ place it?
```

Fast test placement:

```json
"spacing": 12,
"separation": 4
```

Rare settings during development are self-sabotage wearing a hat.

### 20.2 `/place template` fails

Likely causes:

```text
- NBT under data/<modid>/structure instead of data/<modid>/structures
- wrong namespace
- wrong path
- uppercase path characters
- template was not copied into src/main/resources
```

### 20.3 `/place jigsaw` fails

Likely causes:

```text
- missing template pool JSON
- pool path mismatch
- empty pool
- bad element location
- missing NBT template
- wrong jigsaw target name
- invalid max depth
```

The `/place jigsaw` command takes a pool, a target name, and a max depth; community command docs list the Java syntax and note validation errors for missing pools/templates and invalid biomes.

### 20.4 `/place structure` fails

Likely causes:

```text
- bad structure JSON
- missing start_pool
- wrong type
- codec parse failure
- biome at target rejected
- invalid height placement
- broken custom StructureType
```

Check `latest.log` and `debug.log`. Read the first meaningful error, not the last page of stacktrace confetti.

### 20.5 `/locate structure` fails but `/place structure` works

Likely causes:

```text
- missing or broken structure_set
- spacing/separation too rare
- biome restriction excludes nearby terrain
- structure cannot generate in this dimension
- searched area has old generated chunks
```

Fix by testing a fresh world with common placement values.

### 20.6 Missing template pool

Symptom:

```text
Unknown registry key in ResourceKey[minecraft:root / minecraft:worldgen/template_pool]
```

If `start_pool` is:

```json
"examplemod:abandoned_watchtower/start_pool"
```

then the file must be:

```text
data/examplemod/worldgen/template_pool/abandoned_watchtower/start_pool.json
```

### 20.7 Missing NBT template

If pool `location` is:

```json
"examplemod:abandoned_watchtower/watchtower_base"
```

then NBT must be:

```text
data/examplemod/structures/abandoned_watchtower/watchtower_base.nbt
```

Plural `structures`. Drill it into the wall.

### 20.8 Codec errors

Codec errors usually mean JSON and Java disagree.

Common causes:

```text
- wrong type id
- missing required JSON field
- field has wrong type
- custom codec forgot a field
- custom registry object not registered
- copied 1.21 JSON into 1.20.1
- copied Fabric/Yarn names into Mojang mappings
- typo in resource location
```

Debug method:

```text
1. Read the first codec error.
2. Identify the JSON file.
3. Compare fields to the actual 1.20.1 codec.
4. Fix one field.
5. Relaunch or /reload.
```

Do not shotgun-edit ten files. That is not debugging. That is archaeology with a shovel made of panic.

### 20.9 Datapack loading errors

Common causes:

```text
- trailing comma
- JSON comments
- uppercase namespace/path
- wrong folder name
- wrong pack metadata copied from another MC version
- duplicate registry ids
- generated resources shadowing manual resources
- invalid tag path
```

External JSON validators only check syntax. Minecraft's codec still gets the final punch.

### 20.10 Registry errors

Common causes:

```text
- DeferredRegister not registered to the mod event bus
- registry object accessed too early
- MOD_ID mismatch
- duplicate id
- static Java registration attempted for dynamic datapack data
- missing Java registration for custom type used by JSON
- wrong import from another loader ecosystem
```

Forge docs cover mod lifecycle, registry events, and event bus separation. Use the mod event bus for mod registration.

### 20.11 Dedicated server crash

Likely causes:

```text
- client-only class loaded on server
- renderer/screen/keybind referenced from common code
- dependency marked incorrectly
- client package touched by common setup
```

Fix:

```text
- isolate client code under client/
- register client-only handlers only on client side
- run ./gradlew runServer before release
```

---

## 21. Minimal single-piece jigsaw structure

Replace `examplemod` and names.

### 21.1 Biome tag

`data/examplemod/tags/worldgen/biome/has_structure/abandoned_watchtower.json`

```json
{
  "replace": false,
  "values": [
    "minecraft:plains",
    "minecraft:forest"
  ]
}
```

### 21.2 Template pool with inline empty processors

`data/examplemod/worldgen/template_pool/abandoned_watchtower/start_pool.json`

```json
{
  "fallback": "minecraft:empty",
  "elements": [
    {
      "weight": 1,
      "element": {
        "element_type": "minecraft:single_pool_element",
        "location": "examplemod:abandoned_watchtower/watchtower_base",
        "processors": {
          "processors": []
        },
        "projection": "rigid"
      }
    }
  ]
}
```

Required NBT:

```text
data/examplemod/structures/abandoned_watchtower/watchtower_base.nbt
```

### 21.3 Structure

`data/examplemod/worldgen/structure/abandoned_watchtower.json`

```json
{
  "type": "minecraft:jigsaw",
  "biomes": "#examplemod:has_structure/abandoned_watchtower",
  "step": "surface_structures",
  "terrain_adaptation": "none",
  "spawn_overrides": {},
  "start_pool": "examplemod:abandoned_watchtower/start_pool",
  "size": 1,
  "start_height": {
    "absolute": 0
  },
  "project_start_to_heightmap": "WORLD_SURFACE_WG",
  "max_distance_from_center": 80,
  "use_expansion_hack": false
}
```

Why `terrain_adaptation: "none"` here? Because this is a minimal test. Use `beard_thin`, `bury`, or `beard_box` only when you intentionally want those terrain interactions and have checked vanilla examples.

### 21.4 Structure set

`data/examplemod/worldgen/structure_set/abandoned_watchtower.json`

```json
{
  "structures": [
    {
      "structure": "examplemod:abandoned_watchtower",
      "weight": 1
    }
  ],
  "placement": {
    "type": "minecraft:random_spread",
    "spacing": 24,
    "separation": 8,
    "salt": 91827364
  }
}
```

For debugging, temporarily use:

```json
"spacing": 12,
"separation": 4
```

### 21.5 Optional processor list

`data/examplemod/worldgen/processor_list/mossify_watchtower.json`

```json
{
  "processors": [
    {
      "processor_type": "minecraft:rule",
      "rules": [
        {
          "input_predicate": {
            "predicate_type": "minecraft:random_block_match",
            "block": "minecraft:cobblestone",
            "probability": 0.15
          },
          "location_predicate": {
            "predicate_type": "minecraft:always_true"
          },
          "output_state": {
            "Name": "minecraft:mossy_cobblestone"
          }
        }
      ]
    }
  ]
}
```

Then in the pool, replace inline empty processors with:

```json
"processors": "examplemod:mossify_watchtower"
```

---

## 22. Verification order

Use this order:

```bash
./gradlew runData
./gradlew build
./gradlew runClient
./gradlew runServer
```

In game:

```mcfunction
/reload
/place template examplemod:abandoned_watchtower/watchtower_base ~ ~ ~
/place structure examplemod:abandoned_watchtower ~ ~ ~
/locate structure examplemod:abandoned_watchtower
```

For jigsaw debugging:

```mcfunction
/place jigsaw examplemod:abandoned_watchtower/start_pool minecraft:empty 1 ~ ~ ~
```

The exact `<target_name>` for `/place jigsaw` depends on your jigsaw blocks. If your NBT uses a real target name, use that. Do not cargo-cult `minecraft:empty` if your jigsaw setup says otherwise.

---

## 23. Common wrong-version tutorial traps

| Bad advice | Why it is bad for this project |
|---|---|
| Use `StructureFeature` / `ConfiguredStructureFeature` | Old pipeline. Not the normal 1.20.1 structure JSON pipeline. |
| Use Fabric `BiomeModifications.addStructure` | Fabric API, not Forge. |
| Use `Identifier` | Yarn/Fabric naming. Mojang mappings use `ResourceLocation`. |
| Put NBT under `data/<modid>/structure` | 1.21-style singular path. Minecraft 1.20.1 uses `structures/`. |
| Use Bukkit/Paper APIs | Server plugin APIs, not Forge mods. |
| Fix normal structure spawning with Forge biome modifiers | Wrong first tool. Use structure JSON, structure sets, and biome tags. |
| Change spacing in TOML and expect JSON worldgen to change | Structure placement is datapack registry data unless custom code bridges it. |
| Test rare structures with huge spacing | You are debugging absence. Use common test values first. |
| Edit generated files by hand | `runData` can overwrite them. Edit providers or manual resources intentionally. |
| Copy 1.21 examples silently | This target is 1.20.1. Version drift is the bug factory. |

---

## 24. Source map

Use these as anchors when modifying this document or the mod:

### Official Forge sources

- [Forge 1.20.1 downloads](https://files.minecraftforge.net/net/minecraftforge/forge/index_1.20.1.html) — confirms Forge 47.4.20 for Minecraft 1.20.1.
- [Forge 1.20.1 getting started](https://docs.minecraftforge.net/en/1.20.1/gettingstarted/) — Java 17 JDK requirement, Gradle tasks, IntelliJ run generation.
- [ForgeGradle 6.x docs](https://docs.minecraftforge.net/en/fg-6.x/) — MDK setup and IDE import workflow.
- [Forge registries](https://docs.minecraftforge.net/en/1.20.1/concepts/registries/) — `DeferredRegister`, `RegistryObject`, dynamic registry limitations.
- [Forge lifecycle](https://docs.minecraftforge.net/en/1.20.1/concepts/lifecycle/) — lifecycle timing and setup events.
- [Forge events](https://docs.minecraftforge.net/en/1.20.1/concepts/events/) — mod event bus vs Forge event bus.
- [Forge datagen](https://docs.minecraftforge.net/en/1.20.1/datagen/) — `runData`, `GatherDataEvent`, providers, datapack registry generation.
- [Forge config](https://docs.minecraftforge.net/en/1.20.1/misc/config/) — `ForgeConfigSpec` and TOML config behavior.
- [Forge tags](https://docs.minecraftforge.net/en/1.20.1/resources/server/tags/) — tag file paths, optional tag entries, biome/structure tag keys.
- [Forge codecs](https://docs.minecraftforge.net/en/1.20.1/datastorage/codecs/) — `Codec`, `RecordCodecBuilder`, and codec error behavior.

### Vanilla 1.20.1 data examples

- [Vanilla 1.20.1 structures folder](https://github.com/misode/mcmeta/tree/1.20.1-data/data/minecraft/structures)
- [Vanilla 1.20.1 worldgen structures](https://github.com/misode/mcmeta/tree/1.20.1-data/data/minecraft/worldgen/structure)
- [Vanilla 1.20.1 structure sets](https://github.com/misode/mcmeta/tree/1.20.1-data/data/minecraft/worldgen/structure_set)
- [Vanilla 1.20.1 template pools](https://github.com/misode/mcmeta/tree/1.20.1-data/data/minecraft/worldgen/template_pool)
- [Vanilla 1.20.1 processor lists](https://github.com/misode/mcmeta/tree/1.20.1-data/data/minecraft/worldgen/processor_list)
- [Village plains structure JSON](https://raw.githubusercontent.com/misode/mcmeta/1.20.1-data/data/minecraft/worldgen/structure/village_plains.json)
- [Ancient city structure JSON](https://raw.githubusercontent.com/misode/mcmeta/1.20.1-data/data/minecraft/worldgen/structure/ancient_city.json)
- [Village structure set JSON](https://raw.githubusercontent.com/misode/mcmeta/1.20.1-data/data/minecraft/worldgen/structure_set/villages.json)
- [Village plains town center pool JSON](https://raw.githubusercontent.com/misode/mcmeta/1.20.1-data/data/minecraft/worldgen/template_pool/village/plains/town_centers.json)
- [Mossify 20 percent processor list](https://raw.githubusercontent.com/misode/mcmeta/1.20.1-data/data/minecraft/worldgen/processor_list/mossify_20_percent.json)
- [Village plains biome tag](https://raw.githubusercontent.com/misode/mcmeta/1.20.1-data/data/minecraft/tags/worldgen/biome/has_structure/village_plains.json)

### Command references

- [Minecraft commands overview](https://www.minecraft.net/en-us/article/minecraft-commands)
- [Minecraft Wiki — `/locate`](https://minecraft.fandom.com/wiki/Commands/locate)
- [Minecraft Wiki — `/place` Java Edition](https://minecraft.fandom.com/wiki/Commands/place_%28Java_Edition%29)

### Real-world Forge structure mod reference

- [Repurposed Structures branches](https://github.com/TelepathicGrunt/RepurposedStructures/branches/all) — useful for advanced custom structure patterns. Check branch/version before copying anything.

---

## 25. Final instruction for AI agents

When in doubt:

```text
1. Read existing project files.
2. Identify whether the target change is JSON/data or Java behavior.
3. Check the exact Minecraft/Forge version.
4. Avoid Fabric/NeoForge/Bukkit/Paper/Bedrock APIs.
5. Make the smallest correct change.
6. Run build.
7. Test with /place template, /place jigsaw, /place structure, and /locate.
8. Test in a fresh world or new chunks.
9. Only then tune rarity or architecture.
```

Most structure bugs are not mystical. They are wrong paths, wrong ids, wrong versions, broken codecs, or agents pretending they know APIs they never checked. Do the boring verification. It wins.
