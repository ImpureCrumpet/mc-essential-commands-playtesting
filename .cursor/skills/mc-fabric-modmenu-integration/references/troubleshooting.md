# Troubleshooting and optional APIs

Load the symptom table only after a verification step fails.

| Symptom | Likely cause |
|---|---|
| No **Configure…** button | Missing `entrypoints.modmenu`, factory returns null, entrypoint is in the wrong mod container, or `MidnightConfig.init` was not called |
| Raw `mymod.midnightconfig.fieldName` in config UI | Language key missing or Java `@Entry` field name does not match the key suffix |
| Label has no tooltip but control does | Only `.tooltip` is set; labels require `.label.tooltip` with no fallback |
| `Failed to load config screen for '…'` | Factory throws; fix the logged stack trace. Mod Menu reports the **target mod id**, not Mod Menu itself |
| Raw `modmenu.descriptionTranslation.foo` in UI | Language entry missing for that mod id |
| Config works in dev, not for players | `modmenu` entrypoint class missing from the release JAR because client source-set/split packaging is not wired |
| Button on wrong mod | `modmenu` entrypoint registered in another module's `fabric.mod.json` |
| Tooltip covers enum selector/reset | Entry `.setTooltip()` on an enum/dropdown; use `ConfigCategory.setDescription()` instead |
| Mod listed but no Configure, especially on 26.x | Cloth Config missing, or `getModConfigScreenFactory` eagerly loads Cloth through `…::create`; Mod Menu logs a broken `ModMenuApi` |
| Configure works on 1.21.x, not 26.x | Mod Menu is present but Cloth `26.2.x` is absent, or the 26.1 Mod Menu build is installed on 26.2 |

## Optional APIs

Load this section only when the default metadata and config mechanisms cannot satisfy the request.

- `getUpdateChecker()` overrides Modrinth hash checking for the mod.
- `getProvidedConfigScreenFactories()` / `getProvidedUpdateCheckers()` are for **library mods only**.
- `attachModpackBadges(Consumer<String>)` marks bundled mods with the Modpack badge.
- `ModMenuApi.createModsScreen(parent)` / `createModsButtonText()` embeds Mod Menu in another UI.
