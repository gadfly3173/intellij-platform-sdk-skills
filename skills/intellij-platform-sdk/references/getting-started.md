# Getting Started with IntelliJ Platform SDK

Use this reference when the user needs to create a new plugin, configure Gradle, choose target IDEs, or understand plugin packaging basics.

## What this file covers

- IntelliJ Platform Gradle Plugin 2.x setup
- `build.gradle.kts` structure
- `settings.gradle.kts` basics
- `plugin.xml` essentials
- target IDE and bundled plugin selection
- minimal project layout
- starter snippets for new plugins

## Minimal project structure

```text
my-plugin/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── src/
│   ├── main/
│   │   ├── java/ or kotlin/
│   │   └── resources/
│   │       └── META-INF/plugin.xml
│   └── test/
└── gradle/
```

## Recommended baseline

- JDK 17+ (minimum required by the IntelliJ Platform Gradle Plugin 2.x; JDK 21 recommended for 2025.1+ targets)
- Gradle 8.13+
- IntelliJ Platform Gradle Plugin `2.13.1`
- Prefer Kotlin DSL in Gradle files

## `build.gradle.kts` template

```kotlin
plugins {
    id("java")
    id("org.jetbrains.intellij.platform") version "2.13.1"
}

group = "com.example"
version = "1.0.0"

repositories {
    mavenCentral()
    intellijPlatform {
        defaultRepositories()
    }
}

dependencies {
    intellijPlatform {
        intellijIdea("2024.3")
        bundledPlugin("com.intellij.java")

        testFramework(TestFrameworkType.Platform)
    }
}

intellijPlatform {
    pluginConfiguration {
        id = "com.example.myplugin"
        name = "My Plugin"
        version = project.version.toString()
        description = "Plugin description"
        vendor {
            name = "Gadfly"
        }
        ideaVersion {
            sinceBuild = "243"
            untilBuild = "243.*"
        }
    }
}
```

## `settings.gradle.kts` template

Minimal:

```kotlin
rootProject.name = "my-plugin"
```

With centralized dependency resolution (recommended for multi-module or strict builds):

```kotlin
import org.jetbrains.intellij.platform.gradle.extensions.intellijPlatform

plugins {
    id("org.jetbrains.intellij.platform.settings") version "2.13.1"
}

dependencyResolutionManagement {
    repositoriesMode = RepositoriesMode.FAIL_ON_PROJECT_REPOS

    repositories {
        mavenCentral()

        intellijPlatform {
            defaultRepositories()
        }
    }
}

rootProject.name = "my-plugin"
```

## `gradle.properties` template

```properties
# IntelliJ Platform Gradle Plugin settings
org.jetbrains.intellij.platform.downloadSources=true
org.jetbrains.intellij.platform.selfUpdateCheck=false

# Gradle performance
org.gradle.configuration-cache=true
org.gradle.caching=true
```

## Target IDE selection

Choose the IDE dependency based on the real target. The function name determines which IDE is used for development and testing:

- `intellijIdea("2024.3")` -- IntelliJ IDEA (Community + Ultimate)
- `pycharm("2024.3")` -- PyCharm
- `webstorm("2024.3")` -- WebStorm
- `goland("2024.3")` -- GoLand
- `rider("2024.3")` -- Rider
- `clion("2024.3")` -- CLion
- `phpstorm("2024.3")` -- PhpStorm
- `rubymine("2024.3")` -- RubyMine
- `rustRover("2024.3")` -- RustRover
- `androidStudio("2024.3")` -- Android Studio
- `datagrip("2024.3")` -- DataGrip
- `dataspell("2024.3")` -- DataSpell
- `gateway("2024.3")` -- Gateway
- `fleetBackend("2024.3")` -- Fleet Backend
- `mps("2024.3")` -- MPS

Only add bundled plugins you truly need.

## `plugin.xml` essentials

```xml
<idea-plugin>
    <id>com.example.myplugin</id>
    <name>My Plugin</name>
    <vendor>Gadfly</vendor>

    <depends>com.intellij.modules.platform</depends>

    <extensions defaultExtensionNs="com.intellij">
        <!-- extension points go here -->
    </extensions>

    <actions>
        <!-- actions go here -->
    </actions>
</idea-plugin>
```

Add product/language-specific modules only when needed (e.g. `<depends>com.intellij.modules.java</depends>` for Java-specific PSI).

## Dependency strategy

Prefer the smallest viable dependency set:

- Use `com.intellij.modules.platform` for generic plugins
- Add `com.intellij.modules.lang` if you work with PSI/editor language features
- Add `com.intellij.modules.java` only for Java-specific PSI or UI integration
- Use optional dependencies for integrations that may not exist in every IDE

In Gradle, use `bundledPlugin("pluginId")` for bundled plugin dependencies and `bundledModule("moduleId")` for platform module dependencies. These are distinct helpers -- modules represent platform capabilities while plugins are optional feature bundles.

## Optional dependency pattern

When your plugin has extensions that depend on an optional plugin, use both `optional` and `config-file` attributes. The `config-file` is **required** when `optional` is `true`:

```xml
<depends optional="true"
         config-file="myPluginId-optionalPluginName.xml">org.jetbrains.plugins.yaml</depends>
```

The `config-file` points to a separate XML file (in the same directory as `plugin.xml`) that contains only the extensions for the optional integration. This file must have a unique name to avoid classloader conflicts in tests.

## Project setup advice

When implementing for a user:

1. Identify target IDE(s)
2. Identify required language/plugin dependencies
3. Keep the initial setup minimal
4. Put implementation classes in `src/main/java` or `src/main/kotlin`
5. Put icons, templates, and `plugin.xml` resources under `src/main/resources`

## Multi-module projects

For larger plugins, split into multiple Gradle modules. Use `org.jetbrains.intellij.platform.module` plugin in submodules and `pluginModule()` in the main module's dependencies to wire them together:

```kotlin
// In submodule build.gradle.kts
plugins {
    id("org.jetbrains.intellij.platform.module")
}

// In main module build.gradle.kts dependencies
dependencies {
    intellijPlatform {
        pluginModule(implementation(project(":submodule")))
    }
}
```

## Common setup mistakes

- Using the legacy `org.jetbrains.intellij` plugin instead of `org.jetbrains.intellij.platform`
- Forgetting `intellijPlatform { defaultRepositories() }`
- Declaring too many bundled plugins
- Setting an overly broad `untilBuild`
- Targeting Java APIs without `com.intellij.modules.java`
- Missing `config-file` in optional `<depends>` when separate extensions are needed
- Using Gradle version below the minimum required by the chosen plugin version

## When to also read other references

- Read `compatibility.md` for version migration or build-range questions
- Read `extension_points.md` for exact XML registration syntax
- Read `testing-and-publishing.md` for verifier, signing, and Marketplace release
