# MidnightLib path

Load this reference only when settings extend `MidnightConfig` from [MidnightLib](https://modrinth.com/mod/midnightlib), whether bundled or declared as a dependency.

No `ModMenuApi` entrypoint is required. MidnightLib registers **Configure…** through `getProvidedConfigScreenFactories` for mods that call `MidnightConfig.init(...)` during mod initialization.

```java
MidnightConfig.init(MODID, MyModConfig.class);
```

`MyModConfig` extends `MidnightConfig` with `@Entry` fields and optional `@Comment` intros. `@Condition` is available on 1.9.x and later.

## Translation keys

Use the `<modid>.midnightconfig.` prefix, for example `guardvillagers.midnightconfig.`.

| Key | Used for |
|---|---|
| `<modid>.midnightconfig.title` | Screen title |
| `<modid>.midnightconfig.category.<name>` | Tab label (`@Entry(category = "…")`) |
| `<modid>.midnightconfig.<fieldName>` | Option label; it **must match** the `@Entry` / `@Comment` Java field name |
| `<modid>.midnightconfig.<fieldName>.label.tooltip` | Hover on the **label** (`EntryInfo.getTooltip(false)`) |
| `<modid>.midnightconfig.<fieldName>.tooltip` | Hover on the **control** such as a toggle, slider, or text field (`getTooltip(true)`) |

`EntryInfo.getTooltip(boolean isButton)` builds the key as `translationKey + (isButton ? "" : ".label") + ".tooltip"`. Missing keys show no tooltip. Labels do **not** fall back to `.tooltip` when `.label.tooltip` is absent, so set both keys when the label and control should share the same help text.

`@Comment` intro lines use the field name as the translation key, for example `guardsIntro` becomes `guardvillagers.midnightconfig.guardsIntro`.

```json
{
  "mymod.midnightconfig.title": "My Mod",
  "mymod.midnightconfig.category.general": "General",
  "mymod.midnightconfig.generalIntro": "Short intro shown at the top of the tab.",
  "mymod.midnightconfig.enableFeature": "Enable Feature",
  "mymod.midnightconfig.enableFeature.label.tooltip": "Shown when hovering the option name.",
  "mymod.midnightconfig.enableFeature.tooltip": "Shown when hovering the toggle."
}
```

## Version notes

| Feature | 1.5.7 (1.21.x) | 1.9.x (26.x) |
|---|---|---|
| `@Comment` tab intros | yes | yes |
| `.label.tooltip` / `.tooltip` split | yes (`MidnightConfig$EntryInfo`) | yes (`EntryInfo`) |
| `@Condition` (show/hide fields) | **no** | yes |

Source: MidnightLib `EntryInfo.getTooltip()` in 1.9.x and `MidnightConfig$EntryInfo` in 1.5.7. This was validated on mc-guardvillagers with 1.5.7 on 1.21.11 and 1.9.x on 26.2.
