---
name: mc-fabric-mod-version-bump
description: "Bumps Fabric mod SemVer during development whenever playtest JARs are rebuilt: gradle.properties mod_version, every fabric.mod.json version field, artifact filenames, then rebuild."
triggers:
  - version bump
  - bump version
  - mod_version
  - bump jars
  - rebuild jars
  - playtest jar
  - patch version
  - fabric.mod.json version
  - archiveVersion
  - mc-fabric-mod-version-bump
dependencies:
  - mc-gradle-daemon-hygiene
version: "1.1.1"
---

# Fabric mod version bump (during development)

**Scope:** Multi-`mc*` Fabric mod repos that ship one SemVer (`mod_version`) across all Minecraft lines. Aligns with **`.cursor/fabric-mod-build-release-guide-v4.3.md`** §5 JAR naming and § Step 3 versioning.

**Rule (verbatim intent):** **Jars should be version bumped during development.** Do not keep rebuilding `1.0.0` / stale SemVer playtest artifacts after behavior changes — bump **before** (or as part of) the next full JAR rebuild so Mod Menu, logs, and `mods/` folders show a distinct build.

## Routing — not these skills

| User intent | This skill | Route elsewhere |
|-------------|------------|-----------------|
| Bump **`mod_version`** + sync **`fabric.mod.json`** before playtest/release JAR rebuild | **`mc-fabric-mod-version-bump`** | — |
| Rename a mod/artifact/namespace or replace its JAR across client and server | — | **`mc-fabric-rename-and-jar-rollout`** |
| Upload JARs to a storefront; **`version_number`** with `+mc` suffix | — | Out of scope for this skill. **Do not** PATCH-bump mod semver for a straight 26.x port of unchanged behavior |
| Add a new **`mc*`** Gradle subproject / MC line | — | **`mc-scaffold-new-fabric-subproject`** |
| JDK switch, daemon stop, heap before **`./gradlew`** | — | **`mc-gradle-daemon-hygiene`** (run **after** bump when rebuilding) |
| Bump **skills-harness kit** semver | — | **`kit-release`** |
| Update an external registry or sibling repo | — | Out of scope for this skill |

**`version_number` vs `mod_version`:** Modrinth uses **`{modSemVer}+{minecraft}`** per MC line (`2.0.0+26.2`). This skill bumps the **mod SemVer** (`mod_version`) shared across all lines — not the Minecraft patch in the `+` suffix.

## When to use

- User asks to **bump version**, **bump jars**, or **rebuild all jars** after a fix/feature.
- You are about to produce playtest/release JARs and `mod_version` still matches the **previous** shipped or playtested build.
- After a merged fix that players will drop into `mods/` alongside an older JAR with the same filename.

## When **not** to bump

- Pure docs/skill-only edits with **no** JAR rebuild requested.
- A filename/display/repository rename with no changed mod bits — use
  **`mc-fabric-rename-and-jar-rollout`** to preserve identity and replace stale filenames; bump only
  when the artifact contents change.
- User explicitly says keep the current version (e.g. rebuild same `1.1.0` for a clean tree).
- **Straight 26.x port** of existing behavior with no mod release change: keep mod semver. Storefront `+26.2` lines are a publish step, not this skill.
- Tagging/publishing only — still bump if the artifact **content** changed since the last SemVer.

## SemVer guidance (development)

| Change | Bump |
|--------|------|
| Bugfix, Mod Menu/config fix, mixin fix, playtest iteration | **PATCH** (`1.1.0` → `1.1.1`) |
| New player-facing feature (new intensity tier, new command, etc.) | **MINOR** (`1.1.1` → `1.2.0`) |
| Breaking config/command/behavior for existing worlds | **MAJOR** |

Default when unsure and the user said “bump”: **PATCH**.

## Checklist (do all that apply)

1. **Root `gradle.properties`:** set `mod_version=X.Y.Z` (single source of truth for Gradle `project.version` / `archiveVersion`).
2. **Every `mc*/src/main/resources/fabric.mod.json`:** set `"version": "X.Y.Z"` to match. Many Fabric repos hardcode the field (not `${version}` expansion). If the project uses Loom `processResources` expand of `${version}`, bump properties only and verify the built JAR’s `fabric.mod.json`.
3. **Do not** leave old `build/libs/*-oldversion+*.jar` lying next to new ones when handing paths to the user — `clean` then rebuild, or delete stale versioned artifacts.
4. **Rebuild** the requested matrix (`remapJar` on 1.21.x, `jar` on 26.x, `-x test` if empty gametest tasks fail `:build`). Follow **`mc-gradle-daemon-hygiene`** (`j21` / `j25`, `./gradlew --stop` after JDK switch).
5. **Confirm filenames:** `{slug}-{mod_version}+{minecraft}.jar` (e.g. `example-mod-1.1.1+26.2.jar`).
6. **Commit** the version bump with the related fix when shipping to `main` (unless user said not to commit).
7. **Publish** is out of scope. After bump, stop at rebuilt JARs unless the user asked for a storefront upload in this repo.

## Anti-patterns

- Rebuilding playtest JARs without bumping after a user-visible fix → two different bits with the same `1.1.0+26.2` name.
- Bumping only `gradle.properties` while `fabric.mod.json` still shows the old version in Mod Menu.
- Bumping only one `mc*` module’s `fabric.mod.json`.
- Using Minecraft version (`26.2`) as `mod_version`.
- PATCH-bumping mod semver solely because a new MC line (`+26.2`) was added without behavior change.
- Treating a SemVer bump as a migration for a changed Fabric mod ID or registered namespace; route
  that work through **`mc-fabric-rename-and-jar-rollout`**.

## Examples

- "bump all jars to 1.1.1" → set `mod_version` + all `fabric.mod.json` → clean rebuild matrix → list new paths.
- "26.2 Mod Menu fix works, continue playtesting, bump jars" → PATCH bump, rebuild at least the playtest anchors (or full matrix if asked).
- "rebuild jars" after a fix with no version mentioned → **ask or PATCH-bump** per this skill; prefer bumping over silent same-version rebuilds when artifacts will be playtested.
- "add 26.2 to the storefront" with unchanged mod release → skip this skill; **no** `mod_version` bump unless the mod itself changed.
