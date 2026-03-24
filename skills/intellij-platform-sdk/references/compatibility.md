# IntelliJ Platform SDK Version Compatibility

This document covers version compatibility, migration guides, and API changes.

## Version Numbering

IntelliJ Platform uses the following versioning scheme:

| Version | Build Number | Format |
|---------|--------------|--------|
| 2024.3 | 243.xxxx.xx | YYM.BUILD.PATCH |
| 2024.2 | 242.xxxx.xx | YYM.BUILD.PATCH |
| 2024.1 | 241.xxxx.xx | YYM.BUILD.PATCH |

Build number format: `YYM` where:
- `YY` = Year (24 = 2024)
- `M` = Month (1-9 = Jan-Sep, A-C = Oct-Dec)

## Gradle Plugin Versions

| IntelliJ Platform Gradle Plugin | Minimum Gradle | Notes |
|--------------------------------|----------------|-------|
| 2.2.x | 8.5 | Current stable |
| 2.1.x | 8.5 | - |
| 2.0.x | 8.1 | Major rewrite |
| 1.x | 7.3 | Legacy |

## SDK Migration (1.x to 2.x)

### build.gradle.kts Changes

**1.x (Legacy):**
```kotlin
plugins {
    id("java")
    id("org.jetbrains.intellij") version "1.17.3"
}

intellij {
    version.set("2023.2.5")
    type.set("IC")
    plugins.set(listOf("java"))
}

tasks {
    patchPluginXml {
        sinceBuild.set("232")
        untilBuild.set("232.*")
    }
}
```

**2.x (Current):**
```kotlin
plugins {
    id("java")
    id("org.jetbrains.intellij.platform") version "2.2.0"
}

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
        ideaVersion {
            sinceBuild = "243"
            untilBuild = "243.*"
        }
    }
}
```

### Key Differences

| Aspect | 1.x | 2.x |
|--------|-----|-----|
| Plugin ID | `org.jetbrains.intellij` | `org.jetbrains.intellij.platform` |
| IDE Version | `intellij.version` | `intellijPlatform { intellijIdeaCommunity() }` |
| Plugins | `intellij.plugins` | `intellijPlatform { bundledPlugin() }` |
| Patching | `patchPluginXml` | `pluginConfiguration` |
| Repositories | Automatic | Explicit `intellijPlatform { defaultRepositories() }` |

## API Compatibility

### Since/UnUntil Build

```xml
<!-- plugin.xml -->
<idea-version since-build="243" until-build="243.*"/>
```

| Version | Build | Since |
|---------|-------|-------|
| 2024.3 | 243.xxx | 243 |
| 2024.2 | 242.xxx | 242 |
| 2024.1 | 241.xxx | 241 |
| 2023.3 | 233.xxx | 233 |
| 2023.2 | 232.xxx | 232 |
| 2023.1 | 231.xxx | 231 |

### Notable API Changes

#### 2024.3
- New: `ActionUpdateThread.getActionUpdateThread()` required
- New: `DumbAware` marker interface for extension points
- Deprecation: `AnAction.isDumbAware()` override

#### 2024.2
- New: Intention actions can be `DumbAware`
- New: `LightProjectDescriptor` for tests

#### 2024.1
- New: `VirtualFileGist` and `PsiFileGist`
- Deprecation: `FileTypeFactory`

#### 2023.3
- New: `ActionUpdateThread` enum
- Required: Override `getActionUpdateThread()` in actions

#### 2023.2
- New: IntelliJ Platform Gradle Plugin 2.0
- Breaking: Gradle 8.1+ required

#### 2023.1
- New: Lazy service initialization
- New: `@Service` with level parameter

## Language Level Compatibility

| IntelliJ Platform | Java | Kotlin |
|-------------------|------|--------|
| 2024.3 | 21 | 2.0 |
| 2024.2 | 21 | 1.9 |
| 2024.1 | 17 | 1.9 |
| 2023.3 | 17 | 1.9 |
| 2023.2 | 17 | 1.8 |
| 2023.1 | 17 | 1.8 |

## Target IDE Compatibility

