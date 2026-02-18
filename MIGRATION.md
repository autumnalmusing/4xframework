# Kotlin migration plan

This document outlines the process for forking, rewriting the 4X Framework in Kotlin, and getting it into a maintainable state.

## Current status

- **Kotlin support added** to the parent POM (`kotlin.version` 1.9.24, `kotlin-stdlib` in dependency management, `kotlin-maven-plugin` in plugin management).
- **Core module** is configured to use Kotlin (plugin + `kotlin-stdlib`). A placeholder file `core/src/main/kotlin/.../KotlinPlaceholder.kt` verifies the Kotlin source set. You can delete it once you start converting real code.
- **Verify build**: From repo root, run `mvn -pl core compile`. If you use a Maven wrapper, use `./mvnw` instead of `mvn`.

## Fork and remote

If you haven’t already:

1. Fork the repo on GitHub (or your host).
2. Add your fork as a remote and push this branch:
   ```bash
   git remote add origin <your-fork-url>
   git push -u origin <branch>
   ```

## Migration strategy: incremental

Keep **Java and Kotlin side by side** during the rewrite. Kotlin compiles first (`process-sources`), then Java; both output to `target/classes`, so each can call the other. Convert one package or class at a time and run tests often.

Suggested **order of conversion** (dependencies first, then consumers):

1. **Core – model and utils**  
   Start with leaf types: exceptions, `Location`, `Size`, `Range`, `CellType`, `I18NString`, `Validations`, `TreeNode`, `RootlessTree`, etc. Then `Cell`, `Zone`, `GameMap`, `Configuration`, `GameState`, `Game`, `Civilization`, `GameCommand`, domain events.
2. **Core – plugins**  
   Plugin loading, `PluginSpec`, `Plugin`, `PluginManager`, configuration loaders, command/map/lifecycle wrappers. This is the heaviest use of reflection and Vavr; convert to Kotlin idioms (nullable/types, `Result`/exceptions, `listOf`/`mapOf`/`sequence` where it helps).
3. **Console-client-api**  
   Small surface; quick to convert.
4. **Console-client**  
   Lanterna UI and Dagger. You can keep Dagger or move to manual wiring / Koin later.
5. **4x-plugin-standard**  
   Then standard-terminal-client, then civilization plugins (mostly resources; add Kotlin only where they have code).

Per module: add Kotlin the same way as in core (dependency on `kotlin-stdlib`, `kotlin-maven-plugin` from parent). Create `src/main/kotlin` (and `src/test/kotlin` when converting tests) and move or rewrite files there.

## After conversion

- **Drop Java**: Once a module is fully Kotlin, you can remove the Kotlin plugin’s `sourceDir` for `src/main/java` and delete the Java sources. Or leave the plugin as-is and just have no Java files.
- **Dependencies**: Update Vavr, JUnit, Mockito, Dagger, Lanterna, JAXB, Reflections, etc. to current versions and fix deprecations. Consider replacing Vavr gradually with Kotlin stdlib + `kotlinx-coroutines` or Arrow if you want a more Kotlin-idiomatic style.
- **Lint and format**: Add Detekt and/or ktlint, and optionally a code formatter (Spotless with ktlint).
- **README**: Update to describe the Kotlin fork, JDK version, and how to build and run (e.g. `mvn clean install`, then run the console client with the right classpath/plugin path).

## Quick reference: adding Kotlin to another module

In the module’s `pom.xml`:

1. Add dependency:
   ```xml
   <dependency>
       <groupId>org.jetbrains.kotlin</groupId>
       <artifactId>kotlin-stdlib</artifactId>
   </dependency>
   ```
2. Add plugin (inherits config from parent):
   ```xml
   <plugin>
       <groupId>org.jetbrains.kotlin</groupId>
       <artifactId>kotlin-maven-plugin</artifactId>
   </plugin>
   ```
3. Create `src/main/kotlin` with the same package structure as the Java code.

Then convert or add Kotlin files under `src/main/kotlin` (and `src/test/kotlin` for tests).
