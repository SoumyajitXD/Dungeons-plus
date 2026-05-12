# AGENTS.md

Instructions for AI coding agents working in this repository.

This is a **Minecraft Java Edition Forge 1.20.1 structure-generation mod**. Treat it like one. Do not drag in APIs, layout patterns, or tutorial code from other loaders, newer versions, plugins, or Bedrock packs.

---

## 1. Target Stack

Do not change this stack unless the user explicitly asks for a migration.

| Area | Required |
|---|---|
| Game | Minecraft Java Edition |
| Minecraft version | `1.20.1` |
| Mod loader | Forge |
| Forge version | `47.4.20` |
| Java | Java 17 |
| Build | Gradle / ForgeGradle |
| IDE | IntelliJ IDEA |
| Mod focus | Structure generation / worldgen |

Version bumps are not cleanup. They are migrations. Treat them like live explosives.

---

## 2. Hard Rules

### Do Not Use Other Platforms

Never introduce APIs, imports, file formats, or assumptions from:

- Fabric
- NeoForge
- Bukkit
- Spigot
- Paper
- Bedrock Edition
- Datapack formats from unrelated Minecraft versions

Forbidden examples:

```text
net.fabricmc.*
net.neoforged.*
org.bukkit.*
org.spigotmc.*
io.papermc.*
Bedrock behavior packs
```

This repository targets **Forge 1.20.1**. Stay there.

### Do Not Casually Rename IDs

Do not rename these without explicit need and full reference updates:

- Mod ID
- Package root
- Resource namespace
- Registry IDs
- Structure IDs
- Template pool IDs
- Processor list IDs
- Loot table IDs
- Tag IDs
- Config keys
- Generated output paths

Names are load-bearing. Random renames break saves, datapacks, registries, and JSON references.

### Do Not Rewrite Architecture for Style

Prefer the smallest correct change.

Good:

- Fix one bad namespace.
- Add one missing template pool entry.
- Correct one structure set reference.
- Add one registry object using the existing pattern.

Bad:

- Replacing the registry system.
- Moving packages because a tutorial used a different layout.
- Rewriting worldgen because one JSON path was wrong.
- Adding datagen for a one-line manual JSON fix unless requested.

Make the mod load. Make the structure generate. Keep the diff small.

---

## 3. First Move: Inspect Before Editing

Before editing, inspect the repo. No blind changes.

Minimum inspection:

```text
build.gradle
settings.gradle
gradle.properties
src/main/resources/META-INF/mods.toml
src/main/java/
src/main/resources/assets/
src/main/resources/data/
src/generated/resources/   if present
```

Find and confirm:

- Actual mod ID.
- Actual base Java package.
- Existing registry pattern.
- Existing worldgen JSON format.
- Existing namespace conventions.
- Whether datagen exists.
- Whether generated resources are checked in.
- Similar structure/template/pool examples.

Search before renaming anything:

```text
<modid>
<structure_name>
<registry_name>
<template_pool_name>
<processor_list_name>
<loot_table_name>
```

If context is missing, state the assumption and keep the change narrow.

---

## 4. Environment and Commands

Use the Gradle wrapper in this repo. Do not replace it.

### Check Java

Linux/macOS:

```bash
java -version
```

Windows PowerShell:

```powershell
java -version
```

Expected: 64-bit Java 17 JDK.

Do not upgrade to Java 21 unless explicitly requested.

### Check Gradle

Linux/macOS:

```bash
./gradlew --version
```

Windows PowerShell:

```powershell
.\gradlew.bat --version
```

### Compile

Linux/macOS:

```bash
./gradlew compileJava
```

Windows PowerShell:

```powershell
.\gradlew.bat compileJava
```

### Build Jar

Linux/macOS:

```bash
./gradlew build
```

Windows PowerShell:

```powershell
.\gradlew.bat build
```

Expected output:

```text
build/libs/<archive-name>.jar
```

A successful build proves packaging. It does **not** prove structures spawn.

### Run Client

Linux/macOS:

```bash
./gradlew runClient
```

Windows PowerShell:

```powershell
.\gradlew.bat runClient
```

Use for:

