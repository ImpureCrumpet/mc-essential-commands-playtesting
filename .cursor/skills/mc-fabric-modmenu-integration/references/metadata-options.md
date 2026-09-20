# Mod Menu metadata options

Load this reference when configuring links, badges, parent grouping, or update checks in `custom.modmenu`.

## Badges

| Value | When to use |
|---|---|
| *(auto)* **Client** | Set `"environment": "client"` on the mod container |
| `library` | Pure dependency / API; hidden unless the user enables libraries |
| `deprecated` | Legacy shim retained for compatibility |

Mod Menu does not support custom badge strings.

## Links

- Keys are translation keys. Link text comes from the mod language file or Mod Menu defaults such as `modmenu.discord`, `modmenu.modrinth`, and `modmenu.github_releases`; see Mod Menu's `en_us.json` under `src/main/resources/` on its [26.3 branch](https://github.com/TerraformersMC/ModMenu/tree/26.3/src/main/resources).
- Use the mod's namespace for custom link keys. Use `modmenu.*` only when the default Mod Menu label is intended.
- A `sources` contact in standard `fabric.mod.json` metadata is also shown as a link.

## Parents

Use `"parent": "flamingo"` to group a child under an installed mod with that id. When the parent is not a real mod, repeat the same dummy metadata in **every** child `fabric.mod.json`:

```json
"parent": {
  "id": "mymod-suite",
  "name": "My Mod Suite",
  "description": "Grouped modules",
  "icon": "icon.png",
  "badges": ["library"]
}
```

`icon` is a JAR resource path, typically `assets/<modid>/icon.png`.

## Update checker

Mod Menu hashes the JAR and checks Modrinth by default. Set `"update_checker": false` for development builds, unpublished mods, or a custom checker supplied through `ModMenuApi.getUpdateChecker()`.
