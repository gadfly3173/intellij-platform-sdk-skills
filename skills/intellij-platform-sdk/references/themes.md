# Themes

Use this reference when the user needs to create, customize, or extend IDE themes (UI themes), including color schemes, icon overrides, and the Islands theme format.

## What this file covers

- theme project structure
- theme JSON format and metadata
- color scheme integration
- icon customization
- extending existing themes
- Islands theme support
- deploying and publishing themes

## Theme basics

IntelliJ Platform themes control the visual appearance of the IDE, including editor colors and UI component colors. Themes are distributed as plugins.

A theme plugin bundles:

1. A theme descriptor JSON file under `src/main/resources/themes/`
2. (Optionally) an editor color scheme `.icls` file
3. (Optionally) custom icons
4. Standard `plugin.xml` with theme-specific metadata

## Theme JSON format

### Minimal theme descriptor

```json
{
  "name": "My Theme",
  "dark": false,
  "author": "Your Name",
  "editorScheme": "/themes/my_theme.xml",
  "colors": {
    "Button.background": "#ffffff"
  },
  "icons": {
    "ColorPalette": {
      "Actions.Green": "#00ff00"
    }
  }
}
```

### Key fields

- `name` — display name
- `dark` — `true` for dark themes, `false` for light
- `author` — author name (displayed in settings)
- `editorScheme` — path to `.icls` or `.xml` color scheme file (relative to resources)
- `colors` — UI component color overrides
- `icons` — icon color palette overrides
- `ui` — advanced UI property overrides (optional)
- `iconsScheme` — path to an icon theme descriptor (optional)
- `parents` — inherited theme (e.g., `"JetBrains Dark"`)

### Meta information in plugin.xml

```xml
<idea-plugin>
    <id>com.example.my-theme</id>
    <name>My Theme</name>
    <vendor>Your Name</vendor>

    <!-- Critical: mark as theme -->
    <idea-version since-build="241"/>

    <depends>com.intellij.modules.platform</depends>

    <extensions defaultExtensionNs="com.intellij">
        <themeProvider id="com.example.my-theme" path="/themes/my_theme.json"/>
    </extensions>
</idea-plugin>
```

### Theme resource structure

```
src/main/resources/
├── themes/
│   ├── my_theme.json          # theme descriptor
│   └── my_theme.xml           # editor color scheme (optional)
└── icons/                      # custom icons (optional)
```

## Color scheme

Editor color schemes define syntax highlighting colors. Integrate with a theme in two ways:

1. **Separate `.icls` file** — point to it from the theme JSON's `editorScheme` field
2. **Add-on theme** — override parent theme colors only

### Accessing platform colors in code

```java
// Theme-aware colors
Color bg = JBColor.background();          // adapts to dark/light
Color namedColor = JBColor.namedColor(    // adapts to custom theme
    "Button.default.startBackground",
    new JBColor(0xffffff, 0x3c3f41)
);
```

Use `JBColor.namedColor()` for colors that need to adapt to custom user themes. Use `JBColor` with light/dark pairs for basic theme adaptation.

## Extending existing themes

A theme can extend a parent theme by specifying `parents` in the JSON:

```json
{
  "name": "My Custom Dark",
  "dark": true,
  "parents": {
    "JetBrains Dark": {}
  },
  "colors": {
    "Button.background": "#2b2d30"
  }
}
```

When extending, only specify the overrides you need. All unspecified values inherit from the parent.

Child themes can also extend icon palettes from parent themes.

## Islands theme support

The Islands theme is a visual redesign introduced in 2025.x IDE versions. Plugins should test their UI components under the Islands theme to ensure visual consistency.

Key considerations for Islands theme compatibility:

- Use `JBColor.namedColor()` for colors that should adapt to theme variations
- Test custom UI components in both Classic and Islands themes
- Avoid hardcoded colors or font sizes
- Use `JBUI.Borders` and `JBUI.Insets` instead of raw `java.awt` measurements
- For custom icons, the Islands theme may apply different color transforms — test SVG icons in both themes

## Creating a theme project

### Gradle setup

```kotlin
plugins {
    id("org.jetbrains.intellij.platform") version "2.13.1"
}

repositories {
    mavenCentral()
    intellijPlatform {
        defaultRepositories()
    }
}

dependencies {
    intellijPlatform {
        intellijIdea("2024.3")
    }
}

intellijPlatform {
    pluginConfiguration {
        id = "com.example.my-theme"
        name = "My Theme"
        vendor { name = "Your Name" }
        ideaVersion { sinceBuild = "243" }
    }
}
```

### Running and debugging

```bash
./gradlew runIde
```

Theme changes take effect after IDE restart in the sandbox. To test:
1. `./gradlew runIde`
2. In the sandbox IDE: Settings > Appearance & Behavior > Appearance > Theme
3. Select the target theme
4. Verify editor colors, UI components, and icons

## Deploying a theme

1. Build the plugin: `./gradlew buildPlugin`
2. Sign the plugin if required: `./gradlew signPlugin`
3. Publish to Marketplace: manual upload for first release, then `./gradlew publishPlugin`

Themes are subject to the same Marketplace review process as other plugins. Before publishing:
- Test in both dark and light variants
- Verify editor scheme is consistent
- Ensure custom icons render correctly at different DPI settings
- Include a high-quality screenshot in the Marketplace listing

## Guidance

- Start from an existing theme using `parents` inheritance — don't define every color from scratch
- Use the UI Inspector (Internal Actions > UI > UI Inspector) to find component color keys
- `JBColor.namedColor()` is the right API for colors that should respect custom themes
- Theme plugins should have minimal dependencies — typically `com.intellij.modules.platform` only
- Theme `.json` files are hot-reloaded during theme development in internal mode
- For dedicated icon themes, use `iconsScheme` in the theme descriptor to point to a separate icon palette

## When to also read other references

- Read `getting-started.md` for project setup
- Read `ui-settings-and-toolwindows.md` for UI component API details
- Read `testing-and-publishing.md` for signing and Marketplace publishing
- Read `extension_points.md` for `themeProvider` registration
