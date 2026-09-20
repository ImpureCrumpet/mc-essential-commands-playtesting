# Cloth Config path

Load this reference only when settings use [Cloth Config](https://shedaniel.gitbook.io/cloth-config/).

1. Build the screen with `ClothConfigScreenBuilder` / `AutoConfig` for the selected Cloth Config version.
2. Keep the guarded factory and same-class `createClothScreen` helper from the parent skill's §5. Put the `AutoConfig.getConfigScreen(...)` or `ConfigBuilder` call **inside that helper**; do not inline it as `parent -> AutoConfig...` or point the returned method reference directly at a Cloth-dependent class.
3. Keep Cloth Config on `modApi` / `modImplementation` as its documentation requires; Mod Menu stays `implementation` / `modCompileOnly`.
4. If the config class or screen builder is version-specific, place the `ModMenuApi` class in the **client** source set of each `mc*` subproject, following other client entrypoints.

## Tooltips on compact screens

On screens with **one or two controls** such as an enum selector, dropdown, or single toggle, `.setTooltip()` on the entry can render a hover panel that covers the selector and reset button. This was validated on mc-herd-hysteria for 1.21.11 and 26.2 with a panic-intensity enum and reset control.

Prefer static help through `ConfigCategory.setDescription()` and omit entry `.setTooltip()` unless the row has ample space, such as multi-line forms or wide toggles with the label in a separate column.

```java
ConfigCategory category = builder.getOrCreateCategory(
    Component.translatable("text.mymod.config.category.general"));
category.setDescription(new FormattedText[] {
    Component.translatable("text.mymod.config.myOption.description")
});

category.addEntry(builder.entryBuilder()
    .startEnumSelector(
        Component.translatable("text.mymod.config.myOption"),
        MyEnum.class,
        currentValue)
    .setDefaultValue(MyEnum.DEFAULT)
    .setSaveConsumer(value -> { /* persist */ })
    .build()); // no setTooltip on enum/dropdown rows
```

Use a `…description` language key for the category blurb, such as `text.mymod.config.myOption.description`, rather than `…tooltip` on the entry. MidnightLib's `.label.tooltip` / `.tooltip` split does **not** apply to Cloth `ConfigEntryBuilder`; category description is Cloth's equivalent of a tab intro.

Entry tooltips remain suitable for sliders, text fields, or dense forms where the tooltip targets the label column and does not overlap the control column. If playtesting shows overlap, move help text to `setDescription()` or shorten the tooltip.