### Plugin Targets

| IDE | Artifact | Notes |
|-----|----------|-------|
| IntelliJ IDEA Community | `intellijIdeaCommunity()` | Free, open source |
| IntelliJ IDEA Ultimate | `intellijIdeaUltimate()` | Commercial |
| PyCharm Community | `pycharmCommunity()` | Python IDE |
| PyCharm Professional | `pycharmProfessional()` | Commercial |
| WebStorm | `webstorm()` | JavaScript IDE |
| PhpStorm | `phpstorm()` | PHP IDE |
| GoLand | `goland()` | Go IDE |
| Rider | `rider()` | .NET IDE |
| CLion | `clion()` | C/C++ IDE |
| DataGrip | `datagrip()` | Database IDE |
| RubyMine | `rubymine()` | Ruby IDE |
| Android Studio | `androidStudio()` | Android IDE |

### Dependency Declaration

```xml
<!-- For cross-platform plugins -->
<depends>com.intellij.modules.platform</depends>

<!-- For Java-specific plugins -->
<depends>com.intellij.modules.java</depends>

<!-- For Python-specific plugins -->
<depends optional="true">PythonCore</depends>

<!-- For JavaScript-specific plugins -->
<depends optional="true">JavaScript</depends>
```

## Testing Compatibility

### Test Framework Versions

| Platform Version | Test Framework | Notes |
|------------------|----------------|-------|
| 2024.3 | `com.jetbrains.intellij.platform:test-framework` | New in 2.x |
| 2024.2 | `com.jetbrains.intellij.platform:test-framework` | New in 2.x |
| Legacy | `com.jetbrains.intellij.java:java-test-framework` | 1.x style |

### Test Dependencies (2.x)

```kotlin
dependencies {
    intellijPlatform {
        intellijIdeaCommunity("2024.3")
        testFramework(TestFrameworkType.Platform)
        testFramework(TestFrameworkType.Plugin.Java)
    }

    testImplementation("junit:junit:4.13.2")
}
```

## Breaking Changes Log

### 2024.3 Breaking Changes
- `AnAction.isDumbAware()` should not be overridden
- Use `DumbAwareAction` or `getActionUpdateThread()` instead

### 2024.2 Breaking Changes
- `FileTypeFactory` removed (deprecated since 2020.1)
- Use `<fileType>` extension point instead

### 2023.3 Breaking Changes
- `ActionUpdateThread.REQUIRED` added
- Must override `getActionUpdateThread()` in all actions

### 2023.2 Breaking Changes
- Gradle 8.1+ required for 2.x plugin
- Kotlin DSL syntax changes

### 2023.1 Breaking Changes
- `ServiceManager` deprecated
- Use `getService()` on Application/Project/Module instead

### 2022.3 Breaking Changes
- `ActionUpdateThread` introduced
- Threading model enforcement

### 2022.2 Breaking Changes
- `DataKeys` removed
- Use `CommonDataKeys` instead

### 2022.1 Breaking Changes
- `InspectionProfileEntry` constructor changes
- `GlobalInspectionTool` API changes

## Migration Checklist

### From 1.x to 2.x

- [ ] Update plugin ID in `build.gradle.kts`
- [ ] Add `intellijPlatform { defaultRepositories() }`
- [ ] Move IDE version to `dependencies { intellijPlatform {} }`
- [ ] Update plugin dependencies syntax
- [ ] Update `patchPluginXml` to `pluginConfiguration`
- [ ] Update test dependencies
- [ ] Verify Gradle version (8.5+)
- [ ] Test plugin functionality

### General Version Updates

- [ ] Update `since-build` and `until-build`
- [ ] Check deprecated API usage
- [ ] Update Kotlin/Java language level if needed
- [ ] Run plugin verifier
- [ ] Test on target IDE versions
- [ ] Update changelog

## Plugin Verifier

Run the plugin verifier to check compatibility:

```bash
./gradlew verifyPlugin
```

Or with specific IDE versions:

```bash
./gradlew verifyPlugin -PideVersions=IC-2024.3,IC-2024.2
```