- Startup validation.
- Resource loading.
- Datapack loading.
- Singleplayer world tests.
- `/locate structure <modid>:<structure_name>`.
- Visual inspection.

### Run Server

Linux/macOS:

```bash
./gradlew runServer
```

Windows PowerShell:

```powershell
.\gradlew.bat runServer
```

Use for:

- Dedicated-server safety.
- Client-only class crashes.
- Registry sync issues.
- Datapack load errors.
- Server config behavior.

The first server run may require:

```text
run/eula.txt
eula=true
```

### Run Datagen

Only if datagen is configured or explicitly requested.

Linux/macOS:

```bash
./gradlew runData
```

Windows PowerShell:

```powershell
.\gradlew.bat runData
```

After datagen:

- Inspect generated diffs.
- Confirm namespaces.
- Confirm output paths.
- Confirm no unrelated files were regenerated.

### IntelliJ IDEA

Open the repository root as a Gradle project.

Use:

```bash
./gradlew genIntellijRuns
```

when run configs are missing.

If IntelliJ disagrees with Gradle, trust Gradle first. IntelliJ is useful, not sacred.

---

## 5. Expected Project Layout

Exact names may differ. Inspect first.

```text
.
├── build.gradle
├── settings.gradle
├── gradle.properties
├── gradlew
├── gradlew.bat
├── gradle/wrapper/
├── AGENTS.md
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── generated/
│       └── resources/
└── run/
```

### Java Source

```text
src/main/java/
```

Common package areas:

```text
<base/package>/<modid>/
├── <MainModClass>.java
├── registry/
├── config/
├── worldgen/
├── worldgen/structure/
├── datagen/
├── event/
├── util/
└── client/
```

Follow the existing package root. Do not invent a new one.

### Resources

```text
src/main/resources/
├── META-INF/
│   └── mods.toml
├── pack.mcmeta
├── assets/
│   └── <modid>/
└── data/
    └── <modid>/
```

Client assets go under `assets/`.

Gameplay/server datapack data goes under `data/`.

---

## 6. Resource and Worldgen Paths

Use lowercase snake_case for resource paths and file names.

Good:

```text
ancient_watchtower.json
small_ruin_01.nbt
buried_ruins.json
```

Bad:

```text
AncientWatchtower.json
ancientWatchtower.json
My Cool Structure.nbt
ancient-watchtower.json
```

### Assets

```text
src/main/resources/assets/<modid>/
```

Common paths:

```text
assets/<modid>/lang/en_us.json
assets/<modid>/textures/
assets/<modid>/models/
assets/<modid>/blockstates/
assets/<modid>/sounds.json
assets/<modid>/sounds/
```

### Server Data

```text
src/main/resources/data/<modid>/
```

Common structure-generation paths:

```text
data/<modid>/worldgen/structure/
data/<modid>/worldgen/structure_set/
data/<modid>/worldgen/template_pool/
data/<modid>/worldgen/processor_list/
data/<modid>/structures/
data/<modid>/loot_tables/
data/<modid>/tags/
```

Forge biome modifiers for Forge 1.20.1 are commonly under:

```text
data/<modid>/forge/biome_modifier/
```

If this repo already uses a different valid convention, follow the repo. Do not guess.

### Tags

Common biome tag paths:

```text
data/<modid>/tags/worldgen/biome/
data/minecraft/tags/worldgen/biome/
data/forge/tags/worldgen/biome/
```

Use tags for biome targeting. Do not hard-code giant biome lists unless the repo already does.

---

## 7. Structure Generation Pipeline

Most structure bugs are JSON/NBT/reference bugs, not Java bugs.

Typical pipeline:

```text
NBT structure template
        ↓
template_pool JSON
        ↓
structure JSON
        ↓
structure_set JSON
        ↓
biome tag / biome modifier / biome constraints
        ↓
world generation
```

Fix the correct layer. Do not blame Java because a pool ID is misspelled.

---

## 8. Structure Files

### Structure Template NBT

Path:

```text
data/<modid>/structures/<path>.nbt
```

Referenced as:

```text
<modid>:<path>
```

Do not include `.nbt` in references.

Example path:

```text
data/<modid>/structures/watchtower/start.nbt
```

