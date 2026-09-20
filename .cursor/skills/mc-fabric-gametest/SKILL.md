---
name: mc-fabric-gametest
description: "Sets up Fabric server and client game tests with Loom configureTests, fabric-gametest entrypoints, smoke @GameTest classes, optional client game tests, and CI hooks."
triggers:
  - fabric gametest
  - game test fabric
  - FabricGameTestHelper
  - configureTests
  - fabric-gametest
  - fabric-client-gametest
  - runGametest
  - runClientGameTest
  - GameTestHelper
  - "@GameTest"
  - client game test
  - mc-fabric-gametest
dependencies:
  - mc-gradle-daemon-hygiene
version: "1.1.4"
---

# Fabric game tests (server + client)

**Source:** [Fabric Documentation — Automated Testing](https://docs.fabricmc.net/develop/automatic-testing) (Game Tests) plus this repo’s build guide **§4** (`.cursor/fabric-mod-build-release-guide-v4.3.md`).

**Scope:** Boot a real Minecraft **server** (and optionally **client**) under Fabric to validate load, mixins, and gameplay hooks. **Not** unit tests: those needing Fabric Loader, bootstrap, registries, or mapped game classes go to **`mc-fabric-loader-junit`**; pure helpers, math, and parsers stay on the repository's ordinary JUnit setup.

**Build-script DSL:** Examples use **Kotlin DSL**. Translate for Groovy if the repo uses it.

## When to use

- Scaffold or fix **`fabric-gametest`** / Loom **`configureTests`** on a Fabric mod.
- Multi-`mc*` line needs a **load/mixin smoke** per subproject (`./gradlew runGametest` or build-time server tests).
- User asks for **client game tests** / screenshots / `runClientGameTest` (opt-in; heavier).
- User names **`FabricGameTestHelper`** — that type is **removed** from current Fabric; say so and route them to the `@GameTest` + `GameTestHelper` pattern below.

## Route

| Goal | Skill |
|------|--------|
| Server/client game tests, Loom `configureTests` | **This skill** |
| Unit tests needing Fabric Loader, Minecraft bootstrap, registries, or mapped game classes | **`mc-fabric-loader-junit`** |
| Pure helpers, math, parsers, registry-free codecs | Repository's **ordinary JUnit** setup — no Fabric test skill |
| New `mc*` module | **`mc-scaffold-new-fabric-subproject`** (calls for gametest wiring — follow **this** skill for details) |

## Default multi-`mc*` pattern

Match a per-module main-source smoke layout unless the repo already uses Fabric’s separate `src/gametest` source set:

1. **Per `mc*` `build.gradle.kts`:**

```kotlin
fabricApi {
    configureTests {
        eula = true // agrees to Minecraft EULA for test runs
    }
}

dependencies {
    // 1.21.x: modImplementation; 26.x: implementation — match sibling deps
    modImplementation(
        fabricApi.module("fabric-gametest-api-v1", fabricApiVersion),
    )
}
```

On **26.x** use `implementation(fabricApi.module(...))` like other Fabric API deps.

2. **Smoke test class** with Fabric `@GameTest` (package under that subproject or shared — follow existing layout):

```java
import net.fabricmc.fabric.api.gametest.v1.GameTest;
import net.minecraft.gametest.framework.GameTestHelper;

public final class ExampleModGameTest {
    @GameTest
    public void smoke(GameTestHelper context) {
        context.succeed();
    }
}
```

Minimal smoke proves the mod **loads** and the gametest entrypoint runs. Add real assertions (`assertBlockPresent`, entity checks, etc.) only when testing a feature.

3. **`fabric.mod.json`** for **that** subproject — entrypoint:

```json
"entrypoints": {
  "fabric-gametest": [
    "com.example.ExampleModGameTest"
  ]
}
```

4. **Legacy `FabricGameTestHelper` (removed API — never for new work):** current Fabric no longer ships that type, so the mapping-qualified `@GameTest` + `GameTestHelper` pattern above is the only path to scaffold. If a **pre-existing** repo still compiles an empty class implementing the removed helper, handle it in place (keep the same `fabric-gametest` entrypoint) or migrate it to `@GameTest` — do not add a second scaffold alongside it.

### Run

```bash
./gradlew runGametest
# or per module, e.g.:
./gradlew :mc1.21.11:runGametest :mc26.2:runGametest
```

Server game tests also run as part of **`build`** when Loom tests are configured. Follow **`mc-gradle-daemon-hygiene`**.

## Fabric docs pattern (separate `gametest` source set)

Use when the user wants the official [createSourceSet](https://docs.fabricmc.net/develop/automatic-testing) layout:

```kotlin
fabricApi {
    configureTests {
        createSourceSet = true
        modId = "example-mod-test-${project.name}"
        enableGameTests = true
        enableClientGameTests = true // opt-in; default true in docs — set false if unused
        eula = true
    }
}
```

Then add `src/gametest/resources/fabric.mod.json` with `fabric-gametest` / `fabric-client-gametest` entrypoints and classes under `src/gametest/java/`. See Fabric docs for full snippets.

**Do not** force this layout onto a multi-`mc*` repo that already uses per-module main-source smoke tests.

## Client game tests (opt-in)

Client tests implement **`FabricClientGameTest`**, run via **`runClientGameTest`** / production-run tasks, and are **heavier** (headless CI needs XVFB on Linux). Defaults:

- **Skip** unless the user explicitly wants client UI/screenshot coverage.
- Prefer Prism / manual playtest for Mod Menu and rendering.
- If enabling CI: follow Fabric docs (`ClientProductionRunTask`, `useXVFB`, optional `-Dfabric.client.gametest.disableNetworkSynchronizer=true` if Actions hits network synchronizer errors).

## `server_retain` / `extract` / third-party forks

For **`track_class: server_retain`** mods: optional **minimal** server smoke only if version ports keep regressing. Do not build a large gametest suite or client CI for pack-only forks.

For **`track_class: extract`** mods: treat as a shippable product once the Fabric scaffold exists. Use the normal gametest path; do not apply the pack-only skip.

## Checklist

- [ ] `configureTests { eula = true }` (or equivalent) on modules that run gametest.
- [ ] `fabric-gametest-api-v1` module dependency with correct `modImplementation` vs `implementation`.
- [ ] `fabric-gametest` entrypoint in each shippable `fabric.mod.json` that should boot tests.
- [ ] At least one `@GameTest` taking `GameTestHelper` that `succeed()`s.
- [ ] `./gradlew … runGametest` or `build` green under the correct JDK.
- [ ] Client gametest only if explicitly requested.

## Examples

- "Add gametest to ASBF" → per-`mc*` `configureTests` + smoke `@GameTest` + `fabric-gametest` entrypoint; no client tests.
- "Scaffold mc26.2" → after **`mc-scaffold-new-fabric-subproject`**, wire gametest for the new module with this skill.
- "Client screenshot CI" → enable client game tests + production run task per Fabric docs; warn about Actions flakiness.

## Failure modes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| No gametest task / silent skip | Missing `configureTests` or entrypoint | Add Loom block + `fabric-gametest` class name |
| EULA / run abort | `eula` not true | Set `eula = true` in `configureTests` |
| Client CI network sync error | Known Actions issue | JVM arg `disableNetworkSynchronizer` per Fabric docs |
| Empty test task fails `:build` | Miswired test source set | Align with sibling modules; or `-x test` only as temporary escape (prefer fix) |

## Related

- **`mc-fabric-loader-junit`** — unit tests.
- **`mc-scaffold-new-fabric-subproject`** — new `mc*` + pointer to this skill.
- **`.cursor/fabric-mod-build-release-guide-v4.3.md`** §4.
- Official: [Automated Testing](https://docs.fabricmc.net/develop/automatic-testing).
