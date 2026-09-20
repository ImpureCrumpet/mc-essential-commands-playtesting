---
name: mc-fabric-loader-junit
description: "Sets up and writes Fabric Loader JUnit unit tests for Fabric mod code that needs Fabric Loader, Minecraft bootstrap, registries, or mapped game classes: Gradle wiring, registry bootstrap, and CI report artifacts; pure helpers, math, parsers, and registry-free codecs stay on the repository's ordinary JUnit setup."
triggers:
  - fabric-loader-junit
  - Fabric Loader JUnit
  - unit test fabric
  - junit fabric mod
  - useJUnitPlatform
  - Bootstrap.bootStrap
  - src/test/java
  - testImplementation fabric-loader-junit
  - mc-fabric-loader-junit
dependencies:
  - mc-gradle-daemon-hygiene
version: "1.1.1"
---

# Fabric Loader JUnit (unit tests)

**Source:** [Fabric Documentation — Automated Testing](https://docs.fabricmc.net/develop/automatic-testing) (Unit Testing).

**Scope:** **Unit tests that need the Fabric environment** — Fabric Loader on the test classpath, Minecraft bootstrap, registries, or mapped game classes (`ItemStack`, `Registry`, registry-backed codecs). **Pure** Java/Kotlin helpers, math, parsers, and registry-free codecs do **not** belong here: run them on the repository's **ordinary JUnit** setup. **Not** full gameplay / mixin load smoke either — that is **`mc-fabric-gametest`**.

**Build-script DSL:** Examples use **Kotlin DSL**. Translate for Groovy if the repo uses it.

## When to use

- Code under test touches **registry- or bootstrap-dependent** vanilla types (`ItemStack`, `Registry`, registry-backed codecs) or otherwise needs **Fabric Loader** on the test classpath, but not a full client/server.
- User asks for **JUnit** / **unit tests** / **`fabric-loader-junit`** on a Fabric mod **and** the logic under test reaches into Minecraft.
- CI should fail on regressions in that Minecraft-touching logic without spinning gametest.

**Do not** use this skill for:

- **Pure logic** — stack math on plain numbers, parsers, formula tables, registry-free codecs. Those go on the repository's **ordinary JUnit** setup (plain `org.junit.jupiter` + `useJUnitPlatform()`); Fabric Loader JUnit adds nothing.
- “Does the mod load / mixins apply?” — use **`mc-fabric-gametest`**.

## Route

| Goal | Skill |
|------|--------|
| Pure helpers, math, parsers, registry-free codecs (no Minecraft types) | Repository's **ordinary JUnit** setup — **not** this skill |
| Fabric Loader, `Bootstrap.bootStrap()`, registries, or mapped game classes in a unit test | **This skill** |
| Server/client load + `@GameTest` / Loom `configureTests` | **`mc-fabric-gametest`** |
| New `mc*` subproject (includes gametest entrypoint pointer) | **`mc-scaffold-new-fabric-subproject`** → then gametest skill |

## Gradle setup

Only for suites in **Scope**. If the module's tests are all pure logic, keep the repository's existing plain-JUnit wiring and stop here.

In the **module that owns `src/test`** (often root or a single-module mod; for multi-`mc*`, put shared unit tests in **root** `src/test` if that source set is wired — follow the existing repo):

```kotlin
dependencies {
    testImplementation("net.fabricmc:fabric-loader-junit:${property("loader_version")}")
    // or hard-code the same loader version string the module already uses
}

tasks.test {
    useJUnitPlatform()
}
```

- Match **`loader_version`** to the Fabric Loader coordinate already used for that line (1.21.x vs 26.x may differ).
- Tests that actually load **Mixin-transformed** or **remapped Minecraft** classes need **`fabric-loader-junit`** — with only `org.junit.jupiter` on the test classpath they break at class-load time, so do not skip the Fabric artifact for those. Tests that touch **no** Minecraft types are unaffected and stay on plain `org.junit.jupiter`.

Reload Gradle, then create tests under **`src/test/java/`**.

## Writing tests

1. Mirror packages of the class under test, or use a `…test…` package if the repo uses Java modules.
2. Use **`org.junit.jupiter.api`** (`@Test`, `@BeforeAll`, `Assertions`).
3. Prefer testing **shared root** classes (no version-specific adapters) so one suite covers all `mc*` lines.

### Registry / vanilla types

Accessing `ItemStack`, registries, or other bootstrapped types without init throws **`Not bootstrapped`**. In `@BeforeAll` (or once per suite):

```java
SharedConstants.tryDetectVersion();
Bootstrap.bootStrap();
```

Only add this when the test touches registry-dependent types. Pure math/helper tests need no bootstrap — and belong on the ordinary JUnit setup (see **Scope**), not here.

### Example shape

A suite that belongs here touches game types, so the bootstrap is real rather than commented out:

```java
public class ExampleItemStackTest {
    @BeforeAll
    static void beforeAll() {
        SharedConstants.tryDetectVersion();
        Bootstrap.bootStrap();
        // MyModTypes.register(); // if the assertions need your registries
    }

    @Test
    void testSomething() {
        Assertions.assertEquals(expected, actual);
    }
}
```

## Existing `main`-method smoke tests

Some mods historically ship a **`main`-method smoke** under `src/test` without JUnit. When adding **new** unit coverage, prefer a **real JUnit suite** so `./gradlew test` / CI actually runs it — ordinary JUnit for pure logic, **Fabric Loader JUnit** only when the suite needs the Fabric environment. Migrating an existing `main` smoke to JUnit is encouraged but not mandatory in the same change.

## Run

```bash
./gradlew test
# or for a subproject:
./gradlew :mc1.21.11:test
```

Follow **`mc-gradle-daemon-hygiene`** (`j21` / `j25`, `./gradlew --stop` after JDK switch).

## CI (GitHub Actions)

If the workflow already runs `./gradlew build`, `test` usually runs as part of `check`/`build`. On failure, upload reports:

```yaml
- name: Store reports
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: reports
    path: |
      **/build/reports/
      **/build/test-results/
```

## Checklist

- [ ] Suite really needs the Fabric environment; pure-logic tests left on (or moved to) the ordinary JUnit setup.
- [ ] `fabric-loader-junit` on `testImplementation` with matching loader version.
- [ ] `useJUnitPlatform()` on `test` task.
- [ ] Bootstrap only when registries / vanilla types are used.
- [ ] Tests target shared logic — not a substitute for gametest.
- [ ] `./gradlew test` green under the correct JDK.

## Examples

- "Add JUnit for my panic-stack math" → if the math is plain numbers, use the repo's ordinary JUnit setup and assert stack/decay invariants; come back here only if it needs `ItemStack` / registries.
- "ItemStack assertions fail Not bootstrapped" → add `SharedConstants.tryDetectVersion()` + `Bootstrap.bootStrap()` in `@BeforeAll`.
- "Should I unit-test mixins?" → no — use **`mc-fabric-gametest`**.

## Related

- **`mc-fabric-gametest`** — server/client game tests.
- **`mc-gradle-daemon-hygiene`** — JDK / daemon before Gradle.
- Official: [Automated Testing](https://docs.fabricmc.net/develop/automatic-testing).