Example reference:

```text
<modid>:watchtower/start
```

### Template Pools

Path:

```text
data/<modid>/worldgen/template_pool/<pool_name>.json
```

Template pools select and connect pieces.

Check carefully:

- `fallback`
- `elements`
- `weight`
- `element_type`
- `location`
- `processors`
- `projection`
- Jigsaw `name`
- Jigsaw `target`
- Jigsaw `target_pool`
- Jigsaw orientation

Common failures:

- Missing NBT template.
- Wrong namespace.
- Empty `elements`.
- Broken fallback pool.
- Bad processor list reference.
- Mismatched jigsaw names.
- `single_pool_element` vs `legacy_single_pool_element` behavior mismatch.
- `max_distance_from_center` too small for the piece graph.

### Structure JSON

Path:

```text
data/<modid>/worldgen/structure/<structure_name>.json
```

Common fields may include:

```text
type
biomes
step
terrain_adaptation
start_pool
size
start_height
project_start_to_heightmap
max_distance_from_center
use_expansion_hack
```

The exact format is structure-type specific. Match existing 1.20.1 files in this repo before adding new fields.

### Structure Sets

Path:

```text
data/<modid>/worldgen/structure_set/<structure_set_name>.json
```

Structure sets control placement attempts.

Check:

- Structure ID reference.
- `spacing`
- `separation`
- `salt`
- `frequency`
- `frequency_reduction_method`
- Dimension assumptions.
- Duplicate structure sets.

Rules:

- `separation` must be lower than `spacing`.
- Do not copy salts blindly.
- A structure JSON that is not referenced by a structure set may load but never generate naturally.

### Processor Lists

Path:

```text
data/<modid>/worldgen/processor_list/<processor_name>.json
```

Use processors for:

- Ignoring structure blocks.
- Handling structure void behavior.
- Protected block rules.
- Random decay.
- Rule-based block replacement.

Common failures:

- Wrong processor type.
- Bad block ID.
- Processor order causing unwanted replacement.
- Accidentally replacing air.
- Accidentally destroying loot containers.
- Processor list path typo.

### Loot Tables

Path:

```text
data/<modid>/loot_tables/
```

Example file:

```text
data/<modid>/loot_tables/chests/<loot_table_name>.json
```

Referenced in NBT as:

```text
<modid>:chests/<loot_table_name>
```

Do not include `loot_tables` or `.json` in the loot table ID.

---

## 9. Java vs JSON: Pick the Right Tool

### Edit Java When Changing

- Custom `Structure` classes.
- Custom `StructureType`.
- Codecs.
- Forge registry entries.
- Config-controlled generation behavior.
- Datagen providers.
- Event handlers.
- Logging.
- Compatibility hooks.

### Edit JSON/NBT When Changing

- Structure definitions.
- Structure sets.
- Template pools.
- Processor lists.
- Biome tags.
- Biome modifiers.
- Loot tables.
- Template references.
- Spawn spacing/separation/salt.
- Terrain adaptation.
- Jigsaw pool layout.

Do not add Java for a JSON problem. That is how simple bugs become cursed machinery.

---

## 10. Registries and Namespaces

Use the repository’s existing registry pattern.

If the repo uses `DeferredRegister`, keep using it.

Namespace consistency must hold across:

```text
@Mod("<modid>")
mods.toml
gradle.properties
build.gradle
ResourceLocation
DeferredRegister
assets/<modid>/
data/<modid>/
loot table IDs
structure IDs
template pool IDs
processor list IDs
NBT template references
language keys
```

Use `ResourceLocation` consistently:

```java
new ResourceLocation(MOD_ID, "some_id")
```

Do not hard-code the mod ID repeatedly if a `MOD_ID` constant exists.

Registry/resource IDs should be lowercase snake_case:

```text
ancient_watchtower
small_ruin
overgrown_gatehouse
```

Not:

```text
AncientWatchtower
ancientWatchtower
ancient-watchtower
ancient watchtower
```

---

## 11. Generated Data and Datagen

Generated data usually belongs under:

```text
src/generated/resources/
```

Generated layout mirrors main resources:

