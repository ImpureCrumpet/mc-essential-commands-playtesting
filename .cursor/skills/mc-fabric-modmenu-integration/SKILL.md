---
name: mc-fabric-modmenu-integration
description: "Ensures Fabric mods present Mod Menu metadata and config screens correctly: fabric.mod.json custom.modmenu block, translation keys, ModMenuApi entrypoints, Cloth Config and MidnightLib wiring, and multi-version Gradle dependency conventions."
triggers:
  - mod menu
  - modmenu
  - mod config screen
  - configure button
  - Edit Config
  - ModMenuApi
  - ConfigScreenFactory
  - modmenu entrypoint
  - custom.modmenu
  - modmenu.descriptionTranslation
  - cloth config mod menu
  - midnightconfig
  - midnightlib
  - midnightconfig tooltip
  - mod menu integration
  - mod menu metadata
  - mod menu badge
  - library badge modmenu
dependencies: []
version: "1.1.7"
---

# Fabric Mod Menu integration

**Source:** [TerraformersMC/ModMenu developer guide](https://github.com/TerraformersMC/ModMenu#developers) (Fabric Metadata API, Translation API, Java API).

**Scope:** Client-side presentation and config-button wiring. Mod Menu is **client-only** — never ship `modmenu` JARs on dedicated servers; mark API/library modules with the `library` badge when they are not end-user mods.

## 1. Route the work

| Goal | Mechanism | Java required? |
|---|---|---|
| Better name / summary / description in the list | Translation API keys | No |
| Links, badges, parent grouping, update-checker opt-out | `fabric.mod.json` → `custom.modmenu` | No |
| **Configure…** button opens your settings | `ModMenuApi` + `entrypoints.modmenu` | Yes |
| Settings built with Cloth Config | `ModMenuApi.getModConfigScreenFactory` returning a Cloth `Screen` | Yes |
| Settings via `MidnightConfig` | `MidnightConfig.init(modid, Config.class)` — MidnightLib registers the button | Init only |
| API module that should stay hidden | `"badges": ["library"]` + optional `"environment": "client"` | Metadata only |

If the mod has **no** user-facing config, skip the Java API — still add metadata when the mod is client-facing or split into modules.

## 2. Presentation — `fabric.mod.json` metadata

Add a `custom.modmenu` block (Quilt: top-level `"modmenu"` instead of under `custom`).

```json
"custom": {
  "modmenu": {
    "links": {
      "modmenu.discord": "https://discord.gg/example",
      "modmenu.modrinth": "https://modrinth.com/mod/your-mod"
    },
    "badges": ["library"],
    "parent": "parent-mod-id",
    "update_checker": true
  }
}
```

When configuring links, badges, parent grouping, or update checks, load [references/metadata-options.md](references/metadata-options.md). Keep the block in the mod container whose list entry it describes.

## 3. Presentation — Translation API

Add keys to `assets/<namespace>/lang/en_us.json` (and other locales):

```json
{
  "modmenu.nameTranslation.mymod": "My Mod",
  "modmenu.summaryTranslation.mymod": "One-line summary for the list.",
  "modmenu.descriptionTranslation.mymod": "Longer description shown in the detail pane."
}
```

Replace `mymod` with your mod id. Summary can differ from description; if they match, summary is optional.

## 4. Config screens — Gradle dependency

Before wiring a config screen, load [references/gradle-dependency.md](references/gradle-dependency.md). Add Mod Menu only as a compile/dev dependency, pin the version for each Minecraft line, and keep the runtime JAR client-only.

## 5. Config screens — `ModMenuApi` entrypoint

1. Implement `com.terraformersmc.modmenu.api.ModMenuApi` in a **client** class.
2. Register it under `entrypoints.modmenu` in the **same** `fabric.mod.json` as the mod id that should get the button.

```json
"entrypoints": {
  "modmenu": ["com.example.mymod.MymodModMenu"]
}
```

Mod Menu resolves the **mod id from the entrypoint’s mod container** — the `modmenu` entrypoint must live in the JAR whose id should show **Configure…**, not in a shared root-only stub unless that stub is the container.

```java
public class MymodModMenu implements ModMenuApi {
  @Override
  public ConfigScreenFactory<?> getModConfigScreenFactory() {
    if (!FabricLoader.getInstance().isModLoaded("cloth-config")
        && !FabricLoader.getInstance().isModLoaded("cloth-config2")) {
      return new NullScreenFactory<>(); // no Configure button; avoids "broken ModMenuApi" log
    }
    // Defer Cloth classloading — see "Eager Cloth loading" below.
    return MymodModMenu::createClothScreen;
  }

  private static Screen createClothScreen(Screen parent) {
    // Cloth-dependent code stays behind this same-class helper.
    return AutoConfig.getConfigScreen(MyConfig.class, parent).get();
  }
}
```

`ConfigScreenFactory` is `Screen create(Screen parent)` — always accept `parent` and pass it when constructing sub-screens so **Done** returns to Mod Menu.

### Eager Cloth loading hides Configure (especially noticed on 26.x)

Mod Menu loads every `modmenu` entrypoint during its `ClientModInitializer` and calls `getModConfigScreenFactory()` inside a try/catch. On failure it logs `Mod {} provides a broken implementation of ModMenuApi` and **never registers** a Configure button.

A **method reference or lambda that directly targets a Cloth Config screen class** (e.g. `MymodClothScreens::create`, or `parent -> AutoConfig.getConfigScreen(...).get()` when that path class-initializes Cloth) links/loads Cloth-dependent classes **during that init call**. If the player installed Mod Menu but not Cloth Config — common on **26.x** where Mod Menu `20.x` and Cloth `26.2.x` are separate Modrinth downloads — registration fails and Configure is missing while the mod still appears in the list.

**Required pattern:**

1. Check `FabricLoader.isModLoaded("cloth-config")` (and `"cloth-config2"` for older provides) before registering a real factory; otherwise return `NullScreenFactory`.
2. Return a method reference to a **helper on your ModMenuApi class** (or use reflection) so Cloth types load only when Configure is clicked — not while Mod Menu enumerates entrypoints.
3. Document for players: **Mod Menu + Cloth Config**, matching the Minecraft line (26.2 ≠ 26.1 Mod Menu builds).
4. Prefer `"suggests": { "modmenu": "*", "cloth-config": "*" }` in `fabric.mod.json`.

Validated on mc-herd-hysteria: 1.21.11 Configure worked with both mods installed; 26.2 showed the mod without Configure until Cloth was present and factory registration no longer eagerly loaded Cloth.

### Do not use `getProvidedConfigScreenFactories` for your own mod

That map is for **libraries** (e.g. Cloth Config, **MidnightLib**) registering screens for **other** mod ids. Your mod’s button comes from `getModConfigScreenFactory` on **your** entrypoint — or, for MidnightLib, from MidnightLib’s own provider after `MidnightConfig.init(...)`. Do **not** add a custom `ModMenuApi` just to re-register your MidnightConfig screen.

## 6. Cloth Config path

When settings use Cloth Config, load [references/cloth-config.md](references/cloth-config.md) before implementing the screen. It preserves the guarded same-class helper from §5, dependency/source-set rules, and the compact-screen tooltip pattern.

## 6b. MidnightLib path

When settings extend `MidnightConfig`, load [references/midnightlib.md](references/midnightlib.md) before changing initialization or translations. MidnightLib owns the Mod Menu provider; do not add a redundant `ModMenuApi` entrypoint.

## 7. Multi-version subprojects

- **Metadata** (`custom.modmenu`, lang keys): usually identical in every subproject’s `fabric.mod.json`; duplicate or merge from root `src/main/resources` per your repo’s resource strategy.
- **Java API**: client-only; under `src/client/java` (or per-subproject client sources). Do not register `modmenu` entrypoints on server-only artifacts.
- **Dependency coordinate style:** `implementation` on **26.x**; `modImplementation` / `modCompileOnly` on **1.21.x** Yarn lines (see §4).
- **Mappings:** Mod Menu artifacts are named for the game version — pin **per subproject** `modmenu_version` in each `gradle.properties` or version catalog row.

## 8. Verification checklist

Run in a **client** dev env with Mod Menu on the runtime classpath (or in `mods/`):

1. Open Mod Menu → find the mod by name; search **configurable** filter should list it when a config factory is registered.
2. Select the mod → description, links, badges, and credits look correct (not raw translation keys).
3. **Configure…** appears when a config factory is registered (your `ModMenuApi`, Cloth Config, or MidnightLib after `MidnightConfig.init`); click opens your screen; **Done** returns to Mod Menu without error.
4. If using Cloth Config, confirm `AutoConfig` / builder matches the serialized config file on disk after save; on compact screens confirm enum/dropdown **reset** is clickable (no entry tooltip covering controls — §6).
5. With Mod Menu installed but **without** Cloth Config: Configure must stay hidden (`NullScreenFactory`) and logs must **not** show `broken implementation of ModMenuApi` for your mod (§5).
6. With Mod Menu **and** Cloth Config for the same MC line: Configure opens your screen; Done returns to Mod Menu.
7. If using MidnightLib, confirm every `@Entry` field has a matching `<fieldName>` label key; tooltips use `.label.tooltip` and `.tooltip` as needed (§6b).
8. Library/API modules: hidden by default with `library` badge; visible when **Libraries → Shown**.
9. Multi-module: children nest under the parent; dummy parent shows once with shared icon/description.
10. Dedicated-server pack audit: `modmenu` not in server `mods/` (client-only).

### Common failures

When any verification step fails, load [references/troubleshooting.md](references/troubleshooting.md). It maps symptoms to the registration, translation, eager-loading, packaging, or version error to fix. Load its optional-API section only when the default metadata/config mechanisms cannot satisfy the request.

When changing or evaluating this skill's description or triggers, use [references/trigger-evals.json](references/trigger-evals.json); do not load it during ordinary Mod Menu work.
