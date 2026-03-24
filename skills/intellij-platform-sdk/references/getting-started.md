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

- JDK 21 for current platform targets
- Gradle 8.5+
- IntelliJ Platform Gradle Plugin `2.2.x`
- Prefer Kotlin DSL in Gradle files

## `build.gradle.kts` template

```kotlin
plugins {
    id("java")
    id("org.jetbrains.intellij.platform") version "2.2.0"
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
        intellijIdeaCommunity("2024.3")
        bundledPlugin("com.intellij.java")
    }
}

intellijPlatform {
    pluginConfiguration {
        name = "My Plugin"
        description = "Plugin description"
        ideaVersion {
            sinceBuild = "243"
            untilBuild = "243.*"
        }
    }
}
```

## Target IDE selection

Choose the IDE artifact based on the real target:

- `intellijIdeaCommunity("2024.3")`
- `intellijIdeaUltimate("2024.3")`
- `pycharmCommunity("2024.3")`
- `webstorm("2024.3")`
- `goland("2024.3")`
- `rider("2024.3")`
- `androidStudio("2024.3")`

Only add bundled plugins you truly need.

## `plugin.xml` essentials

```xml
<idea-plugin>
    <id>com.example.myplugin</id>
    <name>My Plugin</name>
    <vendor>Gadfly</vendor>

    <depends>com.intellij.modules.platform</depends>
    <!-- Add product/language specific modules only when needed -->
    <depends>com.intellij.modules.java</depends>

    <extensions defaultExtensionNs="com.intellij">
        <!-- extension points go here -->
    </extensions>

    <actions>
        <!-- actions go here -->
    </actions>
</idea-plugin>
```

## Dependency strategy

Prefer the smallest viable dependency set:

- Use `com.intellij.modules.platform` for generic plugins
- Add `com.intellij.modules.lang` if you work with PSI/editor language features
- Add `com.intellij.modules.java` only for Java-specific PSI or UI integration
- Use optional dependencies for integrations that may not exist in every IDE

## Optional dependency pattern

```xml
<depends optional="true">org.jetbrains.plugins.yaml</depends>
```

Use this when your plugin can still function without the dependency.

## Project setup advice

When implementing for a user:

1. Identify target IDE(s)
2. Identify required language/plugin dependencies
3. Keep the initial setup minimal
4. Put implementation classes in `src/main/java` or `src/main/kotlin`
5. Put icons, templates, and `plugin.xml` resources under `src/main/resources`

## Common setup mistakes

- Using the legacy `org.jetbrains.intellij` plugin instead of `org.jetbrains.intellij.platform`
- Forgetting `intellijPlatform { defaultRepositories() }`
- Declaring too many bundled plugins
- Setting an overly broad `untilBuild`
- Targeting Java APIs without `com.intellij.modules.java`

## When to also read other references

- Read `compatibility.md` for version migration or build-range questions
- Read `extension_points.md` for exact XML registration syntax
- Read `testing-and-publishing.md` for verifier, signing, and Marketplace release