```text
src/generated/resources/assets/<modid>/
src/generated/resources/data/<modid>/
```

Rules:

- Datagen provider code is source.
- Generated JSON is output.
- If output is checked into git, keep diffs clean.
- Do not hand-edit generated files when the provider should be fixed.
- If generated and hand-written files conflict, stop and inspect the build config.

If datagen exists:

- Register providers in the existing datagen event handler.
- Use existing helper classes.
- Keep generated IDs stable.
- Use `ExistingFileHelper` where the project pattern does.
- Generate into the repo’s configured output folder.
- Do not duplicate hand-written files.

If datagen does not exist, do not introduce it for a trivial JSON fix unless asked.

---

## 12. Config Rules

Use the existing Forge config pattern.

Expected style:

- `ForgeConfigSpec`
- Common/server config for worldgen behavior
- Clear defaults
- Numeric ranges
- Useful comments for user-facing settings

Do not put server-side structure spawning behind client config.

Good config categories:

```text
generation
structures
debug
```

Keep options understandable. Users should not need to reverse-engineer Java to tune spawn rates.

---

## 13. Logging and Error Handling

Use the repository’s existing logger.

Common pattern:

```java
private static final Logger LOGGER = LogUtils.getLogger();
```

Logging rules:

- `debug`: noisy diagnostics.
- `info`: important setup milestones only.
- `warn`: recoverable bad config or missing optional data.
- `error`: broken state requiring action.

Do not spam logs during chunk generation.

Never do this:

```java
try {
    // worldgen
} catch (Exception ignored) {
}
```

That is not error handling. That is hiding the body.

If catching broadly is unavoidable, log useful context and rethrow when appropriate.

---

## 14. Validation Workflow

### Minimum Build Check

```bash
./gradlew compileJava
```

Windows:

```powershell
.\gradlew.bat compileJava
```

### Better Check

```bash
./gradlew build
```

Windows:

```powershell
.\gradlew.bat build
```

### Worldgen Behavior Check

Use a fresh test world when validating generation.

Commands:

```mcfunction
/locate structure <modid>:<structure_name>
```

```mcfunction
/place structure <modid>:<structure_name>
```

```mcfunction
/place template <modid>:<template_path>
```

Use `/place template` to validate NBT/template paths.

Use `/place structure` to validate configured structure loading.

Use `/locate structure` to validate natural placement, biome targeting, and structure sets.

If `/place structure` works but `/locate` fails, the structure exists; natural generation is the problem.

Check:

- Structure set exists.
- Structure set references the structure.
- Biome tag includes the tested biome.
- Spacing/separation are valid.
- Dimension assumptions are correct.
- New chunks are being generated.
- Datapack loaded without errors.

---

## 15. Logs to Check

Client/server logs:

```text
run/logs/latest.log
run/logs/debug.log
```

Crash reports:

```text
run/crash-reports/
```

Build failures:

```text
Terminal Gradle output
IntelliJ Build tool window
IntelliJ Event Log
```

Read the first meaningful error. The last stacktrace line is usually just the corpse tag.

Common signals:

```text
Failed to load datapacks
Failed to parse
Couldn't parse element
Unknown registry key
Unknown structure type
Unknown template pool
Unknown processor list
Missing required element
Invalid or unknown ResourceLocation
Duplicate registration
Registry Object not present
ClassNotFoundException
NoClassDefFoundError
```

---

## 16. Debugging Structure Problems

### Structure Missing or Never Spawns

Likely causes:

- No structure set references it.
- Bad biome tag.
- Wrong namespace.
- JSON parse failure.
- Old chunks already generated.
- Spacing too large.
- Dimension mismatch.
- Invalid terrain/height constraints.

Fix strategy:

1. Check logs.
2. Test in a fresh world.
3. Confirm structure JSON loads.
4. Confirm structure set references it.
5. Confirm biome tag includes the test biome.
6. Temporarily reduce spacing for testing.
7. Use `/locate structure`.

### Template Pool Does Nothing

Likely causes:

- Empty pool.
- Bad fallback.
- Missing template.
- Jigsaw `name`/`target` mismatch.
- Jigsaw orientation mismatch.
- Pool element cannot fit.
- Distance limit too small.
- Processor removes required blocks.

