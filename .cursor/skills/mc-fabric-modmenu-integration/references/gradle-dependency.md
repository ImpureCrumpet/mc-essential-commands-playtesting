# Gradle dependency

Load this reference before wiring a Mod Menu config screen.

Add the Terraformers Maven repository and Mod Menu as a compile/development dependency. Mod Menu is optional for players at runtime but required to test the button.

```gradle
repositories {
  maven { name = "Terraformers"; url = "https://maven.terraformersmc.com/" }
}

dependencies {
  // 26.x (Mojmap, no remapJar): plain implementation
  implementation("com.terraformersmc:modmenu:${project.modmenu_version}")

  // ≤ 1.21.11 (Yarn): use modImplementation or modCompileOnly
  // modCompileOnly("com.terraformersmc:modmenu:${project.modmenu_version}")
}
```

Pin `modmenu_version` in `gradle.properties`. Match the Mod Menu release to the Minecraft line through [Modrinth versions](https://modrinth.com/mod/modmenu/versions) or [GitHub releases](https://github.com/TerraformersMC/ModMenu/releases).

| Minecraft line | Typical Mod Menu release (verify before pin) |
|---|---|
| 26.2 | e.g. `20.0.1` |
| 1.21.11 | e.g. `17.x` |

Use `modCompileOnly` when only the compile-time API is needed and testing uses a manually installed Mod Menu JAR.
