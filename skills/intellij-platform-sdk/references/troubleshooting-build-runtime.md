# Build, Runtime, and Verification Troubleshooting

Use this reference when the problem is about Gradle setup, dependency resolution, runtime class loading, action registration, service initialization, verifier warnings, or release-time failures.

## Build issues

### `Could not resolve org.jetbrains.intellij.platform`

Usually means the plugin repositories are incomplete.

```kotlin
repositories {
    mavenCentral()
    intellijPlatform {
        defaultRepositories()
    }
}
```

### `Plugin [id: 'org.jetbrains.intellij.platform'] was not found`

Check:

- plugin version exists
- plugin management/settings are correct
- Gradle version is high enough for the selected plugin version

```kotlin
plugins {
    id("org.jetbrains.intellij.platform.settings") version "2.2.0"
}
```

### Bundled plugin not found

Use the correct bundled plugin ID for the selected IDE.

```kotlin
dependencies {
    intellijPlatform {
        intellijIdeaCommunity("2024.3")
        bundledPlugin("com.intellij.java")
    }
}
```

## Runtime issues

### `ClassNotFoundException`

Check:

1. class/package name in `plugin.xml`
2. source-set placement
3. whether the dependency/module/plugin is actually available at runtime

### Service initialization failed

Common causes:

- missing `@Service`
- wrong service level
- constructor doing too much work
- circular dependency chains

Preferred pattern:

```java
@Service(Service.Level.PROJECT)
public final class MyProjectService {
    private final Project project;

    public MyProjectService(Project project) {
        this.project = project;
    }
}
```

### Extension point class not found

Check:

- correct EP namespace
- required module/plugin dependency exists
- target IDE actually contains that EP

## Action issues

### Action not visible

Check:

1. `plugin.xml` registration is correct
2. target action group exists
3. `update()` sets visibility and enabled state correctly

```java
@Override
public void update(@NotNull AnActionEvent e) {
    e.getPresentation().setEnabledAndVisible(e.getProject() != null);
}
```

### Missing `getActionUpdateThread()` warning

```java
@Override
public @NotNull ActionUpdateThread getActionUpdateThread() {
    return ActionUpdateThread.BGT;
}
```

### Slow action updates / UI freeze

Move heavy work out of `update()` and into `actionPerformed()` or background execution.

## Plugin verifier

### Typical verifier problems

- internal API usage
- deprecated API usage
- missing or wrong build range
- unavailable dependency/plugin classes on some target IDEs

Run:

```bash
./gradlew verifyPlugin
```

Then inspect the generated report under `build/reports/pluginVerifier/`.

### Internal API usage warnings

Prefer public APIs. Only fall back to internal APIs when the tradeoff is explicit and justified.

## Release-time checks

Before publishing, verify:

- signing inputs are present
- verifier is clean enough for target IDEs
- `sinceBuild`/`untilBuild` are realistic
- optional dependencies are truly optional in code paths

## Common error messages

### `Cannot create class`

Usually a constructor/signature/classloading problem.

### `Extension implementation class not found`

Usually a wrong fully qualified class name or unavailable dependency.

### `Service configuration is missing`

Register the service in `plugin.xml` if not using a light service pattern.

## See also

- `getting-started.md`
- `compatibility.md`
- `extension_points.md`
- `testing-and-publishing.md`