Fix strategy:

1. Place the template manually.
2. Simplify the pool to one known-good piece.
3. Remove processors temporarily.
4. Check jigsaw names, targets, pools, and orientation.
5. Restore complexity only after the simple case works.

### Structure Spawns Too Often

Likely causes:

- `spacing` too low.
- `separation` too low.
- Too many biomes targeted.
- Duplicate structure sets.
- Multiple datapack files referencing the same structure.

Fix strategy:

1. Increase spacing.
2. Narrow biome tags.
3. Search for duplicate structure IDs.
4. Check both generated and hand-written resources.

### Loot Missing

Likely causes:

- Chest NBT has no loot table.
- Loot table path typo.
- Wrong namespace.
- Loot table JSON parse error.
- Processor replaced the chest.
- NBT was saved incorrectly.

Fix strategy:

1. Inspect the NBT template.
2. Confirm loot table path under `data/<modid>/loot_tables/`.
3. Confirm ID format: `<modid>:chests/<name>`.
4. Check datapack load logs.

### Dedicated Server Crash

Likely causes:

- Client-only class used in common/server code.
- Client event registered on common path.
- Rendering/model code loaded on server.
- Wrong side assumptions.

Fix strategy:

1. Run `runServer`.
2. Find the first `ClassNotFoundException` or `NoClassDefFoundError`.
3. Move client-only code behind proper client-only setup.
4. Keep common worldgen code server-safe.

---

## 17. Style Rules

### Java

Use boring, correct Java.

- 4 spaces.
- Braces on the same line.
- No wildcard imports.
- No unused imports.
- Clear names.
- Small methods.
- `private static final` for repeated constants.
- Avoid reflection.
- Avoid global mutable state unless the repo already uses it safely.

### Packages

Packages must be lowercase.

Good:

```text
com.example.examplemod.worldgen.structure
```

Bad:

```text
com.example.ExampleMod.WorldGen.Structure
```

Follow the existing root package.

### JSON

- Preserve existing formatting style.
- Use valid JSON.
- No trailing commas.
- No comments.
- Keep IDs namespaced where required.
- Use tags instead of giant biome lists when practical.

### Comments

Add comments only when they prevent real confusion.

Do not narrate obvious code.

Good:

```java
// Keep this salt stable; changing it moves existing structure placement.
```

Bad:

```java
// Create variable.
```

---

## 18. Safe-Edit Checklist

Before editing:

- [ ] Read relevant Java classes.
- [ ] Read similar JSON/NBT resources.
- [ ] Identify actual mod ID.
- [ ] Identify actual package root.
- [ ] Search all references to the target ID.
- [ ] Determine the correct layer: Java, JSON, NBT, config, or datagen.
- [ ] Decide the smallest safe change.

During editing:

- [ ] Preserve existing style.
- [ ] Preserve namespace consistency.
- [ ] Use lowercase snake_case IDs.
- [ ] Do not mix loader APIs.
- [ ] Do not upgrade versions.
- [ ] Do not rename IDs casually.
- [ ] Do not hand-edit generated output when provider code owns it.

After editing:

- [ ] Review the diff.
- [ ] Check changed paths.
- [ ] Check changed IDs.
- [ ] Check JSON syntax.
- [ ] Check imports.
- [ ] Run the narrowest useful Gradle verification.
- [ ] For worldgen changes, test in-game or state why it was not run.

---

## 19. Final Response Requirements for Agents

When reporting changes, include:

- Files changed.
- What changed.
- Why it changed.
- How it was verified.
- Any remaining risk.

Good:

```text
Changed the watchtower structure set spacing from 48 to 32 for testing.
Verified with /locate in a fresh world.
Risk: tune spacing before release if the structure is too common.
```

Bad:

```text
Improved worldgen.
```

That tells future agents nothing. Do better.

---

## 20. Final Rule

This repo is not a playground for framework tourism.

Use Forge 1.20.1.  
Use Java 17.  
Inspect before editing.  
Patch the broken layer.  
Keep IDs stable.  
Keep generated data clean.  
Make the mod load.  
Make the structure generate.
